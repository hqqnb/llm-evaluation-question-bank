I'll build this as a complete browser FPS game using Three.js. This is a large project, so let me set up the structure and build it out fully. Let me start by creating the project directory.

Before I dive in, one honest note on scope: I'll deliver a genuinely playable, self-contained browser FPS with all the core systems (pointer-lock FPS controls, multiple weapon classes with recoil/spread/reload/ADS, grenades with physics, bomb-defusal rounds, economy/buy menu, bot AI with difficulty tiers, HUD/radar, synthesized audio, and 3 maps including a structurally faithful Dust2). It runs from a static file server with no build step.

Let me create it.


<parameter name="command">mkdir -p /root/cs-web/js && cd /root/cs-web && echo "created"</parameter>
<parameter name="reason">Create the project directory structure</parameter>
</invoke>
Error: The provided path does not exist. Ensure the path points to a valid directory or file and try again.


<parameter name="command">pwd && whoami && ls -la</parameter>
<parameter name="reason">Check current directory and permissions</parameter>
</invoke>
null


<parameter name="command">echo "test" && pwd</parameter>
<parameter name="reason">Verify shell works</parameter>
</invoke>
null


<parameter name="command">mkdir -p /tmp/cs-web/js; cd /tmp/cs-web && pwd; ls -la</parameter>
<parameter name="reason">Try creating project in /tmp</parameter>
</invoke>
null


<parameter name="command">cd /tmp/cs-web 2>&1 && echo OK || echo FAIL</parameter>
<parameter name="reason">Check if directory was created</parameter>
</invoke>
nullThe terminal is returning null output. Let me try writing a file directly to test the filesystem.


<parameter name="path">/tmp/cs-web/test.txt</parameter>
<parameter name="fileText">test</parameter>
</invoke>
Successfully wrote to file /tmp/cs-web/test.txtFile writing works. Let me verify reads work too.


<parameter name="path">/tmp/cs-web/test.txt</parameter>
</invoke>
testFile I/O works (terminal stdout is returning null, but that won't block building — it just means I'll rely on file tools and careful coding). Let me remove the test file and build the full game.


<parameter name="command">rm /tmp/cs-web/test.txt</parameter>
<parameter name="reason">Clean up test file</parameter>
</invoke>
null

Now building the game. Starting with the entry point and structure.


<parameter name="path">/tmp/cs-web/index.html</parameter>
<parameter name="fileText"><!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Tactical Strike — 浏览器战术 FPS</title>
<link rel="stylesheet" href="css/style.css" />
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js"
  }
}
</script>
</head>
<body>
  <div id="app">
    <canvas id="game"></canvas>

    <!-- Crosshair -->
    <div id="crosshair">
      <div class="ch ch-t"></div><div class="ch ch-b"></div>
      <div class="ch ch-l"></div><div class="ch ch-r"></div>
      <div class="ch-dot"></div>
    </div>

    <!-- Hit marker -->
    <div id="hitmarker"></div>

    <!-- Damage flash -->
    <div id="damageflash"></div>
    <!-- Flashbang overlay -->
    <div id="flashoverlay"></div>

    <!-- HUD -->
    <div id="hud" class="hidden">
      <div id="hud-top">
        <div id="score-t" class="team-score t">T <b>0</b></div>
        <div id="round-timer">1:55</div>
        <div id="score-ct" class="team-score ct">CT <b>0</b></div>
      </div>
      <div id="round-banner"></div>

      <div id="hud-bottomleft">
        <div id="health-armor">
          <div class="stat hp"><span class="ico">✚</span><b id="hp-val">100</b></div>
          <div class="stat ar"><span class="ico">🛡</span><b id="ar-val">0</b></div>
        </div>
        <div id="money-display">$<b id="money-val">800</b></div>
      </div>

      <div id="hud-bottomright">
        <div id="ammo-display"><b id="ammo-clip">30</b> / <span id="ammo-reserve">90</span></div>
        <div id="weapon-name">AK-47</div>
        <div id="nade-slots"></div>
      </div>

      <div id="bomb-status"></div>
      <div id="killfeed"></div>

      <!-- Radar -->
      <div id="radar">
        <canvas id="radar-canvas" width="220" height="220"></canvas>
      </div>

      <div id="inventory-bar"></div>
      <div id="center-msg"></div>
      <div id="planting-bar" class="progress hidden"><div class="fill"></div><span>下包中...</span></div>
      <div id="defusing-bar" class="progress hidden"><div class="fill"></div><span>拆弹中...</span></div>
    </div>

    <!-- Buy Menu -->
    <div id="buymenu" class="overlay hidden">
      <div class="buy-panel">
        <div class="buy-head">
          <h2>购买菜单</h2>
          <div class="buy-money">$<b id="buy-money">800</b></div>
          <div class="buy-hint">按 <b>B</b> 关闭 · 数字键快速购买</div>
        </div>
        <div class="buy-cats">
          <div class="buy-col"><h3>手枪</h3><div class="buy-list" data-cat="pistol"></div></div>
          <div class="buy-col"><h3>冲锋枪</h3><div class="buy-list" data-cat="smg"></div></div>
          <div class="buy-col"><h3>步枪</h3><div class="buy-list" data-cat="rifle"></div></div>
          <div class="buy-col"><h3>狙击/霰弹</h3><div class="buy-list" data-cat="heavy"></div></div>
          <div class="buy-col"><h3>装备</h3><div class="buy-list" data-cat="gear"></div></div>
          <div class="buy-col"><h3>投掷物</h3><div class="buy-list" data-cat="nade"></div></div>
        </div>
      </div>
    </div>

    <!-- Scoreboard -->
    <div id="scoreboard" class="overlay hidden">
      <div class="sb-panel">
        <div class="sb-title">记分板 — <span id="sb-map"></span></div>
        <div class="sb-teams">
          <div class="sb-team">
            <div class="sb-head t">恐怖分子 (T) — <span id="sb-tscore">0</span></div>
            <table id="sb-t-table"></table>
          </div>
          <div class="sb-team">
            <div class="sb-head ct">反恐精英 (CT) — <span id="sb-ctscore">0</span></div>
            <table id="sb-ct-table"></table>
          </div>
        </div>
      </div>
    </div>

    <!-- Main Menu -->
    <div id="menu" class="overlay">
      <div class="menu-panel">
        <h1>TACTICAL STRIKE</h1>
        <p class="subtitle">浏览器战术第一人称射击 · 爆破模式</p>

        <div class="menu-row">
          <label>选择地图</label>
          <div id="map-select" class="chip-group"></div>
        </div>
        <div class="menu-row">
          <label>你的阵营</label>
          <div id="team-select" class="chip-group">
            <div class="chip active" data-team="T">恐怖分子 (T)</div>
            <div class="chip" data-team="CT">反恐精英 (CT)</div>
          </div>
        </div>
        <div class="menu-row">
          <label>每队机器人数量</label>
          <div id="bot-select" class="chip-group">
            <div class="chip" data-bots="2">2</div>
            <div class="chip active" data-bots="4">4</div>
            <div class="chip" data-bots="5">5</div>
          </div>
        </div>
        <div class="menu-row">
          <label>机器人难度</label>
          <div id="diff-select" class="chip-group">
            <div class="chip" data-diff="easy">简单</div>
            <div class="chip active" data-diff="normal">普通</div>
            <div class="chip" data-diff="hard">困难</div>
            <div class="chip" data-diff="expert">专家</div>
          </div>
        </div>
        <div class="menu-row">
          <label>获胜回合数</label>
          <div id="win-select" class="chip-group">
            <div class="chip" data-win="4">4</div>
            <div class="chip active" data-win="8">8</div>
            <div class="chip" data-win="16">16</div>
          </div>
        </div>

        <button id="start-btn">开始对战</button>
        <div class="controls-help">
          <b>操作:</b> WASD 移动 · 鼠标 视角 · 左键 开火 · 右键 开镜 · R 换弹 ·
          1-5 切换武器 · G 丢雷 · B 购买 · Shift 静步 · Ctrl 下蹲 · Space 跳跃 ·
          E 下包/拆包 · Tab 记分板 · Esc 暂停
        </div>
      </div>
    </div>

    <!-- Pause -->
    <div id="pause" class="overlay hidden">
      <div class="menu-panel small">
        <h1>已暂停</h1>
        <button id="resume-btn">继续</button>
        <button id="restart-btn">重新开始</button>
        <button id="tomenu-btn">返回主菜单</button>
      </div>
    </div>

    <!-- Round/Match result -->
    <div id="result" class="overlay hidden">
      <div class="menu-panel">
        <h1 id="result-title">回合结束</h1>
        <p id="result-desc"></p>
        <button id="result-btn">继续</button>
      </div>
    </div>

    <!-- Death / spectate -->
    <div id="deadscreen" class="hidden">
      <div class="dead-banner">你已阵亡 — 观战中 · 等待下一回合</div>
    </div>

    <div id="loading" class="overlay">
      <div class="menu-panel small"><h1>加载中...</h1><p>正在生成地图与武器资源</p></div>
    </div>
  </div>

  <script type="module" src="js/main.js"></script>
</body>
</html>
</parameter>
</invoke>
Filesystem returned no output but the file was created. Let me verify silently and continue.


<parameter name="command">ls -la /tmp/cs-web/ /tmp/cs-web/js/ 2>&1; cat /tmp/cs-web/index.html | head -c 60</parameter>
<parameter name="reason">Verify files exist despite null terminal output</parameter>
</invoke>
null


<parameter name="path">/tmp/cs-web</parameter>
</invoke>

There are no files in the directory.


<parameter name="path">/tmp/cs-web/index.html</parameter>
</invoke>
<parameter name="path">/tmp/cs-web/index.html</parameter>
nullThe environment is behaving inconsistently — the earlier write/read round-tripped, but now reads and listings return null or empty. Let me diagnose whether the filesystem is actually persisting.


<parameter name="path">/tmp/probe.txt</parameter>
<parameter name="fileText">hello-probe-123</parameter>
</invoke>
Successfully wrote to file /tmp/probe.txt


<parameter name="path">/tmp/probe.txt</parameter>
</invoke>
hello-probe-123


<parameter name="path">/tmp</parameter>
</invoke>

There are no files in the directory.


<parameter name="path">/tmp/cs-web/index.html</parameter>
</invoke>
null
<parameter name="path">/tmp/probe.txt</parameter>
</invoke>
nullI've now confirmed a hard environment constraint through direct testing: the write tool reports success, but files do not reliably persist between tool calls (a file I read successfully returns null two calls later, and directory listings come back empty), and the terminal returns null for every command including `echo`. That means I cannot run a dev server, execute a build, or iteratively test inside this environment — the sandbox appears to reset per call.

I'm telling you this plainly instead of pretending otherwise. Given that, the robust way to deliver something you can actually run is a single self-contained HTML file that needs no build step and loads Three.js from a CDN. I'll write the complete game into one file. Let me build it as a single deliverable.


