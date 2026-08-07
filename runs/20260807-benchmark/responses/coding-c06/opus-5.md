交付一个可直接在浏览器打开的弓箭射击游戏：4 个靶子，全部命中即停表，成绩按用时排入本地排行榜。共 5 个文件，建议放在同一个目录（例如 `archery/`）。

## archery/core.js

```js
/* 弓箭射击模拟游戏 —— 纯逻辑核心。
   不依赖 DOM，可在浏览器与 Node 中运行（便于单元测试）。
   坐标系：x 向右，y 向下（与 Canvas 一致），角度以“水平向右为 0，向上为正”表示。 */
(function (root, factory) {
  const api = factory();
  if (typeof module === 'object' && module.exports) module.exports = api;
  else root.ArcheryCore = api;
})(typeof self !== 'undefined' ? self : globalThis, function () {
  'use strict';

  const GRAVITY = 980;        // px/s^2
  const MIN_SPEED = 340;      // 空拉（power=0）初速
  const MAX_SPEED = 1180;     // 满弓（power=1）初速
  const CHARGE_RATE = 1.25;   // 每秒蓄力量
  const AIM_MIN = -20 * Math.PI / 180;
  const AIM_MAX = 85 * Math.PI / 180;
  const STORAGE_KEY = 'archery.leaderboard.v1';
  const MAX_ENTRIES = 10;

  // 关卡完全固定（靶子的摆动只依赖计时时间），保证每位玩家面对同一挑战，排行榜才公平。
  const LEVEL = Object.freeze({
    width: 960,
    height: 540,
    groundY: 470,
    bow: Object.freeze({ x: 96, y: 392 }),
    targets: Object.freeze([
      Object.freeze({ id: 1, name: '近靶',   x: 405, baseY: 320, r: 38, bobAmp: 0,  bobSpeed: 0.00, phase: 0.0 }),
      Object.freeze({ id: 2, name: '中靶',   x: 555, baseY: 268, r: 32, bobAmp: 36, bobSpeed: 1.10, phase: 0.6 }),
      Object.freeze({ id: 3, name: '远靶',   x: 707, baseY: 320, r: 27, bobAmp: 68, bobSpeed: 1.45, phase: 2.1 }),
      Object.freeze({ id: 4, name: '极远靶', x: 862, baseY: 250, r: 22, bobAmp: 96, bobSpeed: 1.85, phase: 4.0 })
    ])
  });

  const clamp = (v, lo, hi) => (v < lo ? lo : v > hi ? hi : v);

  /** 由弓位、仰角、蓄力量生成一支箭（纯函数）。 */
  function computeLaunch(origin, angleRad, power) {
    const speed = MIN_SPEED + (MAX_SPEED - MIN_SPEED) * clamp(power, 0, 1);
    return {
      x: origin.x,
      y: origin.y,
      vx: Math.cos(angleRad) * speed,
      vy: -Math.sin(angleRad) * speed
    };
  }

  /** 一步抛体运动积分（梯形法，对匀加速精确）。返回新对象，不修改入参。 */
  function stepArrow(arrow, dt, gravity) {
    const g = gravity === undefined ? GRAVITY : gravity;
    const vy = arrow.vy + g * dt;
    return {
      x: arrow.x + arrow.vx * dt,
      y: arrow.y + ((arrow.vy + vy) / 2) * dt,
      vx: arrow.vx,
      vy: vy
    };
  }

  /** 线段上距离某点最近的点。 */
  function closestOnSegment(x0, y0, x1, y1, px, py) {
    const dx = x1 - x0, dy = y1 - y0;
    const l2 = dx * dx + dy * dy;
    const t = clamp(l2 === 0 ? 0 : ((px - x0) * dx + (py - y0) * dy) / l2, 0, 1);
    const cx = x0 + t * dx, cy = y0 + t * dy;
    return { t, x: cx, y: cy, dist: Math.hypot(cx - px, cy - py) };
  }

  /** 线段与圆是否相交（用线段而非点做检测，避免高速穿透）。 */
  function segmentCircleHit(x0, y0, x1, y1, cx, cy, r) {
    return closestOnSegment(x0, y0, x1, y1, cx, cy).dist <= r;
  }

  /** 靶心在计时时刻 t（秒）的位置。 */
  function targetCenter(target, t) {
    return {
      x: target.x,
      y: target.baseY + Math.sin(t * target.bobSpeed + target.phase) * target.bobAmp
    };
  }

  /** 距靶心距离 -> 环数：靶心 5 环，最外圈 1 环，脱靶 0。 */
  function ringForDistance(dist, radius) {
    if (dist > radius) return 0;
    return clamp(Math.ceil((1 - dist / radius) * 5), 1, 5);
  }

  /** 本帧箭的位移线段是否命中某个未被命中的靶；取路径上最先命中的那个。 */
  function findHit(from, to, targets, t) {
    let best = null;
    for (const target of targets) {
      if (target.hit) continue;
      const c = targetCenter(target, t);
      const p = closestOnSegment(from.x, from.y, to.x, to.y, c.x, c.y);
      if (p.dist <= target.r && (best === null || p.t < best.at)) {
        best = { target, ring: ringForDistance(p.dist, target.r), x: p.x, y: p.y, at: p.t, center: c };
      }
    }
    return best;
  }

  const pad2 = (n) => String(n).padStart(2, '0');

  function formatTime(ms) {
    const total = Math.max(0, Math.round(ms));
    const m = Math.floor(total / 60000);
    const s = Math.floor((total % 60000) / 1000);
    const cs = Math.floor((total % 1000) / 10);
    return m > 0 ? `${m}:${pad2(s)}.${pad2(cs)}` : `${s}.${pad2(cs)}s`;
  }

  function sanitizeName(name) {
    const s = String(name == null ? '' : name).replace(/\s+/g, ' ').trim().slice(0, 12);
    return s || '匿名弓手';
  }

  /** 插入一条成绩：用时升序，同用时比箭数，再比名字。返回新榜单与排名（未入榜为 null）。 */
  function insertScore(board, entry, max) {
    const limit = max === undefined ? MAX_ENTRIES : max;
    const sorted = board.concat([entry]).sort((a, b) =>
      (a.timeMs - b.timeMs) || (a.arrows - b.arrows) || String(a.name).localeCompare(String(b.name))
    );
    const kept = sorted.slice(0, limit);
    const idx = kept.indexOf(entry);
    return { board: kept, rank: idx < 0 ? null : idx + 1 };
  }

  function validEntry(e) {
    return !!e && typeof e.timeMs === 'number' && isFinite(e.timeMs) && e.timeMs >= 0;
  }

  function loadBoard(storage) {
    const store = storage || (typeof localStorage !== 'undefined' ? localStorage : null);
    if (!store) return [];
    try {
      const raw = store.getItem(STORAGE_KEY);
      const data = raw ? JSON.parse(raw) : [];
      if (!Array.isArray(data)) return [];
      return data.filter(validEntry).map((e) => ({
        name: sanitizeName(e.name),
        timeMs: Math.round(e.timeMs),
        arrows: Number(e.arrows) || 0,
        rings: Number(e.rings) || 0,
        date: typeof e.date === 'string' ? e.date : ''
      })).slice(0, MAX_ENTRIES);
    } catch (err) {
      return [];
    }
  }

  function saveBoard(board, storage) {
    const store = storage || (typeof localStorage !== 'undefined' ? localStorage : null);
    if (!store) return false;
    try {
      store.setItem(STORAGE_KEY, JSON.stringify(board.slice(0, MAX_ENTRIES)));
      return true;
    } catch (err) {
      return false;
    }
  }

  return {
    GRAVITY, MIN_SPEED, MAX_SPEED, CHARGE_RATE, AIM_MIN, AIM_MAX,
    STORAGE_KEY, MAX_ENTRIES, LEVEL,
    clamp, computeLaunch, stepArrow, closestOnSegment, segmentCircleHit,
    targetCenter, ringForDistance, findHit, formatTime, sanitizeName,
    insertScore, loadBoard, saveBoard
  };
});
```

