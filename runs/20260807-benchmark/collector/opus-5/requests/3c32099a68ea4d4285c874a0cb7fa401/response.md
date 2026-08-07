下面是完整可运行的交付物：一个基于 Three.js 的真 3D 腕表（GMT 双时区），指针按系统当前时间连续平滑走动，带日历/星期窗口和 6 点位 24 小时副表盘。时间逻辑被抽成纯模块，可用 `node --test` 直接测试。

## 目录结构

```
kiro-3d-watch/
├── index.html                  # 入口 + UI 覆盖层
├── package.json
├── README.md
├── src/
│   ├── timekeeping.js          # 纯时间/时区/角度计算（无 DOM，可测试）
│   ├── textures.js             # Canvas 生成表盘/外圈/副盘/窗口/环境贴图
│   ├── watchModel.js           # 3D 腕表建模（表壳、镜面、指针、表链）
│   └── main.js                 # 场景、渲染循环、交互、UI 绑定
└── test/
    └── timekeeping.test.mjs    # 单元测试（含 DST、半小时时区、角度）
```

## src/timekeeping.js

```js
/**
 * 纯时间计算模块：无 DOM 依赖，可在 Node 中直接单元测试。
 * 所有角度均为「从 12 点位起、顺时针方向」的弧度值。
 */

export const TAU = Math.PI * 2;

export const WEEKDAY_EN = ['SUN', 'MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT'];
export const WEEKDAY_ZH = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];

const formatterCache = new Map();

function formatterFor(timeZone) {
  let f = formatterCache.get(timeZone);
  if (!f) {
    f = new Intl.DateTimeFormat('en-US', {
      timeZone,
      hourCycle: 'h23',
      year: 'numeric',
      month: '2-digit',
      day: '2-digit',
      hour: '2-digit',
      minute: '2-digit',
      second: '2-digit',
    });
    formatterCache.set(timeZone, f);
  }
  return f;
}

/** 校验 IANA 时区名是否被当前运行时支持。 */
export function isValidTimeZone(timeZone) {
  if (typeof timeZone !== 'string' || timeZone.length === 0) return false;
  try {
    new Intl.DateTimeFormat('en-US', { timeZone });
    return true;
  } catch {
    return false;
  }
}

/**
 * 把一个瞬间（Date）换算到指定时区的日历字段。
 * @returns {{timeZone:string, year:number, month:number, day:number,
 *            hour:number, minute:number, second:number, ms:number,
 *            weekday:number, offsetMinutes:number, epochMs:number}}
 */
export function zonedTime(date = new Date(), timeZone = 'UTC') {
  const parts = formatterFor(timeZone).formatToParts(date);
  const v = Object.create(null);
  for (const p of parts) {
    if (p.type !== 'literal') v[p.type] = p.value;
  }

  const year = Number(v.year);
  const month = Number(v.month);
  const day = Number(v.day);
  const hour = Number(v.hour) % 24; // 防御某些实现返回 24
  const minute = Number(v.minute);
  const second = Number(v.second);
  const ms = date.getMilliseconds();

  // 与语言无关的星期计算（0=周日）
  const weekday = new Date(Date.UTC(year, month - 1, day)).getUTCDay();

  // 该瞬间在该时区的 UTC 偏移（分钟）
  const asUTC = Date.UTC(year, month - 1, day, hour, minute, second);
  const offsetMinutes = Math.round((asUTC - (date.getTime() - ms)) / 60000);

  return {
    timeZone,
    year,
    month,
    day,
    hour,
    minute,
    second,
    ms,
    weekday,
    offsetMinutes,
    epochMs: date.getTime(),
  };
}

/**
 * 由时间字段计算各指针角度。
 * @param t zonedTime() 的返回值
 * @param opts.sweep      true = 连续平滑扫秒（默认）；false = 整秒跳动
 * @param opts.beatsPerSecond >0 时把秒的小数部分量化为机械表的「每秒 N 跳」
 */
export function handAngles(t, opts = {}) {
  const sweep = opts.sweep !== false;
  const beats = Number(opts.beatsPerSecond) || 0;

  let frac = t.ms / 1000;
  if (!sweep) frac = 0;
  else if (beats > 0) frac = Math.floor(frac * beats) / beats;

  const seconds = t.second + frac;
  const minutes = t.minute + seconds / 60;
  const hours12 = (t.hour % 12) + minutes / 60;
  const hours24 = t.hour + minutes / 60;

  return {
    second: (seconds / 60) * TAU,
    minute: (minutes / 60) * TAU,
    hour: (hours12 / 12) * TAU,
    hour24: (hours24 / 24) * TAU,
  };
}

export function pad2(n) {
  return String(n).padStart(2, '0');
}

/** 481 -> "UTC+08:01"，-240 -> "UTC-04:00" */
export function formatOffset(minutes) {
  const sign = minutes < 0 ? '-' : '+';
  const abs = Math.abs(minutes);
  return `UTC${sign}${pad2(Math.floor(abs / 60))}:${pad2(abs % 60)}`;
}

/** 两地时差的可读表示：0 -> "同时区"，-330 -> "-5h30m" */
export function formatOffsetDelta(minutes) {
  if (minutes === 0) return '同时区';
  const sign = minutes < 0 ? '-' : '+';
  const abs = Math.abs(minutes);
  const h = Math.floor(abs / 60);
  const m = abs % 60;
  return `${sign}${h}h${m ? `${pad2(m)}m` : ''}`;
}

export function formatClock(t, withSeconds = true) {
  return withSeconds
    ? `${pad2(t.hour)}:${pad2(t.minute)}:${pad2(t.second)}`
    : `${pad2(t.hour)}:${pad2(t.minute)}`;
}

export function formatISODate(t) {
  return `${t.year}-${pad2(t.month)}-${pad2(t.day)}`;
}

/** 民用日期差（天）：b 相对 a。用于「第二时区已经是明天」这类提示。 */
export function dayDelta(a, b) {
  const da = Date.UTC(a.year, a.month - 1, a.day);
  const db = Date.UTC(b.year, b.month - 1, b.day);
  return Math.round((db - da) / 86400000);
}

/** 'America/New_York' -> 'NEW YORK'，用于副表盘上的短标签。 */
export function shortZoneLabel(timeZone, maxLen = 12) {
  const tail = String(timeZone).split('/').pop() || String(timeZone);
  const label = tail.replace(/_/g, ' ').toUpperCase();
  return label.length > maxLen ? `${label.slice(0, maxLen - 1)}.` : label;
}
```