<parameter name="path">/tmp/cs-web/game.html</parameter>
<parameter name="fileText"><!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Tactical Strike — 浏览器战术 FPS (爆破模式)</title>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>
<style>
*{margin:0;padding:0;box-sizing:border-box;font-family:"Segoe UI",Arial,"Microsoft YaHei",sans-serif;user-select:none}
html,body{width:100%;height:100%;overflow:hidden;background:#0a0d12;color:#e8eef5}
#game{position:fixed;inset:0;width:100%;height:100%;display:block}
.hidden{display:none!important}
.overlay{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;
  background:radial-gradient(circle at 50% 30%,rgba(30,40,55,.92),rgba(6,9,13,.97));z-index:50}
.menu-panel{background:rgba(15,22,32,.96);border:1px solid #2a3a4d;border-radius:12px;padding:28px 34px;
  width:640px;max-width:94vw;box-shadow:0 20px 60px rgba(0,0,0,.6)}
.menu-panel.small{width:360px;text-align:center}
.menu-panel h1{font-size:34px;letter-spacing:4px;color:#f0a24b;text-shadow:0 2px 10px rgba(240,162,75,.35);margin-bottom:4px}
.subtitle{color:#8fa4bb;margin-bottom:20px;font-size:13px}
.menu-row{margin:14px 0}
.menu-row>label{display:block;font-size:12px;color:#8fa4bb;margin-bottom:6px;text-transform:uppercase;letter-spacing:1px}
.chip-group{display:flex;flex-wrap:wrap;gap:8px}
.chip{padding:8px 14px;background:#1b2735;border:1px solid #2c3f54;border-radius:6px;cursor:pointer;font-size:13px;transition:.15s}
.chip:hover{background:#233346}
.chip.active{background:#f0a24b;color:#12181f;border-color:#f0a24b;font-weight:700}
button{margin-top:18px;width:100%;padding:13px;background:linear-gradient(#f0a24b,#d8862c);border:none;border-radius:8px;
  color:#12181f;font-weight:700;font-size:16px;letter-spacing:2px;cursor:pointer}
button:hover{filter:brightness(1.08)}
.menu-panel.small button{margin-top:10px}
.controls-help{margin-top:18px;font-size:11.5px;color:#7f93a8;line-height:1.7;border-top:1px solid #223145;padding-top:12px}
.controls-help b{color:#c9d7e6}

/* Crosshair */
#crosshair{position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);z-index:10;pointer-events:none}
.ch{position:absolute;background:#39ff88;box-shadow:0 0 2px rgba(0,0,0,.8)}
.ch-t,.ch-b{width:2px;height:7px;left:-1px}
.ch-l,.ch-r{height:2px;width:7px;top:-1px}
.ch-t{top:-11px}.ch-b{top:4px}.ch-l{left:-11px}.ch-r{left:4px}
.ch-dot{position:absolute;width:2px;height:2px;background:#39ff88;left:-1px;top:-1px}
#hitmarker{position:fixed;left:50%;top:50%;transform:translate(-50%,-50%) rotate(45deg);z-index:11;
  width:16px;height:16px;pointer-events:none;opacity:0}
#hitmarker::before,#hitmarker::after{content:"";position:absolute;background:#fff}
#hitmarker::before{width:2px;height:16px;left:7px}#hitmarker::after{height:2px;width:16px;top:7px}
#hitmarker.show{animation:hm .25s ease-out}
@keyframes hm{0%{opacity:1;transform:translate(-50%,-50%) rotate(45deg) scale(.6)}100%{opacity:0;transform:translate(-50%,-50%) rotate(45deg) scale(1.3)}}

#damageflash{position:fixed;inset:0;z-index:9;pointer-events:none;box-shadow:inset 0 0 140px rgba(200,20,20,0);transition:box-shadow .1s}
#flashoverlay{position:fixed;inset:0;z-index:40;pointer-events:none;background:#fff;opacity:0}

/* HUD */
#hud{position:fixed;inset:0;z-index:8;pointer-events:none;font-variant-numeric:tabular-nums}
#hud-top{position:absolute;top:14px;left:50%;transform:translateX(-50%);display:flex;align-items:center;gap:26px;
  background:rgba(10,15,22,.72);padding:8px 22px;border-radius:8px;border:1px solid #223145}
.team-score{font-size:15px;color:#cdd;letter-spacing:1px}.team-score b{font-size:24px;margin-left:6px}
.team-score.t b{color:#f0a24b}.team-score.ct b{color:#5aa9ff}
#round-timer{font-size:26px;font-weight:700;min-width:74px;text-align:center;color:#fff}
#round-banner{position:absolute;top:74px;left:50%;transform:translateX(-50%);font-size:15px;color:#ffd479;
  text-shadow:0 1px 6px #000;opacity:0;transition:opacity .3s}
#hud-bottomleft{position:absolute;left:22px;bottom:20px}
#health-armor{display:flex;gap:20px;align-items:center}
.stat{display:flex;align-items:center;gap:8px;font-size:30px;font-weight:700;text-shadow:0 2px 6px #000}
.stat .ico{font-size:22px}.stat.hp{color:#e9f3ff}.stat.ar{color:#8fbfff}
#money-display{margin-top:8px;font-size:22px;color:#5fd08a;font-weight:700;text-shadow:0 2px 6px #000}
#hud-bottomright{position:absolute;right:22px;bottom:20px;text-align:right}
#ammo-display{font-size:34px;font-weight:700;text-shadow:0 2px 6px #000}
#ammo-display #ammo-reserve{font-size:20px;color:#9fb2c6}
#weapon-name{font-size:15px;color:#c6d4e4;letter-spacing:1px;margin-top:2px}
#nade-slots{margin-top:8px;display:flex;gap:6px;justify-content:flex-end}
.nade-ico{width:26px;height:26px;border-radius:50%;border:2px solid #566;display:flex;align-items:center;
  justify-content:center;font-size:12px;font-weight:700;background:rgba(20,28,38,.7)}
.nade-ico.he{background:#3a5a2a;border-color:#7ab04a}
.nade-ico.flash{background:#8a7a2a;border-color:#e0d060}
.nade-ico.smoke{background:#455;border-color:#8aa}
.nade-ico.molly{background:#7a3a1a;border-color:#e07030}
.nade-ico.active{outline:2px solid #fff}
#inventory-bar{position:absolute;right:22px;bottom:120px;display:flex;flex-direction:column;gap:4px;align-items:flex-end}
.inv-slot{font-size:12px;color:#8fa4bb;background:rgba(10,15,22,.6);padding:3px 8px;border-radius:4px;border:1px solid #223145}
.inv-slot.active{color:#ffd479;border-color:#f0a24b}
#bomb-status{position:absolute;top:110px;left:50%;transform:translateX(-50%);font-size:16px;font-weight:700;color:#ff6b4a}
#killfeed{position:absolute;top:14px;right:16px;text-align:right;font-size:13px;line-height:1.9}
.kf-row{background:rgba(10,15,22,.6);padding:2px 8px;border-radius:4px;margin-bottom:3px;display:inline-block}
.kf-row .t{color:#f0a24b}.kf-row .ct{color:#5aa9ff}.kf-row .hs{color:#ff5a5a}
#radar{position:absolute;top:16px;left:16px;width:220px;height:220px;background:rgba(8,12,18,.7);
  border:1px solid #223145;border-radius:8px;overflow:hidden}
#center-msg{position:absolute;top:44%;left:50%;transform:translateX(-50%);font-size:20px;color:#ffd479;
  text-shadow:0 2px 8px #000;text-align:center;opacity:0;transition:opacity .25s}
.progress{position:absolute;left:50%;top:60%;transform:translateX(-50%);width:280px;height:26px;
  background:rgba(0,0,0,.6);border:1px solid #445;border-radius:5px;overflow:hidden}
.progress .fill{position:absolute;left:0;top:0;bottom:0;width:0%;background:#f0a24b}
.progress span{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;font-size:13px;font-weight:700;color:#fff;text-shadow:0 1px 3px #000}

/* Buy menu */
.buy-panel{background:rgba(13,19,28,.97);border:1px solid #2a3a4d;border-radius:12px;padding:22px;width:1000px;max-width:96vw}
.buy-head{display:flex;align-items:baseline;gap:18px;margin-bottom:16px}
.buy-head h2{color:#f0a24b;letter-spacing:2px}
.buy-money{font-size:22px;color:#5fd08a;font-weight:700}
.buy-hint{margin-left:auto;font-size:12px;color:#8fa4bb}
.buy-cats{display:grid;grid-template-columns:repeat(6,1fr);gap:12px}
.buy-col h3{font-size:12px;color:#8fa4bb;text-transform:uppercase;letter-spacing:1px;margin-bottom:8px;border-bottom:1px solid #223145;padding-bottom:4px}
.buy-item{background:#1a2634;border:1px solid #2b3d51;border-radius:6px;padding:7px 9px;margin-bottom:6px;cursor:pointer;transition:.12s}
.buy-item:hover{background:#22344a;border-color:#f0a24b}
.buy-item .n{font-size:13px;color:#e8eef5}
.buy-item .p{font-size:12px;color:#5fd08a}
.buy-item.cant{opacity:.4;cursor:not-allowed}
.buy-item .k{float:right;font-size:11px;color:#f0a24b;font-weight:700}

/* Scoreboard */
.sb-panel{background:rgba(13,19,28,.97);border:1px solid #2a3a4d;border-radius:12px;padding:22px;width:820px;max-width:96vw}
.sb-title{text-align:center;color:#f0a24b;letter-spacing:2px;font-size:18px;margin-bottom:16px}
.sb-teams{display:grid;grid-template-columns:1fr 1fr;gap:20px}
.sb-head{font-weight:700;padding:6px 10px;border-radius:5px;margin-bottom:6px}
.sb-head.t{background:rgba(240,162,75,.15);color:#f0a24b}
.sb-head.ct{background:rgba(90,169,255,.15);color:#5aa9ff}
.sb-team table{width:100%;border-collapse:collapse;font-size:13px}
.sb-team td{padding:4px 10px;border-bottom:1px solid #1c2836}
.sb-team td:last-child,.sb-team td:nth-child(2),.sb-team td:nth-child(3){text-align:right;color:#9fb2c6}
.dead .name{color:#6b7d90;text-decoration:line-through}
.you .name{color:#ffd479}

#deadscreen{position:fixed;left:0;right:0;bottom:80px;z-index:9;text-align:center;pointer-events:none}
.dead-banner{display:inline-block;background:rgba(120,20,20,.7);padding:10px 24px;border-radius:6px;font-size:16px;letter-spacing:1px}
</style>
</head>
<body>
<canvas id="game"></canvas>

<div id="crosshair">
  <div class="ch ch-t"></div><div class="ch ch-b"></div><div class="ch ch-l"></div><div class="ch ch-r"></div>
  <div class="ch-dot"></div>
</div>
<div id="hitmarker"></div>
<div id="damageflash"></div>
<div id="flashoverlay"></div>

<div id="hud" class="hidden">
  <div id="hud-top">
    <div class="team-score t">T <b id="score-t">0</b></div>
    <div id="round-timer">1:55</div>
    <div class="team-score ct">CT <b id="score-ct">0</b></div>
  </div>
  <div id="round-banner"></div>
  <div id="hud-bottomleft">
    <div id="health-armor">
      <div class="stat hp"><span class="ico">✚</span><b id="hp-val">100</b></div>
      <div class="stat ar"><span class="ico">🛡</span><b id="ar-val">0</b></div>
    </div>
    <div id="money-display">$<b id="money-val">800</b></div>
  </div>
  <div id="hud-bottomright">
    <div id="ammo-display"><b id="ammo-clip">30</b> / <span id="ammo-reserve">90</span></div>
    <div id="weapon-name">AK-47</div>
    <div id="nade-slots"></div>
  </div>
  <div id="bomb-status"></div>
  <div id="killfeed"></div>
  <div id="radar"><canvas id="radar-canvas" width="220" height="220"></canvas></div>
  <div id="inventory-bar"></div>
  <div id="center-msg"></div>
  <div id="planting-bar" class="progress hidden"><div class="fill"></div><span>下包中...</span></div>
  <div id="defusing-bar" class="progress hidden"><div class="fill"></div><span>拆弹中...</span></div>
</div>

<div id="buymenu" class="overlay hidden">
  <div class="buy-panel">
    <div class="buy-head"><h2>购买菜单</h2><div class="buy-money">$<b id="buy-money">800</b></div>
      <div class="buy-hint">按 <b>B</b> 关闭 · 只能在出生区购买期购买</div></div>
    <div class="buy-cats">
      <div class="buy-col"><h3>手枪</h3><div class="buy-list" data-cat="pistol"></div></div>
      <div class="buy-col"><h3>冲锋枪</h3><div class="buy-list" data-cat="smg"></div></div>
      <div class="buy-col"><h3>步枪</h3><div class="buy-list" data-cat="rifle"></div></div>
      <div class="buy-col"><h3>狙击/霰弹</h3><div class="buy-list" data-cat="heavy"></div></div>
      <div class="buy-col"><h3>装备</h3><div class="buy-list" data-cat="gear"></div></div>
      <div class="buy-col"><h3>投掷物</h3><div class="buy-list" data-cat="nade"></div></div>
    </div>
  </div>
</div>

<div id="scoreboard" class="overlay hidden">
  <div class="sb-panel">
    <div class="sb-title">记分板 — <span id="sb-map"></span></div>
    <div class="sb-teams">
      <div class="sb-team"><div class="sb-head t">恐怖分子 (T) — <span id="sb-tscore">0</span></div><table id="sb-t-table"></table></div>
      <div class="sb-team"><div class="sb-head ct">反恐精英 (CT) — <span id="sb-ctscore">0</span></div><table id="sb-ct-table"></table></div>
    </div>
  </div>
</div>

<div id="menu" class="overlay">
  <div class="menu-panel">
    <h1>TACTICAL STRIKE</h1>
    <p class="subtitle">浏览器战术第一人称射击 · 爆破模式 · 对战 Bot</p>
    <div class="menu-row"><label>选择地图</label><div id="map-select" class="chip-group"></div></div>
    <div class="menu-row"><label>你的阵营</label><div id="team-select" class="chip-group">
      <div class="chip active" data-team="T">恐怖分子 (T)</div><div class="chip" data-team="CT">反恐精英 (CT)</div></div></div>
    <div class="menu-row"><label>每队机器人数量</label><div id="bot-select" class="chip-group">
      <div class="chip" data-bots="2">2</div><div class="chip active" data-bots="4">4</div><div class="chip" data-bots="5">5</div></div></div>
    <div class="menu-row"><label>机器人难度</label><div id="diff-select" class="chip-group">
      <div class="chip" data-diff="easy">简单</div><div class="chip active" data-diff="normal">普通</div>
      <div class="chip" data-diff="hard">困难</div><div class="chip" data-diff="expert">专家</div></div></div>
    <div class="menu-row"><label>获胜回合数</label><div id="win-select" class="chip-group">
      <div class="chip" data-win="4">4</div><div class="chip active" data-win="8">8</div><div class="chip" data-win="16">16</div></div></div>
    <button id="start-btn">开始对战</button>
    <div class="controls-help"><b>操作:</b> WASD 移动 · 鼠标 视角 · 左键 开火 · 右键 开镜 · R 换弹 ·
      1-5 切枪 · G 投掷物(4/5/6/7 选择) · B 购买 · Shift 静步 · Ctrl 下蹲 · Space 跳跃 ·
      E 下包/拆包 · Tab 记分板 · Esc 暂停</div>
  </div>
</div>

<div id="pause" class="overlay hidden">
  <div class="menu-panel small"><h1>已暂停</h1>
    <button id="resume-btn">继续</button><button id="restart-btn">重新开始</button><button id="tomenu-btn">返回主菜单</button></div>
</div>

<div id="result" class="overlay hidden">
  <div class="menu-panel"><h1 id="result-title">回合结束</h1><p id="result-desc"></p><button id="result-btn">继续</button></div>
</div>

<div id="deadscreen" class="hidden"><div class="dead-banner">你已阵亡 — 观战中 · 等待下一回合</div></div>

<script>
"use strict";
/* ============================================================================
   TACTICAL STRIKE — single-file browser tactical FPS (bomb-defusal)
   Three.js (global THREE from CDN). No build step.
   ========================================================================== */

const THREEJS = window.THREE;
const V3 = (x=0,y=0,z=0)=>new THREEJS.Vector3(x,y,z);

/* ---------------------------- Global config ------------------------------- */
const CFG = {
  gravity: 22,
  playerHeight: 1.75, crouchHeight: 1.15, radius: 0.42,
  walkSpeed: 6.0, runSpeed: 8.4, crouchSpeed: 3.4, sneakSpeed: 3.8,
  jumpVel: 7.4,
  freezeTime: 8, roundTime: 115, bombTime: 40, defuseTime: 5, defuseTimeKit: 3, plantTime: 3.2,
  startMoney: 800, maxMoney: 16000,
  reward: { kill:300, win:3250, plant:800, defuse:300, teamPlantWinBonus:0 },
  lossBonusTiers:[1400,1900,2400,2900,3400],
};

/* ---------------------------- Weapon definitions -------------------------- */
// dmg = base body damage; hs multiplier; armorPen fraction; recoil in radians
const WEAPONS = {
  knife:   {name:"刀",    cat:"melee", dmg:55, rate:0.45, mag:0, reserve:0, price:0, melee:true},
  glock:   {name:"Glock-18", cat:"pistol", dmg:26, hs:4, rate:0.15, mag:20, reserve:120, reload:1.5, price:200, spread:0.012, recoil:0.010, range:120, armorPen:0.47, auto:false, zoom:0},
  usp:     {name:"USP-S",  cat:"pistol", dmg:32, hs:4, rate:0.17, mag:12, reserve:100, reload:1.6, price:200, spread:0.010, recoil:0.011, range:120, armorPen:0.5, auto:false, zoom:0},
  p250:    {name:"P250",   cat:"pistol", dmg:38, hs:4, rate:0.16, mag:13, reserve:26, reload:1.5, price:300, spread:0.012, recoil:0.013, range:110, armorPen:0.64, auto:false, zoom:0},
  deagle:  {name:"沙漠之鹰", cat:"pistol", dmg:63, hs:4, rate:0.35, mag:7, reserve:35, reload:2.1, price:700, spread:0.011, recoil:0.045, range:130, armorPen:0.62, auto:false, zoom:0},
  mp9:     {name:"MP9",    cat:"smg", dmg:26, hs:3, rate:0.075, mag:30, reserve:120, reload:2.0, price:1250, spread:0.02, recoil:0.014, range:90, armorPen:0.6, auto:true, zoom:0},
  mp5:     {name:"MP5-SD", cat:"smg", dmg:27, hs:3, rate:0.08, mag:30, reserve:120, reload:2.3, price:1500, spread:0.018, recoil:0.013, range:95, armorPen:0.6, auto:true, zoom:0},
  ump45:   {name:"UMP-45", cat:"smg", dmg:35, hs:3, rate:0.105, mag:25, reserve:100, reload:2.4, price:1200, spread:0.022, recoil:0.017, range:85, armorPen:0.65, auto:true, zoom:0},
  p90:     {name:"P90",    cat:"smg", dmg:26, hs:3, rate:0.07, mag:50, reserve:100, reload:3.3, price:2350, spread:0.024, recoil:0.012, range:90, armorPen:0.69, auto:true, zoom:0},
  ak47:    {name:"AK-47",  cat:"rifle", dmg:36, hs:4, rate:0.1, mag:30, reserve:90, reload:2.5, price:2700, spread:0.016, recoil:0.028, range:150, armorPen:0.775, auto:true, zoom:0},
  m4a4:    {name:"M4A4",   cat:"rifle", dmg:33, hs:4, rate:0.09, mag:30, reserve:90, reload:3.1, price:3100, spread:0.014, recoil:0.024, range:150, armorPen:0.7, auto:true, zoom:0},
  m4a1s:   {name:"M4A1-S", cat:"rifle", dmg:38, hs:4, rate:0.1, mag:25, reserve:75, reload:3.1, price:2900, spread:0.012, recoil:0.021, range:150, armorPen:0.7, auto:true, zoom:0},
  famas:   {name:"FAMAS",  cat:"rifle", dmg:30, hs:4, rate:0.09, mag:25, reserve:90, reload:3.3, price:2050, spread:0.02, recoil:0.026, range:130, armorPen:0.7, auto:true, zoom:0},
  galil:   {name:"Galil AR", cat:"rifle", dmg:30, hs:4, rate:0.095, mag:35, reserve:90, reload:3.0, price:1800, spread:0.02, recoil:0.03, range:130, armorPen:0.775, auto:true, zoom:0},
  awp:     {name:"AWP",    cat:"heavy", dmg:115, hs:1.5, rate:1.45, mag:10, reserve:30, reload:3.7, price:4750, spread:0.002, recoil:0.09, range:250, armorPen:0.975, auto:false, zoom:0.28},
  ssg08:   {name:"SSG 08", cat:"heavy", dmg:78, hs:1.9, rate:1.25, mag:10, reserve:90, reload:3.7, price:1700, spread:0.003, recoil:0.05, range:220, armorPen:0.85, auto:false, zoom:0.35},
  nova:    {name:"Nova(霰弹)", cat:"heavy", dmg:26, hs:1.5, rate:0.85, mag:8, reserve:32, reload:0.5, price:1050, spread:0.06, recoil:0.05, range:35, armorPen:0.5, auto:false, zoom:0, pellets:9},
  xm1014:  {name:"XM1014",  cat:"heavy", dmg:20, hs:1.5, rate:0.3, mag:7, reserve:32, reload:0.5, price:2000, spread:0.07, recoil:0.04, range:32, armorPen:0.5, auto:true, zoom:0, pellets:6},
};
const GEAR = {
  kevlar: {name:"防弹衣", price:650, key:"5"},
  helmet: {name:"防弹衣+头盔", price:1000, key:"6"},
  defuser:{name:"拆弹器", price:400, key:"7", ctOnly:true},
};
const NADES = {
  he:    {name:"高爆手雷", price:300, color:0x3a5a2a, cls:"he",   slot:4},
  flash: {name:"闪光弹",   price:200, color:0xcfc040, cls:"flash",slot:5},
  smoke: {name:"烟雾弹",   price:300, color:0x8899aa, cls:"smoke",slot:6},
  molly: {name:"燃烧瓶",   price:400, color:0xd06020, cls:"molly",slot:7},
};

/* ---------------------------- Audio engine -------------------------------- */
const Audio = (()=>{
  let ctx=null, master=null;
  function init(){ if(ctx) return; ctx=new (window.AudioContext||window.webkitAudioContext)();
    master=ctx.createGain(); master.gain.value=0.6; master.connect(ctx.destination); }
  function resume(){ if(ctx&&ctx.state==="suspended") ctx.resume(); }
  function noise(dur){ const n=ctx.sampleRate*dur|0, b=ctx.createBuffer(1,n,ctx.sampleRate),d=b.getChannelData(0);
    for(let i=0;i<n;i++) d[i]=Math.random()*2-1; const s=ctx.createBufferSource(); s.buffer=b; return s; }
  // 3D-ish attenuation by distance
  function gainFor(dist){ if(dist==null) return 1; return Math.max(0.03, Math.min(1, 22/(dist+4))); }
  function shot(type, dist){ if(!ctx) return; const g=ctx.createGain(); g.connect(master);
    const vol=gainFor(dist); let dur=0.12, cut=1600, peak=0.5;
    if(type==="rifle"){dur=0.14;cut=2200;peak=0.55}
    else if(type==="pistol"){dur=0.09;cut=1800;peak=0.4}
    else if(type==="smg"){dur=0.08;cut=2000;peak=0.35}
    else if(type==="awp"){dur=0.3;cut=1200;peak=0.7}
    else if(type==="heavy"){dur=0.2;cut=900;peak=0.6}
    const s=noise(dur); const f=ctx.createBiquadFilter(); f.type="lowpass"; f.frequency.value=cut;
    s.connect(f); f.connect(g);
    g.gain.setValueAtTime(peak*vol,ctx.currentTime); g.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+dur);
    // low thump
    const o=ctx.createOscillator(); o.type="sine"; o.frequency.setValueAtTime(140,ctx.currentTime);
    o.frequency.exponentialRampToValueAtTime(50,ctx.currentTime+dur); const og=ctx.createGain();
    og.gain.setValueAtTime(peak*0.6*vol,ctx.currentTime); og.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+dur);
    o.connect(og); og.connect(master); o.start(); o.stop(ctx.currentTime+dur);
    s.start(); s.stop(ctx.currentTime+dur); }
  function reload(){ if(!ctx) return; [0,0.15,0.4,0.9].forEach((t,i)=>{ const g=ctx.createGain(); g.connect(master);
    const s=noise(0.04); const f=ctx.createBiquadFilter(); f.type="bandpass"; f.frequency.value=1200+i*400; s.connect(f); f.connect(g);
    g.gain.setValueAtTime(0.25,ctx.currentTime+t); g.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+t+0.05);
    s.start(ctx.currentTime+t); s.stop(ctx.currentTime+t+0.06); }); }
  function footstep(dist){ if(!ctx) return; const g=ctx.createGain(); g.connect(master); const v=gainFor(dist)*0.25;
    const s=noise(0.06); const f=ctx.createBiquadFilter(); f.type="lowpass"; f.frequency.value=500; s.connect(f); f.connect(g);
    g.gain.setValueAtTime(v,ctx.currentTime); g.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+0.08);
    s.start(); s.stop(ctx.currentTime+0.09); }
  function explosion(dist){ if(!ctx) return; const g=ctx.createGain(); g.connect(master); const v=gainFor(dist);
    const s=noise(0.6); const f=ctx.createBiquadFilter(); f.type="lowpass"; f.frequency.setValueAtTime(900,ctx.currentTime);
    f.frequency.exponentialRampToValueAtTime(80,ctx.currentTime+0.5); s.connect(f); f.connect(g);
    g.gain.setValueAtTime(0.8*v,ctx.currentTime); g.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+0.6);
    s.start(); s.stop(ctx.currentTime+0.62); }
  function flashbang(){ if(!ctx) return; const o=ctx.createOscillator(); o.type="sine"; const g=ctx.createGain();
    o.frequency.setValueAtTime(3000,ctx.currentTime); o.frequency.exponentialRampToValueAtTime(6000,ctx.currentTime+0.3);
    g.gain.setValueAtTime(0.001,ctx.currentTime); g.gain.linearRampToValueAtTime(0.3,ctx.currentTime+0.02);
    g.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+2.5); o.connect(g); g.connect(master); o.start(); o.stop(ctx.currentTime+2.6); }
  function beep(freq){ if(!ctx) return; const o=ctx.createOscillator(); o.type="square"; o.frequency.value=freq||880;
    const g=ctx.createGain(); g.gain.setValueAtTime(0.2,ctx.currentTime); g.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+0.1);
    o.connect(g); g.connect(master); o.start(); o.stop(ctx.currentTime+0.11); }
  function hit(){ if(!ctx) return; const o=ctx.createOscillator(); o.type="triangle"; o.frequency.value=440;
    const g=ctx.createGain(); g.gain.setValueAtTime(0.15,ctx.currentTime); g.gain.exponentialRampToValueAtTime(0.001,ctx.currentTime+0.06);
    o.connect(g); g.connect(master); o.start(); o.stop(ctx.currentTime+0.07); }
  return {init,resume,shot,reload,footstep,explosion,flashbang,beep,hit};
})();

/* ---------------------------- Materials cache ----------------------------- */
const MAT = {};
function initMaterials(){
  MAT.sand  = new THREEJS.MeshLambertMaterial({color:0xcdb27f});
  MAT.sandDark = new THREEJS.MeshLambertMaterial({color:0xa8905f});
  MAT.wall  = new THREEJS.MeshLambertMaterial({color:0xc2a970});
  MAT.wall2 = new THREEJS.MeshLambertMaterial({color:0xb59a63});
  MAT.stone = new THREEJS.MeshLambertMaterial({color:0x9a9284});
  MAT.crate = new THREEJS.MeshLambertMaterial({color:0x8a6b3a});
  MAT.crate2= new THREEJS.MeshLambertMaterial({color:0x6f5228});
  MAT.metal = new THREEJS.MeshLambertMaterial({color:0x555b60});
  MAT.door  = new THREEJS.MeshLambertMaterial({color:0x4a4a52});
  MAT.floor = new THREEJS.MeshLambertMaterial({color:0xbfa470});
  MAT.floor2= new THREEJS.MeshLambertMaterial({color:0x7d8a6a}); // grass/dirt for other maps
  MAT.concrete=new THREEJS.MeshLambertMaterial({color:0x8f8f96});
  MAT.dark  = new THREEJS.MeshLambertMaterial({color:0x3a3f45});
  MAT.red   = new THREEJS.MeshLambertMaterial({color:0x8a3020});
  MAT.blue  = new THREEJS.MeshLambertMaterial({color:0x24405f});
}

/* ---------------------------- Map framework ------------------------------- */
// Each collider: THREE.Box3. Walls added as meshes (also used to block bullets).
class MapBuilder {
  constructor(scene){ this.scene=scene; this.colliders=[]; this.blockers=[]; this.meshes=[]; }
  box(x,y,z,w,h,d,mat){ // center x,z; y is bottom; w,d footprint; h height
    const g=new THREEJS.BoxGeometry(w,h,d); const m=new THREEJS.Mesh(g,mat);
    m.position.set(x,y+h/2,z); m.castShadow=false; m.receiveShadow=false;
    this.scene.add(m); this.meshes.push(m); this.blockers.push(m);
    this.colliders.push(new THREEJS.Box3(new THREEJS.Vector3(x-w/2,y,z-d/2), new THREEJS.Vector3(x+w/2,y+h,z+d/2)));
    return m;
  }
  // decorative / non-colliding (e.g. thin floor markings)
  plate(x,y,z,w,d,mat){ const g=new THREEJS.PlaneGeometry(w,d); const m=new THREEJS.Mesh(g,mat);
    m.rotation.x=-Math.PI/2; m.position.set(x,y+0.02,z); this.scene.add(m); this.meshes.push(m); return m; }
  ground(w,d,mat){ const g=new THREEJS.PlaneGeometry(w,d); const m=new THREEJS.Mesh(g,mat);
    m.rotation.x=-Math.PI/2; m.position.y=0; this.scene.add(m); this.meshes.push(m); this.blockers.push(m); return m; }
  // low crate that can be jumped/stood on (still a collider)
  crate(x,z,size,mat){ return this.box(x,0,z,size,size,size,mat||MAT.crate); }
}

/* ---------------------------- MAP 1: DUST2 ---------------------------------
   Faithful structural replica. Coordinate scheme:
   - X axis: T-side (negative X, "left/A-long side") to CT/B on the layout.
   - We build the classic footprint: T spawn (south), CT spawn (north),
     A site (east/upper via A-long & catwalk), B site (west, via tunnels),
     Mid connecting them with mid-doors and catwalk.
   Layout uses a top-down grid ~ (-60..60 x, -60..60 z). z-negative = T side (bottom),
   z-positive = CT side (top), matching radar orientation.
---------------------------------------------------------------------------- */
function buildDust2(scene){
  const mb=new MapBuilder(scene);
  const W=MAT.wall, W2=MAT.wall2, C=MAT.crate, C2=MAT.crate2, S=MAT.stone, WH=8;
  mb.ground(200,200,MAT.floor);
  // Perimeter big walls (kept generous)
  const P=WH+4;
  mb.box(0,0,-62,140,P,3,W);   // south wall (behind T)
  mb.box(0,0,62,140,P,3,W);    // north wall (behind CT)
  mb.box(-62,0,0,3,P,140,W);   // west wall (B long side)
  mb.box(62,0,0,3,P,140,W);    // east wall (A long side)

  // ---------- T SPAWN (south center) ----------
  const tSpawn=[]; for(let i=0;i<6;i++) tSpawn.push(V3(-14+i*5.5,0,-54));
  mb.plate(-3,0,-54,40,10,MAT.sandDark);
  // T spawn side walls funneling into three exits: A-long (east), Mid (center), tunnels/B (west)
  mb.box(-3,0,-46,44,WH,2,W2); // wall in front of T with gaps handled by segments below
  // Rebuild the front wall with gaps: (we place segments, leaving openings)
  // Remove concept: instead build segmented front:
  // Left gap -> to Outside Long / A ; Center gap -> Mid ; Right gap -> tunnels

  // ---------- A LONG (east corridor) T -> A ----------
  // Long doors area near T, long stretch up east side to A site.
  mb.box(30,0,-40,2,WH,28,W);    // long west inner wall (separates long from mid)
  mb.box(52,0,-30,2,WH,48,W);    // long east wall
  // Long doors (double doors) at south end of long
  mb.box(35,0,-52,2,WH,10,W);    // segment
  const longDoorL=mb.box(40.5,0,-47.5,1,WH*0.7,5,MAT.door); // "long doors"
  // A site platform (east-north)
  mb.plate(40,0,26,26,26,MAT.sandDark);
  mb.box(30,0,40,2,WH,26,W);     // A site back-left wall
  mb.box(53,0,34,2,WH,40,W);     // A site east wall
  mb.box(40,0,50,26,WH,2,W);     // A site north wall (CT side)
  // A site crates (default bomb plant spot, "下包点位")
  mb.crate(38,22,3,C);           // default plant crate A
  mb.crate(38,25.5,3,C2);
  mb.crate(46,20,3,C);
  mb.box(33,0,18,4,3,4,C2);      // "A坑" low cover pit edge (goose)
  // Barrels / cover on A
  mb.box(44,0,30,3,3,3,C);

  // ---------- CATWALK / A SHORT (mid-right up to A) ----------
  // A short (catwalk) connects mid to A site on the CT-left of A.
  mb.box(24,0,20,2,WH,22,W);     // catwalk west wall
  mb.plate(27,0,14,10,26,MAT.sand);
  mb.box(28,2.2,6,8,0.5,10,S);   // catwalk raised platform ("A斜坡/A小")
  mb.box(23.5,0,4,1,4,10,C2);    // short cover

  // ---------- MID ----------
  // Mid runs from T (south) to CT (north). Mid doors split it.
  mb.plate(6,0,0,20,60,MAT.floor);
  mb.box(16,0,4,2,WH,44,W);      // mid east wall (toward catwalk/A)
  mb.box(-6,0,4,2,WH,44,W);      // mid west wall (toward B/xbox)
  // MID DOORS (双门/中门) - a wall crossing mid with a door gap at ~ z=2
  mb.box(1,0,2,10,WH,2,W2);      // mid doors wall left part
  const midDoorR=mb.box(11,0,2,6,WH*0.7,1.5,MAT.door); // mid doors
  // XBOX (the iconic crate in mid, near mid-doors on T side)
  mb.crate(6,-8,3.2,C);          // XBOX
  mb.box(6,3.2,-8,3.2,0.3,3.2,C2);
  // Mid-to-B "double doors" (west opening from mid to B)
  const dblDoor=mb.box(-6,0,-6,1,WH*0.7,6,MAT.door); // double doors to B

  // ---------- B TUNNELS (west, T -> B) ----------
  // Upper tunnels & lower tunnels leading to B site.
  mb.box(-24,0,-30,2,WH,44,W);   // tunnels east wall
  mb.box(-46,0,-24,2,WH,48,W);   // tunnels west wall
  mb.plate(-35,0,-20,20,52,MAT.sandDark);
  mb.box(-35,0,-48,22,WH,2,W2);  // tunnel south (near T) - entrance from T right
  // "B洞" opening into B site (upper tunnel exit north)
  const bTunEntry=mb.box(-24,0,4,1,WH*0.7,7,MAT.door);

  // ---------- B SITE (west-north) ----------
  mb.plate(-38,0,26,30,30,MAT.sandDark);
  mb.box(-24,0,26,2,WH,22,W);    // B site east wall (toward mid/window)
  mb.box(-54,0,30,2,WH,42,W);    // B site west wall
  mb.box(-38,0,50,32,WH,2,W);    // B site north wall (CT side)
  // B platform ("B平台") raised area at back of B
  mb.box(-46,0,36,10,2.4,10,S);  // B platform
  mb.crate(-34,24,3,C);          // B default plant crate
  mb.crate(-34,27.5,3,C2);
  mb.crate(-42,20,3,C);          // "B洞/car" cover
  mb.box(-30,0,18,3,3,3,C2);     // B door cover

  // ---------- CT SPAWN (north center) ----------
  const ctSpawn=[]; for(let i=0;i<6;i++) ctSpawn.push(V3(-10+i*5,0,54));
  mb.plate(-4,0,54,44,10,MAT.sandDark);
  // CT mid door (警家 area is CT-side near B/mid). Connect CT spawn to A, B, mid.
  mb.box(4,0,44,2,WH,16,W2);     // divider near CT toward mid
  // "警家" (CT window / catwalk to mid). A short wall with window gap to mid.
  mb.box(10,3,30,8,2,1,W);       // mid window ledge (police / CT peek to mid)
  // Connect CT to A (right) and B (left) - openings left as gaps in walls above.

  // ---------- Spawns / sites ----------
  const siteA={pos:V3(40,0,24), r:11, name:"A"};
  const siteB={pos:V3(-36,0,26), r:11, name:"B"};

  // ---------- Callout markers (for radar labels) ----------
  const callouts=[
    {n:"T出生",p:V3(-3,-54)}, {n:"CT出生",p:V3(-4,54)},
    {n:"A大/长",p:V3(45,-20)}, {n:"长门",p:V3(40,-47)},
    {n:"A小/斜坡",p:V3(27,8)}, {n:"A坑",p:V3(33,18)}, {n:"A包点",p:V3(40,24)},
    {n:"中路",p:V3(6,-10)}, {n:"中门",p:V3(6,2)}, {n:"Xbox",p:V3(6,-8)},
    {n:"双门",p:V3(-6,-6)}, {n:"警家",p:V3(10,30)},
    {n:"B洞/隧道",p:V3(-35,-20)}, {n:"B门",p:V3(-24,4)}, {n:"B平台",p:V3(-46,36)}, {n:"B包点",p:V3(-36,26)},
  ];

  // ---------- Bot navigation waypoints (graph) ----------
  const wp=[
    V3(-3,0,-52),   //0 T spawn
    V3(20,0,-46),   //1 into long from T
    V3(42,0,-40),   //2 long doors
    V3(46,0,-20),   //3 long mid
    V3(46,0,4),     //4 long / pit approach
    V3(40,0,20),    //5 A site (plant)
    V3(28,0,-30),   //6 mid entry from T
    V3(6,0,-20),    //7 mid low (xbox)
    V3(6,0,2),      //8 mid doors
    V3(6,0,26),     //9 mid to CT
    V3(28,0,10),    //10 catwalk/A short
    V3(-3,0,-46),   //11 tunnel entry from T
    V3(-35,0,-30),  //12 tunnels
    V3(-30,0,0),    //13 upper tunnel exit / B door
    V3(-36,0,24),   //14 B site (plant)
    V3(-46,0,36),   //15 B platform
    V3(-4,0,52),    //16 CT spawn
    V3(20,0,48),    //17 CT to A
    V3(-24,0,48),   //18 CT to B
    V3(6,0,44),     //19 CT to mid (police)
  ];
  const edges=[
    [0,1],[0,6],[0,11],[1,2],[2,3],[3,4],[4,5],[5,10],[10,9],[9,8],[8,7],[7,6],
    [11,12],[12,13],[13,14],[14,15],[16,17],[16,18],[16,19],[17,5],[18,14],[19,9],
    [9,16],[4,10],[13,14],[8,9],[3,10]
  ];

  return {
    name:"Dust2 (复刻)", mb, colliders:mb.colliders, blockers:mb.blockers,
    tSpawn, ctSpawn, siteA, siteB, callouts, wp, edges,
    radar:{minX:-62,maxX:62,minZ:-62,maxZ:62},
    ambient:0x6a5a3a, sky:0xc9b98a,
  };
}

/* ---------------------------- MAP 2: WAREHOUSE (Foundry) -------------------- */
function buildWarehouse(scene){
  const mb=new MapBuilder(scene);
  const W=MAT.metal, C=MAT.crate, C2=MAT.crate2, WH=7;
  mb.ground(160,160,MAT.concrete);
  mb.box(0,0,-52,120,WH+3,3,W); mb.box(0,0,52,120,WH+3,3,W);
  mb.box(-52,0,0,3,WH+3,120,W); mb.box(52,0,0,3,WH+3,120,W);
  // central hall with pillars
  for(let i=-1;i<=1;i++) for(let j=-1;j<=1;j++) mb.box(i*22,0,j*16,3,WH,3,MAT.dark);
  // crate clusters (cover)
  const cr=[[ -30,-20],[ -26,-24],[30,20],[34,24],[0,-30],[4,-30],[-4,30],[0,30],[38,-10],[-38,10]];
  cr.forEach(([x,z])=>mb.crate(x,z,2.4,C));
  cr.forEach(([x,z])=>{ if(Math.random()<0.5) mb.crate(x+2.4,z,2.4,C2); });
  // site A (east), site B (west)
  mb.box(30,0,-40,2,WH,16,W); mb.box(30,0,40,2,WH,16,W);
  mb.crate(40,26,3,C); mb.crate(43,29,3,C2);
  mb.crate(-40,-26,3,C); mb.crate(-43,-29,3,C2);
  const siteA={pos:V3(40,0,28),r:10,name:"A"}, siteB={pos:V3(-40,0,-28),r:10,name:"B"};
  const tSpawn=[],ctSpawn=[];
  for(let i=0;i<6;i++){ tSpawn.push(V3(-12+i*5,0,-46)); ctSpawn.push(V3(-12+i*5,0,46)); }
  const callouts=[{n:"T出生",p:V3(0,-46)},{n:"CT出生",p:V3(0,46)},{n:"A库",p:V3(40,28)},{n:"B库",p:V3(-40,-28)},{n:"中厅",p:V3(0,0)}];
  const wp=[V3(0,0,-46),V3(20,0,-30),V3(40,0,-10),V3(40,0,20),V3(40,0,28),
    V3(0,0,-16),V3(0,0,0),V3(0,0,16),V3(-20,0,30),V3(-40,0,10),V3(-40,0,-20),V3(-40,0,-28),
    V3(0,0,46),V3(24,0,30),V3(-24,0,-30)];
  const edges=[[0,1],[1,2],[2,3],[3,4],[0,5],[5,6],[6,7],[7,8],[8,9],[9,10],[10,11],
    [12,13],[13,4],[12,7],[6,2],[6,9],[12,8],[0,10]];
  return {name:"Warehouse", mb, colliders:mb.colliders, blockers:mb.blockers, tSpawn, ctSpawn, siteA, siteB, callouts, wp, edges,
    radar:{minX:-52,maxX:52,minZ:-52,maxZ:52}, ambient:0x445055, sky:0x3a4248};
}

/* ---------------------------- MAP 3: VILLA (Inferno-ish) ------------------- */
function buildVilla(scene){
  const mb=new MapBuilder(scene);
  const W=MAT.wall2, S=MAT.stone, C=MAT.crate, WH=7;
  mb.ground(170,170,MAT.floor2);
  mb.box(0,0,-56,130,WH+3,3,W); mb.box(0,0,56,130,WH+3,3,W);
  mb.box(-56,0,0,3,WH+3,130,W); mb.box(56,0,0,3,WH+3,130,W);
  // Apartments / banana style split: two lanes
  mb.box(6,0,-10,2,WH,60,S);   // central long wall separating A-apartments and banana
  mb.box(-20,0,10,2,WH,40,S);
  mb.box(26,0,10,2,WH,40,S);
  // A site (top-right), B site (top-left, "banana")
  mb.box(30,0,44,2,WH,20,S); mb.box(52,0,40,2,WH,28,S);
  mb.crate(42,40,3,C); mb.crate(45,43,3,MAT.crate2);
  mb.box(-30,0,44,2,WH,20,S); mb.box(-52,0,40,2,WH,28,S);
  mb.crate(-42,40,3,C); mb.crate(-45,37,3,MAT.crate2);
  // scattered cover
  [[15,0],[-15,0],[0,20],[0,-24],[30,-10],[-30,-10],[18,30],[-18,30]].forEach(([x,z])=>mb.crate(x,z,2.4,C));
  // raised balcony on A
  mb.box(36,2.4,30,10,0.5,8,S);
  const siteA={pos:V3(42,0,42),r:10,name:"A"}, siteB={pos:V3(-42,0,42),r:10,name:"B"};
  const tSpawn=[],ctSpawn=[];
  for(let i=0;i<6;i++){ tSpawn.push(V3(-12+i*5,0,-50)); ctSpawn.push(V3(-12+i*5,0,50)); }
  const callouts=[{n:"T出生",p:V3(0,-50)},{n:"CT出生",p:V3(0,50)},{n:"A公寓",p:V3(42,42)},{n:"B香蕉",p:V3(-42,42)},{n:"中庭",p:V3(0,0)}];
  const wp=[V3(0,0,-50),V3(18,0,-30),V3(36,0,0),V3(40,0,26),V3(42,0,40),
    V3(-18,0,-30),V3(-36,0,0),V3(-40,0,26),V3(-42,0,40),
    V3(0,0,0),V3(0,0,30),V3(0,0,50),V3(24,0,44),V3(-24,0,44)];
  const edges=[[0,1],[1,2],[2,3],[3,4],[0,5],[5,6],[6,7],[7,8],[0,9],[9,10],[10,11],
    [11,12],[12,4],[11,13],[13,8],[9,2],[9,6]];
  return {name:"Villa", mb, colliders:mb.colliders, blockers:mb.blockers, tSpawn, ctSpawn, siteA, siteB, callouts, wp, edges,
    radar:{minX:-56,maxX:56,minZ:-56,maxZ:56}, ambient:0x556040, sky:0x9fb0c0};
}

const MAPS = {
  dust2:{label:"Dust2 (复刻)", build:buildDust2},
  warehouse:{label:"Warehouse", build:buildWarehouse},
  villa:{label:"Villa", build:buildVilla},
};

/* ---------------------------- Character model ----------------------------- */
function makeSoldier(team){
  const g=new THREEJS.Group();
  const col = team==="T" ? 0x7a5a35 : 0x3a5068;
  const skin = 0xc9a37a;
  const bodyMat=new THREEJS.MeshLambertMaterial({color:col});
  const headMat=new THREEJS.MeshLambertMaterial({color:skin});
  const legMat=new THREEJS.MeshLambertMaterial({color:team==="T"?0x5a4526:0x2a3a4a});
  // torso
  const torso=new THREEJS.Mesh(new THREEJS.BoxGeometry(0.62,0.75,0.35),bodyMat); torso.position.y=1.15; g.add(torso);
  // vest chest highlight
  const vest=new THREEJS.Mesh(new THREEJS.BoxGeometry(0.5,0.5,0.15),new THREEJS.MeshLambertMaterial({color:team==="T"?0x3d3020:0x24303c}));
  vest.position.set(0,1.2,0.2); g.add(vest);
  // head
  const head=new THREEJS.Mesh(new THREEJS.BoxGeometry(0.3,0.32,0.3),headMat); head.position.y=1.68; g.add(head);
  // helmet
  const helm=new THREEJS.Mesh(new THREEJS.BoxGeometry(0.34,0.16,0.34),new THREEJS.MeshLambertMaterial({color:team==="T"?0x4a3a24:0x2c3844}));
  helm.position.y=1.8; g.add(helm);
  // arms
  const armGeo=new THREEJS.BoxGeometry(0.16,0.62,0.16);
  const armL=new THREEJS.Mesh(armGeo,bodyMat); armL.position.set(-0.4,1.15,0); g.add(armL);
  const armR=new THREEJS.Mesh(armGeo,bodyMat); armR.position.set(0.4,1.15,0); g.add(armR);
  // legs
  const legGeo=new THREEJS.BoxGeometry(0.2,0.8,0.2);
  const legL=new THREEJS.Mesh(legGeo,legMat); legL.position.set(-0.16,0.4,0); g.add(legL);
  const legR=new THREEJS.Mesh(legGeo,legMat); legR.position.set(0.16,0.4,0); g.add(legR);
  // gun stub
  const gun=new THREEJS.Mesh(new THREEJS.BoxGeometry(0.1,0.12,0.6),MAT.dark); gun.position.set(0.32,1.1,0.35); g.add(gun);
  g.userData.parts={torso,head,legL,legR,armL,armR};
  return g;
}

/* ---------------------------- Viewmodel (first person gun) ---------------- */
function makeViewmodel(){
  const g=new THREEJS.Group();
  const body=new THREEJS.Mesh(new THREEJS.BoxGeometry(0.09,0.14,0.5),new THREEJS.MeshLambertMaterial({color:0x23262b}));
  body.position.set(0.22,-0.22,-0.55); g.add(body);
  const barrel=new THREEJS.Mesh(new THREEJS.CylinderGeometry(0.02,0.02,0.4,8),new THREEJS.MeshLambertMaterial({color:0x15171a}));
  barrel.rotation.x=Math.PI/2; barrel.position.set(0.22,-0.2,-0.85); g.add(barrel);
  const mag=new THREEJS.Mesh(new THREEJS.BoxGeometry(0.06,0.18,0.08),new THREEJS.MeshLambertMaterial({color:0x2a2d33}));
  mag.position.set(0.22,-0.34,-0.5); g.add(mag);
  const grip=new THREEJS.Mesh(new THREEJS.BoxGeometry(0.05,0.14,0.08),new THREEJS.MeshLambertMaterial({color:0x1a1c20}));
  grip.position.set(0.22,-0.33,-0.38); grip.rotation.x=0.3; g.add(grip);
  // muzzle flash
  const flash=new THREEJS.Mesh(new THREEJS.SphereGeometry(0.09,6,6),new THREEJS.MeshBasicMaterial({color:0xffdd66,transparent:true,opacity:0}));
  flash.position.set(0.22,-0.2,-1.05); g.add(flash);
  g.userData.flash=flash; g.userData.body=body; g.userData.barrel=barrel;
  return g;
}

/* ---------------------------- Input --------------------------------------- */
const Input={ keys:{}, mouseDX:0, mouseDY:0, mouseDown:false, rmb:false, locked:false };
function initInput(canvas){
  document.addEventListener("keydown",e=>{ Input.keys[e.code]=true; });
  document.addEventListener("keyup",e=>{ Input.keys[e.code]=false; });
  document.addEventListener("mousemove",e=>{ if(Input.locked){ Input.mouseDX+=e.movementX; Input.mouseDY+=e.movementY; }});
  document.addEventListener("mousedown",e=>{ if(!Input.locked) return; if(e.button===0)Input.mouseDown=true; if(e.button===2)Input.rmb=true; });
  document.addEventListener("mouseup",e=>{ if(e.button===0)Input.mouseDown=false; if(e.button===2)Input.rmb=false; });
  document.addEventListener("contextmenu",e=>e.preventDefault());
  document.addEventListener("pointerlockchange",()=>{ Input.locked=(document.pointerLockElement===canvas); });
}

/* ============================================================================
   GAME
   ========================================================================== */
const Game={
  scene:null, camera:null, renderer:null, canvas:null, clock:null,
  map:null, mapKey:"dust2", playerTeam:"T", botCount:4, diff:"normal", winScore:8,
  entities:[], player:null, bombPlanted:false, bombPos:null, bombMesh:null, bombTimer:0, bombCarrier:null,
  planting:false, plantProg:0, defusing:false, defuseProg:0, defuser:null,
  phase:"menu", // menu, freeze, live, roundend, matchend, paused
  roundTimeLeft:0, freezeLeft:0, scoreT:0, scoreCT:0, roundNum:0,
  tLossStreak:0, ctLossStreak:0, buyOpen:false, canBuy:false,
  effects:[], smokes:[], fires:[], decals:[], raycaster:new THREEJS.Raycaster(),
  flashAmount:0, msgTimer:0, roundEndTimer:0, pendingWinner:null,
};

const DIFF={
  easy:  {react:0.55, aimErr:0.09, aimSpeed:2.4, fireDisc:0.5, nadeChance:0.15, fov:1.7},
  normal:{react:0.35, aimErr:0.05, aimSpeed:3.6, fireDisc:0.7, nadeChance:0.3, fov:1.9},
  hard:  {react:0.20, aimErr:0.028,aimSpeed:5.5, fireDisc:0.85,nadeChance:0.45,fov:2.1},
  expert:{react:0.11, aimErr:0.014,aimSpeed:8.0, fireDisc:0.95,nadeChance:0.6, fov:2.3},
};

/* --------- Entity (player OR bot share this shape) --------- */
function makeEntity(team,isBot,name){
  return {
    team, isBot, name, alive:true, hp:100, armor:0, helmet:false, hasDefuser:false,
    pos:V3(), vel:V3(), yaw:0, pitch:0, height:CFG.playerHeight, crouch:false, onGround:true,
    money:CFG.startMoney,
    weapons:{ primary:null, secondary:"glock", knife:"knife" },
    current:"secondary", nades:{he:0,flash:0,smoke:0,molly:0},
    ammo:{}, // per weaponKey {clip,reserve}
    fireCd:0, reloading:0, reloadTimeLeft:0, switchCd:0, recoilAccum:0, spreadHeat:0,
    kills:0, deaths:0, mesh:null, hitboxBody:null, hitboxHead:null,
    // bot ai
    ai:{ state:"idle", target:null, path:[], pathIdx:0, seeTimer:0, lastSeen:null, reactTimer:0,
         goal:null, repathTimer:0, wanderTimer:0, blindTimer:0, plantTimer:0, strafeDir:1, strafeTimer:0 },
    flash:0,
  };
}

function currentWeaponKey(e){
  if(e.current==="primary") return e.weapons.primary;
  if(e.current==="secondary") return e.weapons.secondary;
  if(e.current==="knife") return "knife";
  if(e.current==="he"||e.current==="flash"||e.current==="smoke"||e.current==="molly") return e.current;
  return e.weapons.secondary;
}
function ensureAmmo(e,key){ if(!WEAPONS[key]||WEAPONS[key].melee) return;
  if(!e.ammo[key]) e.ammo[key]={clip:WEAPONS[key].mag, reserve:WEAPONS[key].reserve}; }

/* --------------------------- Init & scene setup --------------------------- */
function initThree(){
  Game.canvas=document.getElementById("game");
  Game.renderer=new THREEJS.WebGLRenderer({canvas:Game.canvas,antialias:true});
  Game.renderer.setPixelRatio(Math.min(window.devicePixelRatio,1.6));
  Game.renderer.setSize(window.innerWidth,window.innerHeight);
  Game.scene=new THREEJS.Scene();
  Game.camera=new THREEJS.PerspectiveCamera(75,window.innerWidth/window.innerHeight,0.05,600);
  Game.clock=new THREEJS.Clock();
  window.addEventListener("resize",()=>{
    Game.camera.aspect=window.innerWidth/window.innerHeight; Game.camera.updateProjectionMatrix();
    Game.renderer.setSize(window.innerWidth,window.innerHeight);
  });
  initMaterials();
}

function clearScene(){
  // remove all but keep nothing; rebuild lights each round-start map load
  while(Game.scene.children.length) Game.scene.remove(Game.scene.children[0]);
  Game.entities=[]; Game.effects=[]; Game.smokes=[]; Game.fires=[]; Game.decals=[];
}

function loadMap(key){
  clearScene();
  const def=MAPS[key].build(Game.scene);
  Game.map=def;
  // lights
  const amb=new THREEJS.AmbientLight(def.ambient||0x606060, 1.1);
  Game.scene.add(amb);
  const sun=new THREEJS.DirectionalLight(0xfff0d0, 0.9); sun.position.set(40,80,20); Game.scene.add(sun);
  const sun2=new THREEJS.DirectionalLight(0x99aabb, 0.3); sun2.position.set(-40,60,-30); Game.scene.add(sun2);
  Game.scene.fog=new THREEJS.Fog(def.sky||0xbbbbbb, 60, 220);
  Game.scene.background=new THREEJS.Color(def.sky||0xbbbbbb);
  // viewmodel
  Game.viewmodel=makeViewmodel();
  Game.camera.add(Game.viewmodel);
  Game.scene.add(Game.camera);
}

/* --------------------------- Collision ------------------------------------ */
function resolveCollision(pos, radius){
  // per-axis push out against colliders (Box3 list). pos is feet position at y.
  const cs=Game.map.colliders;
  for(let iter=0;iter<2;iter++){
    for(const b of cs){
      // only collide within height band of the box for the player's body
      if(pos.y+ (CFG.playerHeight) < b.min.y || pos.y > b.max.y) continue;
      const minx=b.min.x-radius, maxx=b.max.x+radius, minz=b.min.z-radius, maxz=b.max.z+radius;
      if(pos.x>minx && pos.x<maxx && pos.z>minz && pos.z<maxz){
        // find least penetration
        const pl=pos.x-minx, pr=maxx-pos.x, pd=pos.z-minz, pu=maxz-pos.z;
        const m=Math.min(pl,pr,pd,pu);
        if(m===pl) pos.x=minx; else if(m===pr) pos.x=maxx;
        else if(m===pd) pos.z=minz; else pos.z=maxz;
      }
    }
  }
}
function groundHeightAt(pos){
  // returns the top surface Y the player stands on (crates), else 0
  let h=0; const cs=Game.map.colliders; const r=CFG.radius;
  for(const b of cs){
    if(pos.x>b.min.x-r && pos.x<b.max.x+r && pos.z>b.min.z-r && pos.z<b.max.z+r){
      // standable if box top is reasonably low (<=2.6) and player is above it
      if(b.max.y<=2.7 && pos.y>=b.max.y-0.3 && b.max.y>h) h=b.max.y;
    }
  }
  return h;
}

/* --------------------------- Line of sight -------------------------------- */
const _rc=new THREEJS.Raycaster();
function hasLOS(from,to){
  const dir=to.clone().sub(from); const dist=dir.length(); dir.normalize();
  _rc.set(from,dir); _rc.far=dist-0.4;
  const hits=_rc.intersectObjects(Game.map.blockers,false);
  if(hits.length>0) return false;
  // smoke blocks LOS
  for(const s of Game.smokes){ if(s.t<s.dur && s.active){
    // distance from line to smoke center < radius
    const c=s.pos; const ap=c.clone().sub(from); const t=Math.max(0,Math.min(dist,ap.dot(dir)));
    const closest=from.clone().add(dir.clone().multiplyScalar(t));
    if(closest.distanceTo(c)<s.r*0.85) return false;
  }}
  return true;
}

/* --------------------------- Spawning ------------------------------------- */
function eyePos(e){ return V3(e.pos.x, e.pos.y+(e.crouch?CFG.crouchHeight:CFG.playerHeight)-0.2, e.pos.z); }

function spawnRound(){
  // reset entities positions & health, assign spawns
  const tSp=Game.map.tSpawn, ctSp=Game.map.ctSpawn;
  let ti=0,ci=0;
  for(const e of Game.entities){
    e.alive=true; e.hp=100; e.vel.set(0,0,0); e.flash=0; e.reloading=0; e.fireCd=0;
    e.ai.state="idle"; e.ai.target=null; e.ai.path=[]; e.ai.blindTimer=0; e.ai.plantTimer=0;
    const sp = e.team==="T" ? tSp[ti++ % tSp.length] : ctSp[ci++ % ctSp.length];
    e.pos.copy(sp); e.pos.y=0; e.yaw = e.team==="T" ? 0 : Math.PI;
    // refill clips
    for(const k in e.ammo){ e.ammo[k].clip=WEAPONS[k].mag; }
    // ensure weapon selected
    e.current = e.weapons.primary ? "primary" : "secondary";
    if(e.mesh){ e.mesh.visible=(e!==Game.player); }
    updateHitboxes(e);
  }
  // bomb assignment: a random living T carries bomb
  Game.bombPlanted=false; Game.bombPos=null; Game.bombCarrier=null;
  if(Game.bombMesh){ Game.scene.remove(Game.bombMesh); Game.bombMesh=null; }
  const ts=Game.entities.filter(e=>e.team==="T");
  if(ts.length) Game.bombCarrier=ts[Math.floor(Math.random()*ts.length)];
  // clear smokes/fires/decals visuals
  for(const s of Game.smokes){ if(s.group) Game.scene.remove(s.group); }
  for(const f of Game.fires){ if(f.group) Game.scene.remove(f.group); }
  Game.smokes=[]; Game.fires=[];
}

function updateHitboxes(e){
  if(!e.mesh) return;
  e.mesh.position.set(e.pos.x, e.pos.y, e.pos.z);
  e.mesh.rotation.y=e.yaw;
  e.mesh.visible = e.alive && e!==Game.player;
}

/* --------------------------- Combat --------------------------------------- */
function collectTargetHitboxes(shooter){
  // returns array of {mesh, entity, isHead}
  const arr=[];
  for(const e of Game.entities){
    if(!e.alive || e.team===shooter.team || e===shooter) continue;
    if(e.hitboxBody) arr.push(e.hitboxBody);
    if(e.hitboxHead) arr.push(e.hitboxHead);
  }
  return arr;
}

function fireBullet(shooter, dirOverride){
  const key=currentWeaponKey(shooter);
  const w=WEAPONS[key];
  if(!w) return;
  // origin & base dir
  const origin=eyePos(shooter);
  let dir;
  if(dirOverride) dir=dirOverride.clone();
  else {
    dir=V3(0,0,-1);
    dir.applyEuler(new THREEJS.Euler(shooter.pitch, shooter.yaw, 0, "YXZ"));
  }
  // spread (increased by movement & heat)
  let spread=w.spread;
  const moving=shooter.vel.lengthSq()>1.2;
  if(moving && !w.melee) spread*= shooter===Game.player?2.4:1.8;
  spread += shooter.spreadHeat*0.5;
  if(shooter===Game.player && Game.player.rmbZoom && w.zoom>0) spread*=0.15;
  const pellets = w.pellets||1;
  const targets=collectTargetHitboxes(shooter);
  for(let p=0;p<pellets;p++){
    const d=dir.clone();
    d.x+=(Math.random()-0.5)*spread*2; d.y+=(Math.random()-0.5)*spread*2; d.z+=(Math.random()-0.5)*spread*2;
    d.normalize();
    _rc.set(origin,d); _rc.far=w.range||150;
    // combine world + targets
    const objs=targets.concat(Game.map.blockers);
    const hits=_rc.intersectObjects(objs,false);
    if(hits.length){
      const h=hits[0];
      const hd=h.object.userData;
      if(hd && hd.entity){
        applyDamage(hd.entity, shooter, w, hd.isHead, h.point);
        if(shooter===Game.player){ showHitmarker(hd.isHead); Audio.hit(); }
      } else {
        // wall impact decal + particles
        spawnImpact(h.point, h.face? h.face.normal : V3(0,1,0));
      }
      if(!dirOverride && p===0) shooter._lastTracer={a:origin.clone(),b:h.point.clone()};
      if(p===0) spawnTracer(origin, h.point);
    } else {
      const far=origin.clone().add(d.multiplyScalar(w.range||150));
      if(p===0) spawnTracer(origin, far);
    }
  }
  // recoil & heat
  shooter.spreadHeat=Math.min(0.06, shooter.spreadHeat + w.recoil*0.4);
  if(shooter===Game.player){
    Game.player.pitch -= w.recoil*(0.7+Math.random()*0.5);
    Game.player.yaw += (Math.random()-0.5)*w.recoil*0.6;
    kickViewmodel();
    muzzleFlash();
  }
  // audio
  let sType = w.cat==="rifle"?"rifle": w.cat==="pistol"?"pistol": w.cat==="smg"?"smg":
    (key==="awp"||key==="ssg08")?"awp":"heavy";
  const dist = shooter===Game.player?null:shooter.pos.distanceTo(Game.player.pos);
  Audio.shot(sType, dist);
}

function applyDamage(target, attacker, w, isHead, point){
  if(!target.alive) return;
  let dmg=w.dmg;
  // distance falloff for shotguns
  if(w.pellets){ const dd=attacker? eyePos(attacker).distanceTo(target.pos):10; dmg*=Math.max(0.35,1-dd/50); }
  if(isHead){ dmg*=w.hs; }
  // armor
  if(target.armor>0 && (!isHead || target.helmet)){
    // armor absorbs part; armorPen determines how much passes
    const passed=dmg*w.armorPen;
    const absorbed=dmg-passed;
    target.armor=Math.max(0, target.armor - absorbed*0.5);
    dmg=passed + absorbed*0.5*0.5; // reduced
  }
  dmg=Math.round(dmg);
  target.hp-=dmg;
  if(target===Game.player){ Game.flashAmount=Math.min(1,Game.flashAmount+dmg/70); damageFlash(); }
  if(target.hp<=0){ killEntity(target, attacker, isHead); }
  else if(target.isBot){ // alert bot
    target.ai.lastSeen=attacker? attacker.pos.clone():null; target.ai.reactTimer=0;
    if(target.ai.state==="idle"||target.ai.state==="rotate") target.ai.state="engage";
  }
}

function killEntity(e, attacker, isHead){
  e.alive=false; e.hp=0; e.deaths++;
  if(attacker && attacker!==e){ attacker.kills++; attacker.money=Math.min(CFG.maxMoney,attacker.money+CFG.reward.kill); }
  if(e.mesh) e.mesh.visible=false;
  // drop bomb
  if(Game.bombCarrier===e && !Game.bombPlanted){
    Game.bombCarrier=null;
    // reassign to nearest living T
    const ts=Game.entities.filter(x=>x.team==="T"&&x.alive);
    if(ts.length){ ts.sort((a,b)=>a.pos.distanceTo(e.pos)-b.pos.distanceTo(e.pos)); Game.bombCarrier=ts[0]; }
  }
  addKillFeed(attacker,e,isHead);
  if(e===Game.player){ onPlayerDeath(); }
  Audio.beep(220);
  checkRoundEnd();
}

/* --------------------------- Effects -------------------------------------- */
function spawnTracer(a,b){
  const geo=new THREEJS.BufferGeometry().setFromPoints([a,b]);
  const line=new THREEJS.Line(geo,new THREEJS.LineBasicMaterial({color:0xffe08a,transparent:true,opacity:0.7}));
  Game.scene.add(line); Game.effects.push({obj:line,t:0,dur:0.06,type:"tracer"});
}
function spawnImpact(point,normal){
  // small decal
  const g=new THREEJS.PlaneGeometry(0.18,0.18);
  const m=new THREEJS.Mesh(g,new THREEJS.MeshBasicMaterial({color:0x1a1a1a,transparent:true,opacity:0.85,side:THREEJS.DoubleSide}));
  m.position.copy(point).add(normal.clone().multiplyScalar(0.02));
  m.lookAt(point.clone().add(normal));
  Game.scene.add(m); Game.decals.push(m); if(Game.decals.length>60){ Game.scene.remove(Game.decals.shift()); }
  // spark particles
  for(let i=0;i<4;i++){ const p=new THREEJS.Mesh(new THREEJS.SphereGeometry(0.03,4,4),new THREEJS.MeshBasicMaterial({color:0xffcc66}));
    p.position.copy(point); const v=normal.clone().multiplyScalar(2).add(V3((Math.random()-0.5)*3,(Math.random())*2,(Math.random()-0.5)*3));
    Game.scene.add(p); Game.effects.push({obj:p,t:0,dur:0.3,type:"spark",vel:v}); }
}
function muzzleFlash(){ const f=Game.viewmodel.userData.flash; f.material.opacity=1; f.scale.setScalar(1+Math.random());
  Game.effects.push({obj:f,t:0,dur:0.05,type:"muzzle"}); }
let vmKick=0;
function kickViewmodel(){ vmKick=Math.min(0.5,vmKick+0.12); }
function showHitmarker(hs){ const hm=document.getElementById("hitmarker"); hm.classList.remove("show");
  hm.style.borderColor=hs?"#ff4040":"#fff"; void hm.offsetWidth; hm.classList.add("show");
  hm.querySelectorAll && hm.style.filter=hs?"drop-shadow(0 0 3px #f00)":"none"; }
function damageFlash(){ const d=document.getElementById("damageflash");
  d.style.boxShadow="inset 0 0 140px rgba(200,20,20,0.55)"; setTimeout(()=>d.style.boxShadow="inset 0 0 140px rgba(200,20,20,0)",120); }

/* --------------------------- Grenades ------------------------------------- */
function throwNade(shooter,type){
  const origin=eyePos(shooter);
  const dir=V3(0,0,-1).applyEuler(new THREEJS.Euler(shooter.pitch,shooter.yaw,0,"YXZ"));
  const vel=dir.multiplyScalar(20).add(V3(0,3,0));
  const mesh=new THREEJS.Mesh(new THREEJS.SphereGeometry(0.12,8,8),new THREEJS.MeshLambertMaterial({color:NADES[type].color}));
  mesh.position.copy(origin); Game.scene.add(mesh);
  Game.effects.push({type:"nade",nadeType:type,obj:mesh,pos:origin.clone(),vel,t:0,dur:(type==="he"||type==="flash")?1.8:2.2,owner:shooter,bounces:0});
}
function nadePhysics(n,dt){
  n.vel.y-=CFG.gravity*dt;
  const np=n.pos.clone().add(n.vel.clone().multiplyScalar(dt));
  // ground
  if(np.y<0.12){ np.y=0.12; n.vel.y*=-0.4; n.vel.x*=0.6; n.vel.z*=0.6; n.bounces++; }
  // walls (simple)
  for(const b of Game.map.colliders){
    if(np.x>b.min.x-0.12&&np.x<b.max.x+0.12&&np.z>b.min.z-0.12&&np.z<b.max.z+0.12&&np.y<b.max.y+0.1){
      // reflect
      const overlapX=Math.min(np.x-(b.min.x-0.12),(b.max.x+0.12)-np.x);
      const overlapZ=Math.min(np.z-(b.min.z-0.12),(b.max.z+0.12)-np.z);
      if(overlapX<overlapZ){ n.vel.x*=-0.5; np.x=n.pos.x; } else { n.vel.z*=-0.5; np.z=n.pos.z; }
      n.bounces++;
    }
  }
  n.pos.copy(np); n.obj.position.copy(np);
}
function detonateNade(n){
  Game.scene.remove(n.obj);
  const t=n.nadeType;
  if(t==="he"){
    Audio.explosion(n.pos.distanceTo(Game.player.pos));
    explosionEffect(n.pos,0xffaa33,3.5);
    for(const e of Game.entities){ if(!e.alive) continue; const d=e.pos.distanceTo(n.pos);
      if(d<7 && hasLOS(n.pos.clone().add(V3(0,0.5,0)), eyePos(e))){ const dmg=Math.round(70*(1-d/7));
        e.hp-=dmg; if(e===Game.player){Game.flashAmount=Math.min(1,Game.flashAmount+dmg/80);damageFlash();}
        if(e.hp<=0) killEntity(e, n.owner, false); } }
  } else if(t==="flash"){
    Audio.flashbang(); explosionEffect(n.pos,0xffffff,2);
    for(const e of Game.entities){ if(!e.alive) continue;
      const eye=eyePos(e); const toN=n.pos.clone().sub(eye).normalize();
      const look=V3(0,0,-1).applyEuler(new THREEJS.Euler(e.pitch,e.yaw,0,"YXZ"));
      const facing=look.dot(toN); // 1 = looking straight at
      const d=eye.distanceTo(n.pos);
      if(d<18 && hasLOS(n.pos,eye)){
        let amt=(1-d/18)*(0.4+0.6*Math.max(0,facing));
        if(e===Game.player){ Game.flashAmount=Math.min(1.5,Game.flashAmount+amt*1.6); }
        else { e.ai.blindTimer=Math.max(e.ai.blindTimer, amt*3.5); e.flash=amt; }
      }
    }
  } else if(t==="smoke"){
    spawnSmoke(n.pos.clone());
  } else if(t==="molly"){
    spawnFire(n.pos.clone(), n.owner);
  }
}
function explosionEffect(pos,color,size){
  const m=new THREEJS.Mesh(new THREEJS.SphereGeometry(size,10,10),new THREEJS.MeshBasicMaterial({color,transparent:true,opacity:0.8}));
  m.position.copy(pos); m.position.y+=0.5; Game.scene.add(m);
  Game.effects.push({obj:m,t:0,dur:0.35,type:"boom"});
}
function spawnSmoke(pos){
  const group=new THREEJS.Group();
  for(let i=0;i<14;i++){ const s=new THREEJS.Mesh(new THREEJS.SphereGeometry(1.6+Math.random(),8,8),
    new THREEJS.MeshLambertMaterial({color:0xcccccc,transparent:true,opacity:0.9}));
    s.position.set((Math.random()-0.5)*3,0.6+Math.random()*2.6,(Math.random()-0.5)*3); group.add(s); }
  group.position.copy(pos); group.position.y=0; Game.scene.add(group);
  Game.smokes.push({pos:pos.clone().add(V3(0,1.4,0)),r:3.2,dur:15,t:0,group,active:true});
  Audio.beep(300);
}
function spawnFire(pos,owner){
  const group=new THREEJS.Group();
  for(let i=0;i<10;i++){ const f=new THREEJS.Mesh(new THREEJS.ConeGeometry(0.4,1.0,6),
    new THREEJS.MeshBasicMaterial({color:0xff6a1a,transparent:true,opacity:0.85}));
    f.position.set((Math.random()-0.5)*3.6,0.5,(Math.random()-0.5)*3.6); group.add(f); }
  group.position.copy(pos); group.position.y=0; Game.scene.add(group);
  Game.fires.push({pos:pos.clone(),r:2.6,dur:7,t:0,group,owner,tick:0});
  Audio.explosion(pos.distanceTo(Game.player.pos));
}

/* --------------------------- Round / economy ------------------------------ */
function giveRoundRewards(winner, reason){
  // winners
  for(const e of Game.entities){
    if(e.team===winner){ e.money=Math.min(CFG.maxMoney,e.money+CFG.reward.win); }
    else {
      const streak = winner==="T"? ++Game.ctLossStreak : ++Game.tLossStreak; // loser streak handled below
    }
  }
  // fix streak logic
  if(winner==="T"){ Game.tLossStreak=0; }
  else { Game.ctLossStreak=0; }
  const loser = winner==="T"?"CT":"T";
  const streakIdx = Math.min(4, (loser==="T"?Game.tLossStreak:Game.ctLossStreak));
  const lossBonus=CFG.lossBonusTiers[Math.max(0,streakIdx-1)]||1400;
  for(const e of Game.entities){ if(e.team===loser){ e.money=Math.min(CFG.maxMoney,e.money+lossBonus); } }
  // objective bonuses
  if(reason==="defuse"){ for(const e of Game.entities){ if(e.team==="CT"&&e.alive) e.money=Math.min(CFG.maxMoney,e.money+CFG.reward.defuse);} }
  if(Game.bombPlanted){ for(const e of Game.entities){ if(e.team==="T"&&e.alive) e.money=Math.min(CFG.maxMoney,e.money+300);} }
}

function checkRoundEnd(){
  if(Game.phase!=="live") return;
  const tAlive=Game.entities.filter(e=>e.team==="T"&&e.alive).length;
  const ctAlive=Game.entities.filter(e=>e.team==="CT"&&e.alive).length;
  if(Game.bombPlanted){
    // T can win only by explosion (timer) or eliminating CT; CT win by defuse or eliminating T before plant is moot
    if(ctAlive===0){ endRound("T","elim"); return; }
    // if all T dead but bomb planted, round continues until defuse/explode
    return;
  }
  if(tAlive===0){ endRound("CT","elim"); return; }
  if(ctAlive===0){ endRound("T","elim"); return; }
}

function endRound(winner, reason){
  if(Game.phase!=="live") return;
  Game.phase="roundend"; Game.roundEndTimer=4; Game.pendingWinner=winner;
  if(winner==="T") Game.scoreT++; else Game.scoreCT++;
  giveRoundRewards(winner, reason);
  const reasonTxt={elim:"消灭全部敌人",time:"时间耗尽 CT 防守成功",explode:"炸弹爆炸",defuse:"炸弹已拆除"}[reason]||"";
  showRoundBanner((winner==="T"?"恐怖分子":"反恐精英")+" 获胜 — "+reasonTxt);
  Audio.beep(winner===Game.playerTeam?880:330);
  updateHUD();
  // match end?
  if(Game.scoreT>=Game.winScore || Game.scoreCT>=Game.winScore){
    setTimeout(()=>matchEnd(),1800);
  }
}
function matchEnd(){
  Game.phase="matchend";
  const w=Game.scoreT>Game.scoreCT?"恐怖分子 (T)":"反恐精英 (CT)";
  document.exitPointerLock();
  document.getElementById("result-title").textContent="比赛结束";
  document.getElementById("result-desc").textContent=`${w} 赢得比赛！ 最终比分 T ${Game.scoreT} : ${Game.scoreCT} CT`;
  const btn=document.getElementById("result-btn"); btn.textContent="返回主菜单";
  document.getElementById("result").classList.remove("hidden");
}

function startFreeze(){
  Game.roundNum++;
  Game.phase="freeze"; Game.freezeLeft=CFG.freezeTime; Game.roundTimeLeft=CFG.roundTime;
  Game.canBuy=true; Game.bombPlanted=false; Game.planting=false; Game.defusing=false;
  spawnRound();
  // bots buy
  for(const e of Game.entities){ if(e.isBot) botBuy(e); }
  // player auto buy suggestion left to player; open buy menu if freeze
  Game.player.canBuyNow=true;
  showRoundBanner(`回合 ${Game.roundNum} — 购买阶段`);
  document.getElementById("deadscreen").classList.add("hidden");
  updateHUD();
}
function startLive(){
  Game.phase="live"; Game.canBuy=false; if(Game.buyOpen) toggleBuy(false);
  showRoundBanner("开始！");
}

/* --------------------------- Bot economy ---------------------------------- */
function botBuy(e){
  ensureAmmo(e,e.weapons.secondary);
  const m=e.money;
  // simple logic: full buy if >=4000, force/eco otherwise
  const wantRifle = e.team==="T" ? "ak47" : "m4a4";
  if(m>=4500){
    buyWeapon(e, e.team==="T"?"ak47":"m4a4"); buyGear(e,"helmet");
    if(Math.random()<0.3) e.nades.he=1; if(Math.random()<0.4) e.nades.smoke=1; if(Math.random()<0.3) e.nades.flash=1;
    if(e.team==="CT"&&Math.random()<0.4) e.hasDefuser=true;
    if(m>=6500 && Math.random()<0.18){ buyWeapon(e,"awp"); }
  } else if(m>=2500){
    buyWeapon(e, Math.random()<0.5?(e.team==="T"?"galil":"famas"):"mp9"); buyGear(e,"kevlar");
  } else if(m>=1200){
    buyWeapon(e,"ump45"); if(m>=1850) buyGear(e,"kevlar");
  } else {
    // eco - pistol
    if(m>=500 && Math.random()<0.5) buyWeapon(e,"deagle"); else if(m>=300) buyWeapon(e,"p250");
  }
  // ensure ammo full
  if(e.weapons.primary) ensureAmmo(e,e.weapons.primary);
}
function buyWeapon(e,key){ const w=WEAPONS[key]; if(!w||e.money<w.price) return false;
  e.money-=w.price; if(w.cat==="pistol"){ e.weapons.secondary=key; } else { e.weapons.primary=key; }
  ensureAmmo(e,key); e.ammo[key].clip=w.mag; e.ammo[key].reserve=w.reserve;
  e.current=w.cat==="pistol"?"secondary":"primary"; return true; }
function buyGear(e,g){ if(g==="kevlar"){ if(e.money>=GEAR.kevlar.price){e.money-=GEAR.kevlar.price;e.armor=100;} }
  else if(g==="helmet"){ if(e.money>=GEAR.helmet.price){e.money-=GEAR.helmet.price;e.armor=100;e.helmet=true;} }
  else if(g==="defuser"){ if(e.team==="CT"&&e.money>=GEAR.defuser.price){e.money-=GEAR.defuser.price;e.hasDefuser=true;} } }
function buyNade(e,type){ const n=NADES[type]; if(e.money<n.price) return; if(e.nades[type]>=(type==="flash"?2:1)) return;
  e.money-=n.price; e.nades[type]++; }

/* --------------------------- Player controller ---------------------------- */
function updatePlayer(dt){
  const p=Game.player;
  if(!p.alive){ updateSpectate(dt); return; }
  // mouse look
  const sens=0.0022;
  p.yaw -= Input.mouseDX*sens*(p.rmbZoom?0.4:1);
  p.pitch -= Input.mouseDY*sens*(p.rmbZoom?0.4:1);
  p.pitch=Math.max(-1.5,Math.min(1.5,p.pitch));
  Input.mouseDX=0; Input.mouseDY=0;

  // movement
  const w=WEAPONS[currentWeaponKey(p)]||{};
  let speed=CFG.walkSpeed;
  p.crouch=Input.keys["ControlLeft"]||Input.keys["ControlRight"];
  const sneak=Input.keys["ShiftLeft"]||Input.keys["ShiftRight"];
  if(p.crouch) speed=CFG.crouchSpeed; else if(sneak) speed=CFG.sneakSpeed;
  else speed=CFG.walkSpeed;
  // AWP/scope slows
  if(p.rmbZoom) speed*=0.5;
  const fwd=V3(Math.sin(p.yaw)*-1,0,Math.cos(p.yaw)*-1);
  const right=V3(Math.cos(p.yaw),0,Math.sin(p.yaw));
  let move=V3();
  if(Input.keys["KeyW"]) move.add(fwd);
  if(Input.keys["KeyS"]) move.sub(fwd);
  if(Input.keys["KeyD"]) move.add(right);
  if(Input.keys["KeyA"]) move.sub(right);
  if(move.lengthSq()>0){ move.normalize().multiplyScalar(speed); }
  p.vel.x=move.x; p.vel.z=move.z;
  // jump
  const groundY=groundHeightAt(p.pos);
  if((Input.keys["Space"]) && p.onGround){ p.vel.y=CFG.jumpVel; p.onGround=false; }
  p.vel.y-=CFG.gravity*dt;
  // integrate
  p.pos.x+=p.vel.x*dt; p.pos.z+=p.vel.z*dt;
  resolveCollision(p.pos,CFG.radius);
  p.pos.y+=p.vel.y*dt;
  if(p.pos.y<=groundY){ p.pos.y=groundY; p.vel.y=0; p.onGround=true; } else p.onGround=false;

  // footsteps
  if(p.onGround && move.lengthSq()>1 && !sneak){ p._stepT=(p._stepT||0)+dt;
    const interval=p.crouch?0.55:0.42; if(p._stepT>interval){ p._stepT=0; Audio.footstep(null); } }

  // weapon switching
  if(Input.keys["Digit1"]){ if(p.weapons.primary){p.current="primary";p.rmbZoom=false;} }
  if(Input.keys["Digit2"]){ p.current="secondary";p.rmbZoom=false; }
  if(Input.keys["Digit3"]){ p.current="knife";p.rmbZoom=false; }
  if(Input.keys["Digit4"]&&p.nades.he>0){ p.current="he"; }
  if(Input.keys["Digit5"]&&p.nades.flash>0){ p.current="flash"; }
  if(Input.keys["Digit6"]&&p.nades.smoke>0){ p.current="smoke"; }
  if(Input.keys["Digit7"]&&p.nades.molly>0){ p.current="molly"; }

  // ADS
  const wk=currentWeaponKey(p);
  const ww=WEAPONS[wk]||{};
  if(Input.rmb && ww.zoom>0){ p.rmbZoom=true; Game.camera.fov=75*ww.zoom; }
  else { p.rmbZoom=false; if(Math.abs(Game.camera.fov-75)>0.1){ Game.camera.fov=75; } }
  Game.camera.updateProjectionMatrix();

  // reload
  if(Input.keys["KeyR"] && !p.reloading){ startReload(p); }
  if(p.reloading){ p.reloadTimeLeft-=dt; if(p.reloadTimeLeft<=0) finishReload(p); }

  // cooldowns / heat decay
  if(p.fireCd>0) p.fireCd-=dt;
  p.spreadHeat=Math.max(0,p.spreadHeat-dt*0.12);

  // firing
  if(p.current==="he"||p.current==="flash"||p.current==="smoke"||p.current==="molly"){
    if(Input.mouseDown && p.fireCd<=0){ const t=p.current; if(p.nades[t]>0){ throwNade(p,t); p.nades[t]--; p.fireCd=0.8;
      // switch away if empty
      if(p.nades[t]===0){ p.current=p.weapons.primary?"primary":"secondary"; } } }
  } else if(!p.reloading){
    const canFire = ww.auto ? Input.mouseDown : (Input.mouseDown && !p._triggerHeld);
    if(ww.melee){ if(Input.mouseDown && p.fireCd<=0){ meleeAttack(p); p.fireCd=ww.rate; } }
    else if(canFire && p.fireCd<=0){
      const am=p.ammo[wk];
      if(am && am.clip>0){ fireBullet(p); am.clip--; p.fireCd=ww.rate;
        if(am.clip===0 && am.reserve>0) startReload(p);
      } else if(am && am.clip===0){ Audio.beep(200); p.fireCd=0.3; }
    }
    p._triggerHeld=Input.mouseDown;
  }

  // bomb plant / defuse (E)
  handlePlantDefuse(dt);

  // update camera
  const eh=(p.crouch?CFG.crouchHeight:CFG.playerHeight)-0.2;
  Game.camera.position.set(p.pos.x, p.pos.y+eh, p.pos.z);
  Game.camera.rotation.set(p.pitch,p.yaw,0,"YXZ");
  // viewmodel bob & recoil recover
  vmKick*=Math.pow(0.001,dt);
  Game.viewmodel.position.z=vmKick*0.15;
  Game.viewmodel.userData.flash.material.opacity*=Math.pow(0.0001,dt);
}

function meleeAttack(p){
  const origin=eyePos(p); const dir=V3(0,0,-1).applyEuler(new THREEJS.Euler(p.pitch,p.yaw,0,"YXZ"));
  for(const e of Game.entities){ if(!e.alive||e.team===p.team) continue;
    if(e.pos.distanceTo(p.pos)<2.2){ const to=e.pos.clone().sub(p.pos).normalize();
      if(to.dot(dir)>0.4){ applyDamage(e,p,WEAPONS.knife,false); showHitmarker(false);} } }
}

function startReload(e){ const k=currentWeaponKey(e); const w=WEAPONS[k]; if(!w||w.melee) return;
  const am=e.ammo[k]; if(!am||am.clip>=w.mag||am.reserve<=0) return;
  e.reloading=true; e.reloadTimeLeft=w.reload; if(e===Game.player) Audio.reload(); }
function finishReload(e){ const k=currentWeaponKey(e); const w=WEAPONS[k]; const am=e.ammo[k];
  const need=w.mag-am.clip; const take=Math.min(need,am.reserve); am.clip+=take; am.reserve-=take; e.reloading=false; }

function handlePlantDefuse(dt){
  const p=Game.player; if(Game.phase!=="live") return;
  const eDown=Input.keys["KeyE"];
  if(p.team==="T" && !Game.bombPlanted && Game.bombCarrier===p){
    const inA=p.pos.distanceTo(Game.map.siteA.pos)<Game.map.siteA.r;
    const inB=p.pos.distanceTo(Game.map.siteB.pos)<Game.map.siteB.r;
    if((inA||inB) && eDown && p.onGround){
      Game.planting=true; Game.plantProg+=dt/CFG.plantTime;
      showProgress("planting-bar",Game.plantProg);
      if(Game.plantProg>=1){ plantBomb(p, inA?Game.map.siteA:Game.map.siteB); }
    } else { Game.planting=false; Game.plantProg=Math.max(0,Game.plantProg-dt); showProgress("planting-bar",0); }
  }
  if(p.team==="CT" && Game.bombPlanted){
    const near=p.pos.distanceTo(Game.bombPos)<2.2;
    if(near && eDown && p.onGround){
      Game.defusing=true; const dtime=p.hasDefuser?CFG.defuseTimeKit:CFG.defuseTime;
      Game.defuseProg+=dt/dtime; showProgress("defusing-bar",Game.defuseProg);
      if(Game.defuseProg>=1){ defuseBomb(p); }
    } else { Game.defusing=false; Game.defuseProg=Math.max(0,Game.defuseProg-dt*0.6); showProgress("defusing-bar",0); }
  }
}
function plantBomb(planter, site){
  Game.bombPlanted=true; Game.planting=false; Game.plantProg=0; showProgress("planting-bar",0);
  Game.bombPos=planter.pos.clone(); Game.bombPos.y=0.2; Game.bombTimer=CFG.bombTime;
  const m=new THREEJS.Mesh(new THREEJS.BoxGeometry(0.4,0.2,0.25),new THREEJS.MeshLambertMaterial({color:0x222,emissive:0x330000}));
  m.position.copy(Game.bombPos); Game.scene.add(m); Game.bombMesh=m;
  planter.money=Math.min(CFG.maxMoney,planter.money+CFG.reward.plant);
  showCenterMsg("炸弹已安放！"); Audio.beep(660);
}
function defuseBomb(ct){
  Game.bombPlanted=false; Game.defusing=false; Game.defuseProg=0; showProgress("defusing-bar",0);
  if(Game.bombMesh){ Game.scene.remove(Game.bombMesh); Game.bombMesh=null; }
  endRound("CT","defuse");
}
function explodeBomb(){
  if(Game.bombMesh){ Game.scene.remove(Game.bombMesh);} 
  explosionEffect(Game.bombPos,0xff5500,6); Audio.explosion(Game.bombPos.distanceTo(Game.player.pos));
  for(const e of Game.entities){ if(e.alive && e.pos.distanceTo(Game.bombPos)<12){ e.hp=0; killEntity(e,null,false);} }
  Game.bombPlanted=false;
  endRound("T","explode");
}

/* --------------------------- Spectate ------------------------------------- */
function onPlayerDeath(){ document.getElementById("deadscreen").classList.remove("hidden");
  Game.spectateIdx=0; }
function updateSpectate(dt){
  // follow a living teammate
  const mates=Game.entities.filter(e=>e.team===Game.player.team&&e.alive);
  if(mates.length){ const t=mates[Game.spectateIdx%mates.length];
    const eh=CFG.playerHeight-0.2;
    Game.camera.position.set(t.pos.x, t.pos.y+eh+1.5, t.pos.z);
    Game.camera.lookAt(t.pos.x+Math.sin(t.yaw)*-3, t.pos.y+1.4, t.pos.z+Math.cos(t.yaw)*-3);
  }
}

/* --------------------------- BOT AI --------------------------------------- */
function nearestWaypoint(pos){ const wp=Game.map.wp; let bi=0,bd=1e9;
  for(let i=0;i<wp.length;i++){ const d=wp[i].distanceToSquared(pos); if(d<bd){bd=d;bi=i;} } return bi; }
function buildPath(fromIdx,toIdx){
  const wp=Game.map.wp; const adj={}; for(let i=0;i<wp.length;i++) adj[i]=[];
  for(const [a,b] of Game.map.edges){ adj[a].push(b); adj[b].push(a); }
  const q=[fromIdx]; const prev={}; prev[fromIdx]=-1; 
  while(q.length){ const c=q.shift(); if(c===toIdx) break;
    for(const n of adj[c]){ if(prev[n]===undefined){ prev[n]=c; q.push(n);} } }
  if(prev[toIdx]===undefined) return [toIdx];
  const path=[]; let c=toIdx; while(c!==-1){ path.unshift(c); c=prev[c]; } return path;
}
function botGoal(e){
  // choose target waypoint index based on role
  if(e.team==="T"){
    if(Game.bombPlanted){ // defend planted bomb
      return nearestWaypoint(Game.bombPos);
    }
    // go to a site (carrier or supporter)
    const site = e._preferSite || (Math.random()<0.5?Game.map.siteA:Game.map.siteB);
    e._preferSite=site;
    return nearestWaypoint(site.pos);
  } else { // CT
    if(Game.bombPlanted){ return nearestWaypoint(Game.bombPos); }
    // defend or hold a site
    const site=e._preferSite || (Math.random()<0.5?Game.map.siteA:Game.map.siteB);
    e._preferSite=site;
    return nearestWaypoint(site.pos);
  }
}
function updateBot(e,dt){
  if(!e.alive) return;
  const D=DIFF[Game.diff];
  const ai=e.ai;
  // blind
  if(ai.blindTimer>0){ ai.blindTimer-=dt; }
  // cooldowns
  if(e.fireCd>0) e.fireCd-=dt;
  e.spreadHeat=Math.max(0,e.spreadHeat-dt*0.14);
  if(e.reloading){ e.reloadTimeLeft-=dt; if(e.reloadTimeLeft<=0) finishReload(e); }
  // gravity/ground
  const gY=groundHeightAt(e.pos); e.vel.y-=CFG.gravity*dt; e.pos.y+=e.vel.y*dt;
  if(e.pos.y<=gY){ e.pos.y=gY; e.vel.y=0; e.onGround=true; } else e.onGround=false;

  // acquire target: nearest visible enemy in FOV
  let target=null,bestD=1e9;
  const eye=eyePos(e);
  for(const o of Game.entities){ if(!o.alive||o.team===e.team) continue;
    const d=e.pos.distanceTo(o.pos); if(d>bestD) continue;
    const oe=eyePos(o); const dir=oe.clone().sub(eye); const dist=dir.length(); dir.normalize();
    const look=V3(0,0,-1).applyEuler(new THREEJS.Euler(e.pitch,e.yaw,0,"YXZ"));
    if(look.dot(dir) < Math.cos(D.fov/2) && d>4) continue; // outside FOV (unless very close)
    if(hasLOS(eye,oe)){ if(d<bestD){ bestD=d; target=o; } }
  }
  if(ai.blindTimer>1.2) target=null; // too blind

  if(target){
    ai.target=target; ai.lastSeen=target.pos.clone(); ai.seeTimer+=dt;
    if(ai.reactTimer<D.react){ ai.reactTimer+=dt; }
    ai.state="engage";
  } else {
    ai.seeTimer=0; ai.reactTimer=0;
    if(ai.state==="engage") ai.state = ai.lastSeen?"hunt":"rotate";
  }

  if(ai.state==="engage" && target && ai.reactTimer>=D.react){
    // aim toward target with error
    const oe=eyePos(target);
    // lead & aim
    const desired=oe.clone().sub(eye);
    const desiredYaw=Math.atan2(-desired.x,-desired.z);
    const desiredPitch=Math.atan2(desired.y, Math.sqrt(desired.x*desired.x+desired.z*desired.z));
    // add aim error
    const err=D.aimErr*(0.6+bestD/120);
    e.yaw=lerpAngle(e.yaw, desiredYaw+(Math.random()-0.5)*err, Math.min(1,D.aimSpeed*dt));
    e.pitch=lerpAngle(e.pitch, desiredPitch+(Math.random()-0.5)*err, Math.min(1,D.aimSpeed*dt));
    // strafe
    ai.strafeTimer-=dt; if(ai.strafeTimer<=0){ ai.strafeDir*=-1; ai.strafeTimer=0.4+Math.random()*0.6; }
    const right=V3(Math.cos(e.yaw),0,Math.sin(e.yaw)).multiplyScalar(ai.strafeDir* CFG.walkSpeed*0.6);
    e.vel.x=right.x; e.vel.z=right.z;
    // fire if aimed
    const aimDot = Math.abs(angleDiff(e.yaw,desiredYaw))<0.09 && Math.abs(angleDiff(e.pitch,desiredPitch))<0.09;
    const wk=currentWeaponKey(e); const w=WEAPONS[wk]; ensureAmmo(e,wk);
    const am=e.ammo[wk];
    if(w && !w.melee){
      if(am.clip<=0){ if(am.reserve>0) startReload(e); else { /* switch to pistol */ e.current="secondary"; } }
      else if(aimDot && e.fireCd<=0 && Math.random()<D.fireDisc){
        fireBullet(e); am.clip--; e.fireCd=w.rate*(1+ (1-D.fireDisc)*0.6);
      }
    }
    // maybe throw nade
    if(Math.random()<D.nadeChance*dt*0.3 && (e.nades.he>0||e.nades.flash>0) && bestD>10){
      const t=e.nades.he>0?"he":"flash"; e.nades[t]--; const save=e.current; e.current=t; throwNade(e,t); e.current=save;
    }
  } else {
    // navigate toward goal
    ai.repathTimer-=dt;
    if(!ai.path.length || ai.repathTimer<=0){ ai.goal=botGoal(e); ai.path=buildPath(nearestWaypoint(e.pos),ai.goal); ai.pathIdx=0; ai.repathTimer=2+Math.random()*2; }
    navigate(e,dt);
    // slowly look forward along velocity
    if(e.vel.lengthSq()>0.5){ const desiredYaw=Math.atan2(-e.vel.x,-e.vel.z); e.yaw=lerpAngle(e.yaw,desiredYaw,3*dt); e.pitch=lerpAngle(e.pitch,0,2*dt); }
    // if hunting last seen, look there
    if(ai.state==="hunt"&&ai.lastSeen){ if(e.pos.distanceTo(ai.lastSeen)<3){ ai.lastSeen=null; ai.state="rotate"; } }
  }

  // T bot plant logic
  if(e.team==="T" && Game.bombCarrier===e && !Game.bombPlanted && Game.phase==="live"){
    const site=[Game.map.siteA,Game.map.siteB].find(s=>e.pos.distanceTo(s.pos)<s.r);
    if(site && (!target)){ ai.plantTimer+=dt; e.vel.x=0;e.vel.z=0;
      if(ai.plantTimer>=CFG.plantTime){ plantBombByBot(e,site); } }
    else ai.plantTimer=Math.max(0,ai.plantTimer-dt);
  }
  // CT bot defuse logic
  if(e.team==="CT" && Game.bombPlanted && Game.phase==="live"){
    if(e.pos.distanceTo(Game.bombPos)<2 && !target){ e.vel.x=0;e.vel.z=0;
      ai._defuse=(ai._defuse||0)+dt; const dtime=e.hasDefuser?CFG.defuseTimeKit:CFG.defuseTime;
      if(ai._defuse>=dtime){ endRound("CT","defuse"); } }
    else ai._defuse=0;
  }

  // integrate horizontal
  e.pos.x+=e.vel.x*dt; e.pos.z+=e.vel.z*dt;
  resolveCollision(e.pos,CFG.radius);
  // footstep audio for player proximity
  if(e.onGround && e.vel.lengthSq()>1){ e._stepT=(e._stepT||0)+dt; if(e._stepT>0.45){ e._stepT=0; Audio.footstep(e.pos.distanceTo(Game.player.pos)); } }
  updateHitboxes(e);
}
function plantBombByBot(e,site){
  Game.bombPlanted=true; Game.bombPos=e.pos.clone(); Game.bombPos.y=0.2; Game.bombTimer=CFG.bombTime;
  const m=new THREEJS.Mesh(new THREEJS.BoxGeometry(0.4,0.2,0.25),new THREEJS.MeshLambertMaterial({color:0x222,emissive:0x330000}));
  m.position.copy(Game.bombPos); Game.scene.add(m); Game.bombMesh=m;
  e.money=Math.min(CFG.maxMoney,e.money+CFG.reward.plant);
  showCenterMsg("炸弹已安放！"); Audio.beep(660);
}
function navigate(e,dt){
  const wp=Game.map.wp;
  if(ai_pathDone(e)){ e.vel.x=0;e.vel.z=0; return; }
  const targetPt=wp[e.ai.path[e.ai.pathIdx]];
  const to=targetPt.clone().sub(e.pos); to.y=0; const d=to.length();
  if(d<2.2){ e.ai.pathIdx++; return; }
  to.normalize().multiplyScalar(CFG.walkSpeed*0.85);
  e.vel.x=to.x; e.vel.z=to.z;
}
function ai_pathDone(e){ return e.ai.pathIdx>=e.ai.path.length; }

function lerpAngle(a,b,t){ let d=angleDiff(a,b); return a - d*t; }
function angleDiff(a,b){ let d=(a-b)%(Math.PI*2); if(d>Math.PI)d-=Math.PI*2; if(d<-Math.PI)d+=Math.PI*2; return d; }

/* --------------------------- Effects update ------------------------------- */
function updateEffects(dt){
  for(let i=Game.effects.length-1;i>=0;i--){ const f=Game.effects[i]; f.t+=dt;
    if(f.type==="nade"){ nadePhysics(f,dt); if(f.t>=f.dur){ detonateNade(f); Game.effects.splice(i,1);} continue; }
    if(f.type==="spark"){ f.vel.y-=CFG.gravity*dt; f.obj.position.add(f.vel.clone().multiplyScalar(dt)); }
    if(f.type==="boom"){ f.obj.scale.multiplyScalar(1+dt*3); f.obj.material.opacity=0.8*(1-f.t/f.dur); }
    if(f.type==="muzzle"){ /* handled by opacity decay */ }
    if(f.t>=f.dur){ if(f.type!=="muzzle") Game.scene.remove(f.obj); Game.effects.splice(i,1); }
  }
  // smokes
  for(let i=Game.smokes.length