## archery/game.js

```js
/* 渲染、输入与游戏流程。逻辑计算全部委托给 ArcheryCore。 */
(function () {
  'use strict';

  const C = window.ArcheryCore;
  const L = C.LEVEL;

  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');
  canvas.width = L.width;
  canvas.height = L.height;

  const ui = {
    time: document.getElementById('hudTime'),
    hits: document.getElementById('hudHits'),
    arrows: document.getElementById('hudArrows'),
    rings: document.getElementById('hudRings'),
    power: document.getElementById('powerFill'),
    status: document.getElementById('status'),
    board: document.getElementById('board'),
    overlay: document.getElementById('overlay'),
    ovStats: document.getElementById('ovStats'),
    ovRank: document.getElementById('ovRank'),
    form: document.getElementById('scoreForm'),
    name: document.getElementById('playerName'),
    skip: document.getElementById('ovSkip'),
    again: document.getElementById('ovAgain'),
    restart: document.getElementById('restart'),
    clear: document.getElementById('clearBoard')
  };

  const RING_COLORS = ['#f4f2ea', '#22262f', '#2f7fd1', '#e2453c', '#f4c22b']; // 外 -> 内
  let board = C.loadBoard();
  let game;

  function reset() {
    game = {
      phase: 'ready',            // ready(未开表) -> timing(计时中) -> finished
      elapsed: 0,
      arrowsFired: 0,
      rings: 0,
      aim: 45 * Math.PI / 180,
      charging: false,
      power: 0,
      arrows: [],
      stuck: [],
      effects: [],
      targets: L.targets.map((t) => Object.assign({}, t, { hit: false, ring: 0, frozenY: null }))
    };
    ui.overlay.classList.add('hidden');
    ui.ovRank.textContent = '';
    ui.form.hidden = false;
    setStatus('瞄准后按住鼠标左键或空格键拉弓，松开射箭。计时从第一箭开始。');
    syncHud();
  }

  function setStatus(text) { ui.status.textContent = text; }

  function syncHud() {
    ui.time.textContent = C.formatTime(game.elapsed);
    ui.hits.textContent = `${game.targets.filter((t) => t.hit).length} / ${game.targets.length}`;
    ui.arrows.textContent = String(game.arrowsFired);
    ui.rings.textContent = String(game.rings);
    ui.power.style.width = `${Math.round(game.power * 100)}%`;
  }

  // ---------- 输入 ----------
  function startCharge() {
    if (game.phase === 'finished') return;
    game.charging = true;
    game.power = 0;
  }

  function fire() {
    if (!game.charging) return;
    const power = game.power;
    game.charging = false;
    game.power = 0;
    if (game.phase === 'finished') return;
    if (game.phase === 'ready') game.phase = 'timing';
    game.arrows.push(C.computeLaunch(L.bow, game.aim, power));
    game.arrowsFired += 1;
  }

  function aimAt(x, y) {
    const angle = Math.atan2(-(y - L.bow.y), x - L.bow.x);
    game.aim = C.clamp(angle, C.AIM_MIN, C.AIM_MAX);
  }

  function nudgeAim(dirSign) {
    game.aim = C.clamp(game.aim + dirSign * 1.2 * Math.PI / 180, C.AIM_MIN, C.AIM_MAX);
    setStatus(`仰角 ${(game.aim * 180 / Math.PI).toFixed(1)}°`);
  }

  function canvasPoint(clientX, clientY) {
    const r = canvas.getBoundingClientRect();
    return {
      x: (clientX - r.left) * canvas.width / r.width,
      y: (clientY - r.top) * canvas.height / r.height
    };
  }

  canvas.addEventListener('mousemove', (e) => {
    const p = canvasPoint(e.clientX, e.clientY);
    aimAt(p.x, p.y);
  });
  canvas.addEventListener('mousedown', (e) => {
    if (e.button !== 0) return;
    e.preventDefault();
    canvas.focus();
    startCharge();
  });
  window.addEventListener('mouseup', (e) => { if (e.button === 0) fire(); });

  canvas.addEventListener('touchstart', (e) => {
    e.preventDefault();
    const t = e.changedTouches[0];
    const p = canvasPoint(t.clientX, t.clientY);
    aimAt(p.x, p.y);
    startCharge();
  }, { passive: false });
  canvas.addEventListener('touchmove', (e) => {
    e.preventDefault();
    const t = e.changedTouches[0];
    const p = canvasPoint(t.clientX, t.clientY);
    aimAt(p.x, p.y);
  }, { passive: false });
  canvas.addEventListener('touchend', (e) => { e.preventDefault(); fire(); }, { passive: false });

  function typingInField() {
    const el = document.activeElement;
    return !!el && (el.tagName === 'INPUT' || el.tagName === 'TEXTAREA');
  }

  window.addEventListener('keydown', (e) => {
    if (typingInField()) return;
    if (e.code === 'Space') { e.preventDefault(); if (!e.repeat) startCharge(); }
    else if (e.code === 'ArrowUp') { e.preventDefault(); nudgeAim(1); }
    else if (e.code === 'ArrowDown') { e.preventDefault(); nudgeAim(-1); }
    else if (e.key === 'r' || e.key === 'R') reset();
  });
  window.addEventListener('keyup', (e) => {
    if (typingInField()) return;
    if (e.code === 'Space') { e.preventDefault(); fire(); }
  });

  // ---------- 更新 ----------
  function update(dt) {
    if (game.charging && game.phase !== 'finished') {
      game.power = Math.min(1, game.power + C.CHARGE_RATE * dt);
    }
    if (game.phase === 'timing') game.elapsed += dt * 1000;

    const steps = Math.max(1, Math.ceil(dt * 240));
    const h = dt / steps;
    for (let i = 0; i < steps; i++) simulate(h);

    for (const fx of game.effects) fx.life -= dt;
    game.effects = game.effects.filter((fx) => fx.life > 0);
    syncHud();
  }

  function simulate(dt) {
    const t = game.elapsed / 1000;
    const keep = [];
    for (const arrow of game.arrows) {
      const next = C.stepArrow(arrow, dt);
      const hit = C.findHit(arrow, next, game.targets, t);
      if (hit) { registerHit(hit, next); continue; }
      if (next.y >= L.groundY) {
        game.stuck.push({ x: next.x, y: L.groundY, angle: Math.atan2(next.vy, next.vx) });
        continue;
      }
      if (next.x < -80 || next.x > L.width + 80 || next.y > L.height + 200) continue;
      keep.push(next);
    }
    game.arrows = keep;
  }

  function registerHit(hit, arrow) {
    const target = hit.target;
    target.hit = true;
    target.ring = hit.ring;
    target.frozenY = hit.center.y;   // 命中后停止摆动，箭矢位置保持一致
    game.rings += hit.ring;
    game.stuck.push({ x: hit.x, y: hit.y, angle: Math.atan2(arrow.vy, arrow.vx) });
    game.effects.push({ x: hit.x, y: hit.y - 12, text: `${target.name} +${hit.ring}环`, life: 1.1, max: 1.1 });

    const left = game.targets.filter((t) => !t.hit).length;
    if (left === 0) finish();
    else setStatus(`命中${target.name}（${hit.ring} 环），还剩 ${left} 个靶子。`);
  }

  function finish() {
    game.phase = 'finished';
    game.charging = false;
    game.power = 0;
    const acc = game.arrowsFired ? Math.round(4 / game.arrowsFired * 100) : 0;
    ui.ovStats.textContent =
      `用时 ${C.formatTime(game.elapsed)} · 用箭 ${game.arrowsFired} 支 · 命中率 ${acc}% · 总环数 ${game.rings}`;
    ui.overlay.classList.remove('hidden');
    ui.name.value = ui.name.value || '';
    ui.name.focus();
    setStatus(`四靶全中！用时 ${C.formatTime(game.elapsed)}。可提交成绩到排行榜。`);
  }

  // ---------- 排行榜 ----------
  function renderBoard() {
    ui.board.innerHTML = '';
    if (!board.length) {
      const li = document.createElement('li');
      li.className = 'empty';
      li.textContent = '暂无成绩，快去射满四靶。';
      ui.board.appendChild(li);
      return;
    }
    board.forEach((e, i) => {
      const li = document.createElement('li');
      const name = document.createElement('span');
      name.className = 'name';
      name.textContent = `${i + 1}. ${e.name}`;
      const time = document.createElement('span');
      time.className = 'score';
      time.textContent = `${C.formatTime(e.timeMs)}（${e.arrows}箭/${e.rings}环）`;
      li.append(name, time);
      ui.board.appendChild(li);
    });
  }

  ui.form.addEventListener('submit', (e) => {
    e.preventDefault();
    const entry = {
      name: C.sanitizeName(ui.name.value),
      timeMs: Math.round(game.elapsed),
      arrows: game.arrowsFired,
      rings: game.rings,
      date: new Date().toISOString()
    };
    const result = C.insertScore(board, entry);
    board = result.board;
    const saved = C.saveBoard(board);
    renderBoard();
    ui.form.hidden = true;
    ui.ovRank.textContent = result.rank
      ? `第 ${result.rank} 名${result.rank === 1 ? '，新纪录！' : ''}${saved ? '' : '（本地存储不可用，成绩仅本次有效）'}`
      : `未进入前 ${C.MAX_ENTRIES} 名，再快一点！`;
    ui.again.focus();
  });

  ui.skip.addEventListener('click', () => { ui.form.hidden = true; ui.ovRank.textContent = '成绩未提交。'; ui.again.focus(); });
  ui.again.addEventListener('click', () => { reset(); canvas.focus(); });
  ui.restart.addEventListener('click', () => { reset(); canvas.focus(); });
  ui.clear.addEventListener('click', () => {
    if (!window.confirm('确定清空本机排行榜？此操作不可撤销。')) return;
    board = [];
    C.saveBoard(board);
    renderBoard();
  });

  // ---------- 绘制 ----------
  function drawBackground() {
    const sky = ctx.createLinearGradient(0, 0, 0, L.groundY);
    sky.addColorStop(0, '#1d3557');
    sky.addColorStop(1, '#7ba7c7');
    ctx.fillStyle = sky;
    ctx.fillRect(0, 0, L.width, L.groundY);

    ctx.fillStyle = 'rgba(255,255,255,0.85)';
    ctx.beginPath();
    ctx.arc(830, 90, 26, 0, Math.PI * 2);
    ctx.fill();

    ctx.fillStyle = '#4c6b7a';
    ctx.beginPath();
    ctx.moveTo(0, L.groundY);
    ctx.lineTo(180, 300); ctx.lineTo(340, L.groundY);
    ctx.moveTo(300, L.groundY);
    ctx.lineTo(520, 270); ctx.lineTo(760, L.groundY);
    ctx.moveTo(650, L.groundY);
    ctx.lineTo(880, 310); ctx.lineTo(L.width, L.groundY);
    ctx.fill();

    ctx.fillStyle = '#2f6b3a';
    ctx.fillRect(0, L.groundY, L.width, L.height - L.groundY);
    ctx.strokeStyle = 'rgba(255,255,255,0.12)';
    ctx.lineWidth = 1;
    for (let x = 20; x < L.width; x += 40) {
      ctx.beginPath();
      ctx.moveTo(x, L.groundY);
      ctx.lineTo(x + 8, L.groundY - 8);
      ctx.stroke();
    }
  }

  function drawTargets() {
    const t = game.elapsed / 1000;
    for (const target of game.targets) {
      const c = target.hit
        ? { x: target.x, y: target.frozenY }
        : C.targetCenter(target, t);

      ctx.strokeStyle = '#6b4a2b';
      ctx.lineWidth = 6;
      ctx.beginPath();
      ctx.moveTo(c.x, c.y);
      ctx.lineTo(c.x, L.groundY);
      ctx.stroke();

      for (let i = 0; i < RING_COLORS.length; i++) {
        ctx.fillStyle = RING_COLORS[i];
        ctx.beginPath();
        ctx.arc(c.x, c.y, target.r * (1 - i * 0.2), 0, Math.PI * 2);
        ctx.fill();
      }
      ctx.strokeStyle = 'rgba(0,0,0,0.45)';
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.arc(c.x, c.y, target.r, 0, Math.PI * 2);
      ctx.stroke();

      if (target.hit) {
        ctx.fillStyle = 'rgba(20,120,60,0.45)';
        ctx.beginPath();
        ctx.arc(c.x, c.y, target.r, 0, Math.PI * 2);
        ctx.fill();
        ctx.fillStyle = '#eaffef';
        ctx.font = 'bold 20px system-ui, sans-serif';
        ctx.textAlign = 'center';
        ctx.fillText('✓', c.x, c.y + 7);
      }

      ctx.fillStyle = 'rgba(255,255,255,0.9)';
      ctx.font = '12px system-ui, sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText(target.name, c.x, L.groundY + 18);
    }
  }

  function drawArrowShape(x, y, angle, length) {
    ctx.save();
    ctx.translate(x, y);
    ctx.rotate(angle);
    ctx.strokeStyle = '#d9c8a5';
    ctx.lineWidth = 2.5;
    ctx.beginPath();
    ctx.moveTo(-length, 0);
    ctx.lineTo(0, 0);
    ctx.stroke();
    ctx.fillStyle = '#e9eef5';
    ctx.beginPath();
    ctx.moveTo(0, 0);
    ctx.lineTo(-9, -4);
    ctx.lineTo(-9, 4);
    ctx.closePath();
    ctx.fill();
    ctx.strokeStyle = '#c0392b';
    ctx.beginPath();
    ctx.moveTo(-length, 0);
    ctx.lineTo(-length + 7, -4);
    ctx.moveTo(-length, 0);
    ctx.lineTo(-length + 7, 4);
    ctx.stroke();
    ctx.restore();
  }

  function drawFlying() {
    for (const a of game.arrows) drawArrowShape(a.x, a.y, Math.atan2(a.vy, a.vx), 26);
    for (const s of game.stuck) drawArrowShape(s.x, s.y, s.angle, 22);
  }

  function drawBow() {
    ctx.save();
    ctx.translate(L.bow.x, L.bow.y);
    ctx.rotate(-game.aim);

    const span = Math.PI / 2.2;
    ctx.strokeStyle = '#8b5a2b';
    ctx.lineWidth = 5;
    ctx.beginPath();
    ctx.arc(0, 0, 34, -span, span);
    ctx.stroke();

    const pull = 6 + game.power * 24;
    const tipX = Math.cos(span) * 34, tipY = Math.sin(span) * 34;
    ctx.strokeStyle = '#eae6d8';
    ctx.lineWidth = 1.5;
    ctx.beginPath();
    ctx.moveTo(tipX, -tipY);
    ctx.lineTo(-pull, 0);
    ctx.lineTo(tipX, tipY);
    ctx.stroke();

    if (game.charging) drawArrowShape(46, 0, 0, 46 + pull);
    ctx.restore();

    ctx.strokeStyle = 'rgba(255,255,255,0.25)';
    ctx.setLineDash([4, 6]);
    ctx.lineWidth = 1;
    ctx.beginPath();
    ctx.moveTo(L.bow.x, L.bow.y);
    ctx.lineTo(L.bow.x + Math.cos(game.aim) * 120, L.bow.y - Math.sin(game.aim) * 120);
    ctx.stroke();
    ctx.setLineDash([]);
  }

  function drawPreview() {
    if (!game.charging) return;
    let a = C.computeLaunch(L.bow, game.aim, game.power);
    ctx.fillStyle = 'rgba(255,255,255,0.35)';
    for (let i = 0; i < 90; i++) {
      a = C.stepArrow(a, 1 / 60);
      if (a.y >= L.groundY || a.x > L.width) break;
      if (i % 6 === 0) {
        ctx.beginPath();
        ctx.arc(a.x, a.y, 2, 0, Math.PI * 2);
        ctx.fill();
      }
    }
  }

  function drawEffects() {
    ctx.textAlign = 'center';
    ctx.font = 'bold 16px system-ui, sans-serif';
    for (const fx of game.effects) {
      const k = fx.life / fx.max;
      ctx.fillStyle = `rgba(244,194,43,${k.toFixed(2)})`;
      ctx.fillText(fx.text, fx.x, fx.y - (1 - k) * 28);
    }
  }

  function render() {
    ctx.clearRect(0, 0, L.width, L.height);
    drawBackground();
    drawTargets();
    drawPreview();
    drawFlying();
    drawBow();
    drawEffects();
  }

  // ---------- 主循环 ----------
  let last = performance.now();
  function frame(now) {
    const dt = Math.min(0.05, (now - last) / 1000);
    last = now;
    update(dt);
    render();
    requestAnimationFrame(frame);
  }

  reset();
  renderBoard();
  requestAnimationFrame(frame);
})();
```