## src/textures.js

```js
import * as THREE from 'three';

/* ---------- Canvas 工具：以毫米为单位、+Y 朝上作图 ---------- */

function mmCanvas(sizePx, radiusMM) {
  const canvas = document.createElement('canvas');
  canvas.width = sizePx;
  canvas.height = sizePx;
  const ctx = canvas.getContext('2d');
  const scale = sizePx / (radiusMM * 2);
  ctx.translate(sizePx / 2, sizePx / 2);
  ctx.scale(scale, -scale); // 逻辑坐标系为右手系：+Y 向上
  return { canvas, ctx };
}

function toTexture(canvas) {
  const tex = new THREE.CanvasTexture(canvas);
  tex.colorSpace = THREE.SRGBColorSpace;
  tex.anisotropy = 8;
  return tex;
}

/** 由于绘图坐标系做了 Y 翻转，文字必须再翻转一次才不会镜像。 */
function drawText(ctx, str, o = {}) {
  const {
    x = 0, y = 0, size = 1, weight = 500, color = '#ffffff',
    align = 'center', rotate = 0, spacing = 0,
    family = '"Helvetica Neue", Helvetica, Arial, sans-serif',
  } = o;
  ctx.save();
  ctx.translate(x, y);
  if (rotate) ctx.rotate(rotate);
  ctx.scale(1, -1);
  ctx.font = `${weight} ${size}px ${family}`;
  ctx.fillStyle = color;
  ctx.textBaseline = 'middle';
  if (spacing) {
    ctx.textAlign = 'left';
    const chars = [...str];
    const widths = chars.map((c) => ctx.measureText(c).width);
    const total = widths.reduce((a, b) => a + b, 0) + spacing * (chars.length - 1);
    let cx = align === 'center' ? -total / 2 : 0;
    chars.forEach((c, i) => {
      ctx.fillText(c, cx, 0);
      cx += widths[i] + spacing;
    });
  } else {
    ctx.textAlign = align;
    ctx.fillText(str, 0, 0);
  }
  ctx.restore();
}

function roundRect(ctx, cx, cy, w, h, r) {
  const x = cx - w / 2;
  const y = cy - h / 2;
  const rr = Math.min(r, w / 2, h / 2);
  ctx.beginPath();
  ctx.moveTo(x + rr, y);
  ctx.lineTo(x + w - rr, y);
  ctx.quadraticCurveTo(x + w, y, x + w, y + rr);
  ctx.lineTo(x + w, y + h - rr);
  ctx.quadraticCurveTo(x + w, y + h, x + w - rr, y + h);
  ctx.lineTo(x + rr, y + h);
  ctx.quadraticCurveTo(x, y + h, x, y + h - rr);
  ctx.lineTo(x, y + rr);
  ctx.quadraticCurveTo(x, y, x + rr, y);
  ctx.closePath();
}

/** 太阳放射纹 */
function sunburst(ctx, radius, rays = 360) {
  for (let i = 0; i < rays; i++) {
    const a0 = (i / rays) * Math.PI * 2;
    const a1 = ((i + 1) / rays) * Math.PI * 2;
    ctx.beginPath();
    ctx.moveTo(0, 0);
    ctx.arc(0, 0, radius, a0, a1);
    ctx.closePath();
    ctx.fillStyle = i % 2 ? 'rgba(255,255,255,0.040)' : 'rgba(0,0,0,0.055)';
    ctx.fill();
  }
}

/* ---------- 主表盘 ---------- */

export const DIAL_LAYOUT = {
  radius: 16.5,
  dayWindow: { x: 0, y: 9.2, w: 8.6, h: 3.2 },   // 12 点位：星期
  dateWindow: { x: 9.1, y: 0, w: 4.4, h: 3.4 },  // 3 点位：日期
  subdial: { x: 0, y: -8.6, r: 5.0 },            // 6 点位：第二时区 24 小时盘
};

export function createDialTexture({ size = 2048 } = {}) {
  const R = DIAL_LAYOUT.radius;
  const { canvas, ctx } = mmCanvas(size, R);

  // 底色
  const base = ctx.createRadialGradient(0, 1.5, 0.5, 0, 0, R);
  base.addColorStop(0, '#1d5384');
  base.addColorStop(0.55, '#123a60');
  base.addColorStop(1, '#07182a');
  ctx.beginPath();
  ctx.arc(0, 0, R, 0, Math.PI * 2);
  ctx.fillStyle = base;
  ctx.fill();

  ctx.save();
  ctx.beginPath();
  ctx.arc(0, 0, R, 0, Math.PI * 2);
  ctx.clip();
  sunburst(ctx, R);
  const vig = ctx.createRadialGradient(0, 0, R * 0.5, 0, 0, R);
  vig.addColorStop(0, 'rgba(0,0,0,0)');
  vig.addColorStop(1, 'rgba(0,0,0,0.55)');
  ctx.fillStyle = vig;
  ctx.fillRect(-R, -R, R * 2, R * 2);
  ctx.restore();

  // 分钟刻度轨道
  ctx.beginPath();
  ctx.arc(0, 0, 15.0, 0, Math.PI * 2);
  ctx.lineWidth = 0.07;
  ctx.strokeStyle = 'rgba(220,232,248,0.35)';
  ctx.stroke();

  for (let i = 0; i < 60; i++) {
    const major = i % 5 === 0;
    ctx.save();
    ctx.rotate(-(i * Math.PI) / 30); // 顺时针
    ctx.fillStyle = major ? 'rgba(232,241,252,0.95)' : 'rgba(198,214,236,0.6)';
    const w = major ? 0.34 : 0.12;
    const y0 = major ? 14.85 : 15.35;
    ctx.fillRect(-w / 2, y0, w, 16.15 - y0);
    ctx.restore();
  }

  // 时标（6 点位留给副表盘）
  for (let i = 0; i < 12; i++) {
    if (i === 6) continue;
    ctx.save();
    ctx.rotate(-(i * Math.PI) / 6);

    const metal = ctx.createLinearGradient(-1.4, 0, 1.4, 0);
    metal.addColorStop(0, '#ffffff');
    metal.addColorStop(0.45, '#94a7bd');
    metal.addColorStop(1, '#f4f8ff');

    if (i === 0) {
      // 12 点位倒三角
      ctx.beginPath();
      ctx.moveTo(0, 14.7);
      ctx.lineTo(1.85, 11.6);
      ctx.lineTo(-1.85, 11.6);
      ctx.closePath();
      ctx.fillStyle = metal;
      ctx.fill();
      ctx.beginPath();
      ctx.moveTo(0, 14.0);
      ctx.lineTo(1.32, 12.05);
      ctx.lineTo(-1.32, 12.05);
      ctx.closePath();
      ctx.fillStyle = '#e9f7ec';
      ctx.fill();
    } else {
      roundRect(ctx, 0, 13.1, 1.9, 3.15, 0.42);
      ctx.fillStyle = metal;
      ctx.fill();
      roundRect(ctx, 0, 13.1, 1.16, 2.42, 0.3);
      ctx.fillStyle = '#e9f7ec';
      ctx.fill();
    }
    ctx.restore();
  }

  // 日历 / 星期窗口的金属边框（窗口内容由独立平面显示）
  const frame = (win) => {
    roundRect(ctx, win.x, win.y, win.w + 0.55, win.h + 0.55, 0.6);
    ctx.lineWidth = 0.26;
    ctx.strokeStyle = 'rgba(232,240,250,0.9)';
    ctx.stroke();
    roundRect(ctx, win.x, win.y, win.w + 0.05, win.h + 0.05, 0.4);
    ctx.fillStyle = '#04080d';
    ctx.fill();
  };
  frame(DIAL_LAYOUT.dayWindow);
  frame(DIAL_LAYOUT.dateWindow);

  // 副表盘外圈阴影，制造下沉感
  const sd = DIAL_LAYOUT.subdial;
  const ring = ctx.createRadialGradient(sd.x, sd.y, sd.r * 0.9, sd.x, sd.y, sd.r + 0.8);
  ring.addColorStop(0, 'rgba(0,0,0,0.55)');
  ring.addColorStop(1, 'rgba(0,0,0,0)');
  ctx.beginPath();
  ctx.arc(sd.x, sd.y, sd.r + 0.8, 0, Math.PI * 2);
  ctx.fillStyle = ring;
  ctx.fill();

  // 品牌与文字
  drawText(ctx, 'KIRO', { x: 0, y: 5.9, size: 2.0, weight: 700, spacing: 0.28, color: '#f2f7ff' });
  drawText(ctx, 'DUAL TIME · GMT', { x: 0, y: 3.6, size: 0.92, weight: 500, spacing: 0.14, color: 'rgba(226,238,255,0.8)' });
  drawText(ctx, 'SMOOTH SWEEP · 3D', { x: 0, y: -14.2, size: 0.72, weight: 500, spacing: 0.1, color: 'rgba(210,226,246,0.65)' });

  return toTexture(canvas);
}

/* ---------- 外圈 24 小时刻度（第二时区读数） ---------- */

export const BEZEL = { uvRadius: 20.6, inner: 17.5, outer: 20.3 };

export function createBezelTexture({ size = 2048 } = {}) {
  const { canvas, ctx } = mmCanvas(size, BEZEL.uvRadius);
  const R = BEZEL.uvRadius;

  // 夜间半圈（18:00 -> 06:00，逻辑上半圆）
  ctx.beginPath();
  ctx.arc(0, 0, R, 0, Math.PI);
  ctx.closePath();
  ctx.fillStyle = '#0c1016';
  ctx.fill();
  // 白昼半圈（06:00 -> 18:00，逻辑下半圆）
  ctx.beginPath();
  ctx.arc(0, 0, R, Math.PI, Math.PI * 2);
  ctx.closePath();
  ctx.fillStyle = '#173câ'.slice(0, 0) || '#173c6b';
  ctx.fill();

  // 轻微的环形高光
  ctx.save();
  ctx.beginPath();
  ctx.arc(0, 0, R, 0, Math.PI * 2);
  ctx.clip();
  const sheen = ctx.createLinearGradient(-R, R, R, -R);
  sheen.addColorStop(0, 'rgba(255,255,255,0.10)');
  sheen.addColorStop(0.5, 'rgba(255,255,255,0.0)');
  sheen.addColorStop(1, 'rgba(255,255,255,0.07)');
  ctx.fillStyle = sheen;
  ctx.fillRect(-R, -R, R * 2, R * 2);
  ctx.restore();

  for (let h = 0; h < 24; h++) {
    const a = (h / 24) * Math.PI * 2; // 从 12 点位顺时针
    ctx.save();
    ctx.rotate(-a);
    const even = h % 2 === 0;
    ctx.fillStyle = 'rgba(240,246,255,0.9)';
    if (even) {
      const label = h === 0 ? '24' : String(h);
      drawText(ctx, label, {
        x: 0, y: 18.95, size: h % 6 === 0 ? 2.0 : 1.55,
        weight: h % 6 === 0 ? 700 : 500, color: '#f4f8ff',
      });
    } else {
      ctx.fillRect(-0.11, 18.5, 0.22, 0.9);
    }
    ctx.restore();
  }
  return toTexture(canvas);
}

/* ---------- 6 点位副表盘（第二时区 24 小时） ---------- */

export function createSubdialTexture({ size = 1024, label = '' } = {}) {
  const R = DIAL_LAYOUT.subdial.r;
  const { canvas, ctx } = mmCanvas(size, R);
  const tex = toTexture(canvas);

  function render(text) {
    ctx.save();
    ctx.setTransform(1, 0, 0, 1, 0, 0);
    ctx.clearRect(0, 0, size, size);
    ctx.restore();

    // 昼夜底色
    ctx.beginPath();
    ctx.arc(0, 0, R, 0, Math.PI);
    ctx.closePath();
    ctx.fillStyle = '#080f18';
    ctx.fill();
    ctx.beginPath();
    ctx.arc(0, 0, R, Math.PI, Math.PI * 2);
    ctx.closePath();
    ctx.fillStyle = '#1a4570';
    ctx.fill();

    // 同心细纹
    ctx.lineWidth = 0.045;
    for (let r = 0.6; r < R - 0.35; r += 0.28) {
      ctx.beginPath();
      ctx.arc(0, 0, r, 0, Math.PI * 2);
      ctx.strokeStyle = 'rgba(255,255,255,0.05)';
      ctx.stroke();
    }

    // 24 小时刻度
    for (let h = 0; h < 24; h++) {
      ctx.save();
      ctx.rotate(-(h / 24) * Math.PI * 2);
      const major = h % 6 === 0;
      ctx.fillStyle = major ? 'rgba(255,255,255,0.95)' : 'rgba(219,232,248,0.6)';
      const w = major ? 0.18 : 0.08;
      const y0 = major ? 3.95 : 4.25;
      ctx.fillRect(-w / 2, y0, w, 4.72 - y0);
      ctx.restore();
    }

    // 6/12/18/24 数字（保持正向易读）
    const marks = [['24', 0, 3.15], ['6', 3.15, 0], ['12', 0, -3.15], ['18', -3.15, 0]];
    for (const [txt, x, y] of marks) {
      drawText(ctx, txt, { x, y, size: 1.15, weight: 700, color: '#eef5ff' });
    }

    drawText(ctx, text || '', { x: 0, y: 1.65, size: 0.86, weight: 600, spacing: 0.06, color: 'rgba(238,246,255,0.92)' });
    drawText(ctx, '24H ZONE 2', { x: 0, y: -1.75, size: 0.6, weight: 500, spacing: 0.05, color: 'rgba(205,222,244,0.7)' });

    tex.needsUpdate = true;
  }

  render(label);
  return { texture: tex, setLabel: render };
}

/* ---------- 日历 / 星期窗口 ---------- */

export function createWindowTexture({ w, h, text = '', px = 512 }) {
  const canvas = document.createElement('canvas');
  canvas.width = px;
  canvas.height = Math.max(64, Math.round((px * h) / w));
  const ctx = canvas.getContext('2d');
  const tex = toTexture(canvas);

  function render(value) {
    const W = canvas.width;
    const H = canvas.height;
    ctx.clearRect(0, 0, W, H);
    const g = ctx.createLinearGradient(0, 0, 0, H);
    g.addColorStop(0, '#f7fafc');
    g.addColorStop(0.5, '#e3e9f1');
    g.addColorStop(1, '#c3ccd8');
    ctx.fillStyle = g;
    ctx.fillRect(0, 0, W, H);
    ctx.fillStyle = '#10151c';
    ctx.font = `700 ${Math.round(H * 0.66)}px "Helvetica Neue", Helvetica, Arial, sans-serif`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(String(value ?? ''), W / 2, H * 0.54);
    tex.needsUpdate = true;
  }

  render(text);
  return { texture: tex, setText: render };
}

/* ---------- 程序化环境贴图（金属反射用） ---------- */

export function createEnvTexture() {
  const canvas = document.createElement('canvas');
  canvas.width = 1024;
  canvas.height = 512;
  const ctx = canvas.getContext('2d');

  const g = ctx.createLinearGradient(0, 0, 0, 512);
  g.addColorStop(0, '#0d151f');
  g.addColorStop(0.42, '#66768c');
  g.addColorStop(0.5, '#9aa9ba');
  g.addColorStop(0.58, '#3c4756');
  g.addColorStop(1, '#04070a');
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, 1024, 512);

  // 柔光箱
  const boxes = [
    [140, 70, 300, 120, 1.0],
    [620, 40, 260, 100, 0.85],
    [880, 200, 160, 90, 0.55],
    [420, 300, 220, 80, 0.3],
  ];
  for (const [x, y, w, h, a] of boxes) {
    const rg = ctx.createRadialGradient(x + w / 2, y + h / 2, 1, x + w / 2, y + h / 2, Math.max(w, h) / 1.4);
    rg.addColorStop(0, `rgba(255,255,255,${a})`);
    rg.addColorStop(1, 'rgba(255,255,255,0)');
    ctx.fillStyle = rg;
    ctx.fillRect(x - w, y - h, w * 3, h * 3);
  }

  const tex = new THREE.CanvasTexture(canvas);
  tex.mapping = THREE.EquirectangularReflectionMapping;
  tex.colorSpace = THREE.SRGBColorSpace;
  return tex;
}
```