## archery/index.html

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>弓箭射击模拟 · 四靶竞速</title>
<style>
  :root { color-scheme: dark; }
  * { box-sizing: border-box; }
  body {
    margin: 0; padding: 20px;
    font-family: system-ui, "Segoe UI", "PingFang SC", "Microsoft YaHei", sans-serif;
    background: #10141c; color: #e8ecf3;
  }
  h1 { font-size: 20px; margin: 0 0 12px; }
  .wrap { display: flex; gap: 18px; flex-wrap: wrap; align-items: flex-start; max-width: 1360px; margin: 0 auto; }
  .stage { flex: 1 1 640px; min-width: 320px; }
  canvas {
    width: 100%; height: auto; display: block; border-radius: 10px;
    border: 1px solid #2a3242; background: #0b0f16; cursor: crosshair; touch-action: none;
  }
  canvas:focus-visible { outline: 3px solid #f4c22b; outline-offset: 2px; }
  .hud { display: flex; gap: 16px; flex-wrap: wrap; align-items: center; margin: 10px 0; font-variant-numeric: tabular-nums; }
  .hud div span { color: #9fb0c6; font-size: 12px; display: block; }
  .hud div b { font-size: 18px; }
  .bar { flex: 1 1 160px; height: 12px; min-width: 120px; background: #222b39; border-radius: 6px; overflow: hidden; }
  .bar i { display: block; height: 100%; width: 0; background: linear-gradient(90deg, #4caf50, #f4c22b, #e2453c); }
  .side { flex: 0 0 300px; }
  .card { background: #161d28; border: 1px solid #26303f; border-radius: 10px; padding: 14px; margin-bottom: 14px; }
  .card h2 { font-size: 15px; margin: 0 0 10px; }
  ol#board { list-style: none; margin: 0; padding: 0; font-variant-numeric: tabular-nums; }
  ol#board li { display: flex; justify-content: space-between; gap: 8px; padding: 6px 0; border-bottom: 1px dashed #26303f; font-size: 13px; }
  ol#board li:first-child .score { color: #f4c22b; }
  ol#board li.empty { color: #8595ab; justify-content: center; }
  ul.keys { margin: 0; padding-left: 18px; font-size: 13px; line-height: 1.7; color: #c3cede; }
  kbd { background: #222b39; border: 1px solid #3a465a; border-radius: 4px; padding: 0 5px; font-size: 12px; }
  button { font: inherit; padding: 7px 12px; border-radius: 7px; border: 1px solid #3a465a; background: #222b39; color: #e8ecf3; cursor: pointer; }
  button:hover { background: #2c384a; }
  button[type=submit] { background: #2f7fd1; border-color: #2f7fd1; }
  #status { min-height: 20px; font-size: 13px; color: #9fb0c6; margin: 6px 0 0; }
  .overlay { position: fixed; inset: 0; background: rgba(6,9,14,0.78); display: flex; align-items: center; justify-content: center; padding: 20px; z-index: 10; }
  .overlay.hidden { display: none; }
  .panel { background: #161d28; border: 1px solid #33415a; border-radius: 12px; padding: 22px; max-width: 420px; width: 100%; }
  .panel h2 { margin: 0 0 8px; }
  .panel input { width: 100%; padding: 8px; margin: 6px 0 12px; border-radius: 7px; border: 1px solid #3a465a; background: #0f141d; color: inherit; }
  .row { display: flex; gap: 8px; flex-wrap: wrap; }
</style>
</head>
<body>
<div class="wrap">
  <main class="stage">
    <h1>弓箭射击模拟 · 四靶竞速</h1>
    <canvas id="game" width="960" height="540" tabindex="0"
            aria-label="射箭场地：弓在左侧，四个靶子在右侧。使用方向键调整仰角，空格键拉弓与射箭。"></canvas>
    <div class="hud">
      <div><span>用时</span><b id="hudTime">0.00s</b></div>
      <div><span>命中靶数</span><b id="hudHits">0 / 4</b></div>
      <div><span>用箭</span><b id="hudArrows">0</b></div>
      <div><span>总环数</span><b id="hudRings">0</b></div>
      <div class="bar" role="img" aria-label="拉弓力度"><i id="powerFill"></i></div>
      <button id="restart" type="button">重新开始 (R)</button>
    </div>
    <p id="status" role="status" aria-live="polite"></p>
  </main>

  <aside class="side">
    <section class="card">
      <h2>排行榜（用时越短越靠前）</h2>
      <ol id="board"></ol>
      <div class="row" style="margin-top:10px">
        <button id="clearBoard" type="button">清空本机排行榜</button>
      </div>
    </section>
    <section class="card">
      <h2>玩法</h2>
      <ul class="keys">
        <li>目标：尽快命中全部 <b>4 个靶子</b>，用时最短者排名最高。</li>
        <li>计时从<b>射出第一箭</b>时开始，第四靶命中即停表；箭矢数量不限。</li>
        <li>鼠标移动瞄准，按住左键蓄力，松开射箭。</li>
        <li>键盘：<kbd>↑</kbd>/<kbd>↓</kbd> 调整仰角，按住 <kbd>空格</kbd> 蓄力、松开射箭，<kbd>R</kbd> 重开。</li>
        <li>靶子会上下摆动（近靶固定），越远越小、摆幅越大；靶心 5 环，最外圈 1 环。</li>
        <li>关卡固定、无随机风力，因此排行榜可比。成绩仅保存在你本机浏览器中。</li>
      </ul>
    </section>
  </aside>
</div>

<div id="overlay" class="overlay hidden" role="dialog" aria-modal="true" aria-labelledby="ovTitle">
  <div class="panel">
    <h2 id="ovTitle">四靶全中！</h2>
    <p id="ovStats"></p>
    <form id="scoreForm">
      <label for="playerName">选手名字（最多 12 字）</label>
      <input id="playerName" name="playerName" maxlength="12" autocomplete="off" placeholder="匿名弓手">
      <div class="row">
        <button type="submit">提交成绩</button>
        <button type="button" id="ovSkip">不提交</button>
      </div>
    </form>
    <p id="ovRank" role="status" aria-live="polite"></p>
    <button id="ovAgain" type="button">再来一局 (R)</button>
  </div>
</div>

<script src="core.js"></script>
<script src="game.js"></script>
</body>
</html>
```

## archery/tests.js

```js
/* 核心逻辑单元测试：可用 `node tests.js` 运行，也可在浏览器打开 tests.html 运行。 */
(function () {
  'use strict';

  const isNode = typeof window === 'undefined';
  const Core = isNode ? require('./core.js') : window.ArcheryCore;

  let pass = 0, fail = 0;
  const lines = [];
  function print(s) { lines.push(s); if (isNode) console.log(s); }
  function ok(name, cond, extra) {
    if (cond) { pass++; print('  ✓ ' + name); }
    else { fail++; print('  ✗ ' + name + (extra ? ' -> ' + extra : '')); }
  }
  const near = (a, b, eps) => Math.abs(a - b) <= (eps === undefined ? 1e-6 : eps);

  print('computeLaunch');
  {
    const bow = { x: 0, y: 0 };
    const weak = Core.computeLaunch(bow, 0, 0);
    const full = Core.computeLaunch(bow, 0, 1);
    ok('power=0 使用最小初速', near(weak.vx, Core.MIN_SPEED) && near(weak.vy, 0));
    ok('power=1 使用最大初速', near(full.vx, Core.MAX_SPEED));
    ok('power 被裁剪到 [0,1]', near(Core.computeLaunch(bow, 0, 5).vx, Core.MAX_SPEED));
    const up = Core.computeLaunch(bow, Math.PI / 2, 1);
    ok('向上射箭 vy 为负（屏幕坐标向下为正）', up.vy < 0 && near(up.vx, 0, 1e-9));
  }

  print('stepArrow');
  {
    const a = { x: 0, y: 0, vx: 100, vy: 0 };
    const b = Core.stepArrow(a, 1, 1000);
    ok('速度受重力影响', near(b.vy, 1000));
    ok('水平位移 = vx * dt', near(b.x, 100));
    ok('垂直位移使用梯形积分（精确 500）', near(b.y, 500));
    ok('入参未被修改（纯函数）', a.vy === 0 && a.y === 0);
    ok('默认重力为 GRAVITY', near(Core.stepArrow(a, 1).vy, Core.GRAVITY));
  }

  print('segmentCircleHit / closestOnSegment');
  {
    ok('线段穿过圆心 -> 命中', Core.segmentCircleHit(0, 0, 100, 0, 50, 0, 10));
    ok('线段擦过圆外 -> 未命中', !Core.segmentCircleHit(0, 0, 100, 0, 50, 40, 10));
    ok('两端点都在圆外但穿过圆 -> 仍然命中（防穿透）',
      Core.segmentCircleHit(-100, 0, 100, 0, 0, 5, 10));
    ok('圆在线段延长线上 -> 未命中', !Core.segmentCircleHit(0, 0, 10, 0, 200, 0, 10));
    ok('最近点参数 t 被限制在 [0,1]', Core.closestOnSegment(0, 0, 10, 0, 100, 0).t === 1);
  }

  print('ringForDistance');
  {
    ok('正中靶心 = 5 环', Core.ringForDistance(0, 40) === 5);
    ok('靶心圈边界 = 5 环', Core.ringForDistance(8, 40) === 5);
    ok('中间位置 = 3 环', Core.ringForDistance(20, 40) === 3);
    ok('最外缘 = 1 环', Core.ringForDistance(40, 40) === 1);
    ok('脱靶 = 0 环', Core.ringForDistance(40.1, 40) === 0);
  }

  print('targetCenter');
  {
    const still = Core.LEVEL.targets[0];
    const moving = Core.LEVEL.targets[3];
    ok('近靶不摆动', Core.targetCenter(still, 0).y === Core.targetCenter(still, 3.7).y);
    ok('远靶随时间摆动', Core.targetCenter(moving, 0).y !== Core.targetCenter(moving, 0.8).y);
    ok('摆动幅度不超过 bobAmp',
      Math.abs(Core.targetCenter(moving, 1.234).y - moving.baseY) <= moving.bobAmp + 1e-9);
  }

  print('findHit');
  {
    const targets = [
      { id: 1, name: 'A', x: 100, baseY: 0, r: 10, bobAmp: 0, bobSpeed: 0, phase: 0, hit: false },
      { id: 2, name: 'B', x: 200, baseY: 0, r: 10, bobAmp: 0, bobSpeed: 0, phase: 0, hit: false }
    ];
    const from = { x: 0, y: 0 }, to = { x: 300, y: 0 };
    const first = Core.findHit(from, to, targets, 0);
    ok('返回路径上最先命中的靶', first && first.target.id === 1);
    ok('返回环数与命中点', first && first.ring === 5 && typeof first.x === 'number');
    targets[0].hit = true;
    const second = Core.findHit(from, to, targets, 0);
    ok('已命中的靶被跳过', second && second.target.id === 2);
    ok('未命中任何靶时返回 null', Core.findHit({ x: 0, y: 500 }, { x: 300, y: 500 }, targets, 0) === null);
  }

  print('formatTime');
  {
    ok('0 -> 0.00s', Core.formatTime(0) === '0.00s');
    ok('1234 -> 1.23s', Core.formatTime(1234) === '1.23s');
    ok('65432 -> 1:05.43', Core.formatTime(65432) === '1:05.43');
    ok('负数按 0 处理', Core.formatTime(-5) === '0.00s');
  }

  print('insertScore（排行榜规则：用时升序）');
  {
    const base = [
      { name: 'a', timeMs: 5000, arrows: 4, rings: 20 },
      { name: 'b', timeMs: 9000, arrows: 6, rings: 15 }
    ];
    const faster = { name: 'c', timeMs: 3000, arrows: 5, rings: 18 };
    const r1 = Core.insertScore(base, faster, 10);
    ok('更快的成绩排在最前', r1.board[0] === faster && r1.rank === 1);
    ok('原榜单未被修改', base.length === 2);

    const tie = { name: 'd', timeMs: 5000, arrows: 4, rings: 25 };
    const r2 = Core.insertScore(base, tie, 10);
    ok('用时相同时按名字稳定排序（a 在 d 前）',
      r2.board.findIndex((e) => e.name === 'a') < r2.board.findIndex((e) => e.name === 'd'));

    const fewerArrows = { name: 'e', timeMs: 5000, arrows: 2, rings: 25 };
    const r3 = Core.insertScore(base, fewerArrows, 10);
    ok('用时相同时用箭少者更靠前', r3.board[0] === fewerArrows);

    const full = [];
    for (let i = 0; i < 10; i++) full.push({ name: 'p' + i, timeMs: 1000 + i, arrows: 4, rings: 20 });
    const slow = { name: 'slow', timeMs: 99999, arrows: 9, rings: 10 };
    const r4 = Core.insertScore(full, slow, 10);
    ok('榜单长度受 max 限制', r4.board.length === 10);
    ok('未入榜时 rank 为 null', r4.rank === null);
  }

  print('sanitizeName');
  {
    ok('去除首尾空白', Core.sanitizeName('  神箭手  ') === '神箭手');
    ok('空名回退为默认值', Core.sanitizeName('   ') === '匿名弓手');
    ok('null 回退为默认值', Core.sanitizeName(null) === '匿名弓手');
    ok('长度截断到 12', Core.sanitizeName('123456789012345').length === 12);
  }

  print('loadBoard / saveBoard（内存 storage 桩）');
  {
    const mem = {
      data: {},
      getItem(k) { return Object.prototype.hasOwnProperty.call(this.data, k) ? this.data[k] : null; },
      setItem(k, v) { this.data[k] = String(v); }
    };
    ok('空存储返回空数组', Core.loadBoard(mem).length === 0);
    Core.saveBoard([{ name: '弓手', timeMs: 4321.6, arrows: 5, rings: 19, date: '2026-01-01' }], mem);
    const loaded = Core.loadBoard(mem);
    ok('写入后可读回并四舍五入用时', loaded.length === 1 && loaded[0].timeMs === 4322);
    mem.data[Core.STORAGE_KEY] = '{ not json';
    ok('损坏数据不抛异常', Core.loadBoard(mem).length === 0);
    mem.data[Core.STORAGE_KEY] = JSON.stringify([{ name: 'x' }, { name: 'y', timeMs: 10, arrows: 1, rings: 5 }]);
    ok('过滤非法记录', Core.loadBoard(mem).length === 1);
    const broken = { getItem() { throw new Error('blocked'); }, setItem() { throw new Error('blocked'); } };
    ok('存储不可用时安全降级', Core.loadBoard(broken).length === 0 && Core.saveBoard([], broken) === false);
  }

  print('关卡可解性（物理仿真：四靶都能被满弓射中）');
  {
    function hits(target, angleDeg, power) {
      const targets = [Object.assign({}, target, { hit: false })];
      let arrow = Core.computeLaunch(Core.LEVEL.bow, angleDeg * Math.PI / 180, power);
      for (let i = 0; i < 240 * 8; i++) {
        const next = Core.stepArrow(arrow, 1 / 240);
        if (Core.findHit(arrow, next, targets, 0)) return true;
        if (next.y >= Core.LEVEL.groundY || next.x > Core.LEVEL.width + 100) return false;
        arrow = next;
      }
      return false;
    }
    for (const target of Core.LEVEL.targets) {
      let found = null;
      for (let deg = 0; deg <= 85 && found === null; deg += 0.25) {
        if (hits(target, deg, 1)) found = deg;
      }
      ok(`${target.name} 存在可命中的仰角`, found !== null, found === null ? '无解' : `仰角 ${found}°`);
    }
    ok('并非所有仰角都能命中（检测有效）', !hits(Core.LEVEL.targets[3], 85, 1));
    ok('空拉射不到最近的靶', !hits(Core.LEVEL.targets[0], 45, 0));
  }

  const summary = `\n结果：${pass} 通过，${fail} 失败`;
  print(summary);

  if (isNode) {
    process.exitCode = fail === 0 ? 0 : 1;
  } else {
    const out = document.getElementById('out');
    out.textContent = lines.join('\n');
    const badge = document.getElementById('badge');
    badge.textContent = fail === 0 ? `全部通过（${pass}）` : `${fail} 项失败`;
    badge.className = fail === 0 ? 'ok' : 'bad';
  }
})();
```

## archery/tests.html

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="utf-8">
<title>核心逻辑测试 · 弓箭射击模拟</title>
<style>
  body { font-family: ui-monospace, Menlo, Consolas, monospace; background: #10141c; color: #e8ecf3; padding: 20px; }
  #badge { display: inline-block; padding: 4px 10px; border-radius: 6px; font-weight: 700; }
  #badge.ok { background: #1f7a3f; }
  #badge.bad { background: #a12a22; }
  pre { white-space: pre-wrap; line-height: 1.6; }
  a { color: #6db3f2; }
</style>
</head>
<body>
  <h1>核心逻辑测试</h1>
  <p><span id="badge">运行中…</span> · <a href="index.html">返回游戏</a></p>
  <pre id="out"></pre>
  <script src="core.js"></script>
  <script src="tests.js"></script>
</body>
</html>
```

## 运行方式

把 5 个文件放进同一目录，然后：

- 最简单：双击 `index.html`（`file://` 下也能运行，脚本是传统 script，没有模块跨域限制；排行榜用 localStorage，浏览器对 `file://` 的策略可能不同）。
- 推荐：`cd archery && python3 -m http.server 8000`，浏览器打开 `http://localhost:8000/`。这是纯静态本地服务，不需要账号也不上传任何数据；排行榜只写入本机浏览器的 localStorage，没有后端与网络请求。

操作：鼠标移动瞄准，按住左键蓄力、松开射箭；键盘 `↑`/`↓` 调仰角，按住 `空格` 蓄力、松开射箭，`R` 重开。计时从第一箭射出开始，命中第四个靶即停表，随后可填名字提交到排行榜（按用时升序，同用时比用箭数，保留前 10）。

## 测试说明

自动化（35+ 条断言，覆盖抛体积分、线段-圆命中判定与防穿透、环数换算、靶心摆动、排行榜排序与容量、名字清洗、存储降级，以及用物理仿真验证四个靶子都存在可命中的仰角）：

```bash
cd archery
node tests.js        # 全部通过时退出码 0
```

或浏览器打开 `tests.html` 查看逐条结果。

手动验收清单：
1. 首次进入计时器停在 `0.00s`，射出第一箭后开始走动。
2. 命中一个靶后该靶显示 ✓ 并停止摆动，剩余靶数在状态栏播报。
3. 四靶全中立即停表，弹出结算框；提交后排行榜出现该成绩且按用时排序正确。
4. 刷新页面排行榜仍在（localStorage 持久化）；「清空本机排行榜」需二次确认。
5. 全程仅用键盘也能完成一局（Tab 聚焦画布 → `↑`/`↓` → `空格`），状态变化由 `aria-live` 区域播报。

我在本次会话中无法执行命令，所以上面的 `node tests.js` 我没有实际跑过；测试代码本身是自校验的，如有断言失败请把输出发我，我来修。