## src/watchModel.js

```js
import * as THREE from 'three';
import {
  createDialTexture,
  createBezelTexture,
  createSubdialTexture,
  createWindowTexture,
  DIAL_LAYOUT,
  BEZEL,
} from './textures.js';

/* ---------- 几何工具 ---------- */

/**
 * 绕 Z 轴旋成的回转体。轮廓点为 [半径, 轴向高度]，
 * 需按截面逆时针顺序给出，法线才朝外（three 的法线为切线顺时针旋转 90°）。
 */
function lathe(points, segments = 160) {
  const g = new THREE.LatheGeometry(
    points.map(([x, y]) => new THREE.Vector2(x, y)),
    segments
  );
  g.rotateX(Math.PI / 2); // 回转轴 Y -> Z
  return g;
}

/** 让 Ring/Circle 类几何按「半径映射」取样贴图，便于放射状排版 */
function applyRadialUV(geometry, radius) {
  const pos = geometry.attributes.position;
  const uv = geometry.attributes.uv;
  for (let i = 0; i < pos.count; i++) {
    uv.setXY(i, pos.getX(i) / (2 * radius) + 0.5, pos.getY(i) / (2 * radius) + 0.5);
  }
  uv.needsUpdate = true;
}

function extrude(shape, depth, bevel = 0.06) {
  const g = new THREE.ExtrudeGeometry(shape, {
    depth,
    curveSegments: 24,
    bevelEnabled: bevel > 0,
    bevelThickness: bevel,
    bevelSize: bevel,
    bevelOffset: 0,
    bevelSegments: 2,
  });
  g.translate(0, 0, -depth / 2);
  g.computeVertexNormals();
  return g;
}

/** 指针基础形：指向 +Y，尾部为配重侧 */
function batonShape({ tail, length, halfBase, halfTip, tipLength = 0 }) {
  const yTip = length - tipLength;
  const s = new THREE.Shape();
  s.moveTo(-halfBase, -tail);
  s.lineTo(halfBase, -tail);
  s.lineTo(halfTip, yTip);
  if (tipLength > 0) s.lineTo(0, length);
  s.lineTo(-halfTip, yTip);
  s.closePath();
  return s;
}

/** 箭头形（GMT 指针） */
function arrowShape({ tail, length, halfStem, halfHead, headLength }) {
  const yHead = length - headLength;
  const s = new THREE.Shape();
  s.moveTo(-halfStem, -tail);
  s.lineTo(halfStem, -tail);
  s.lineTo(halfStem, yHead);
  s.lineTo(halfHead, yHead);
  s.lineTo(0, length);
  s.lineTo(-halfHead, yHead);
  s.lineTo(-halfStem, yHead);
  s.closePath();
  return s;
}

function lugShape() {
  const s = new THREE.Shape();
  s.moveTo(-3.7, 15.5);
  s.lineTo(3.7, 15.5);
  s.quadraticCurveTo(4.1, 21.0, 2.0, 25.2);
  s.lineTo(-2.0, 25.2);
  s.quadraticCurveTo(-4.1, 21.0, -3.7, 15.5);
  s.closePath();
  return s;
}

/* ---------- 主构建函数 ---------- */

export function createWatch() {
  const DIAL_Z = -1.2;

  const group = new THREE.Group();
  group.name = 'watch';

  /* 材质 */
  const steelPolished = new THREE.MeshPhysicalMaterial({
    color: 0xdfe4ea, metalness: 1, roughness: 0.14, envMapIntensity: 1.5,
    clearcoat: 0.25, clearcoatRoughness: 0.15,
  });
  const steelBrushed = new THREE.MeshPhysicalMaterial({
    color: 0xc9d0d8, metalness: 1, roughness: 0.38, envMapIntensity: 1.15,
  });
  const handMetal = new THREE.MeshPhysicalMaterial({
    color: 0xf3f6fa, metalness: 1, roughness: 0.1, envMapIntensity: 1.7,
  });
  const gmtMetal = new THREE.MeshPhysicalMaterial({
    color: 0x4d8ee0, metalness: 0.85, roughness: 0.22, envMapIntensity: 1.4,
  });
  const secondMetal = new THREE.MeshPhysicalMaterial({
    color: 0xff5a3c, metalness: 0.55, roughness: 0.3, envMapIntensity: 1.1,
  });
  const lume = new THREE.MeshStandardMaterial({
    color: 0xe4f7e8, emissive: 0x6ff0b