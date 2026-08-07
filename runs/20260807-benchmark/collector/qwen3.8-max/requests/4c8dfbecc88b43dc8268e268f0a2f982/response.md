# 长颈鹿开小车 · 动画 SVG

一个纯 SVG 产物（CSS 动画 + SMIL，无 JS、无外部依赖），保存后双击即可运行。包含：车轮滚动、道路/树木/远山视差滚动、云朵漂移、车身颠簸、长颈鹿脖子摇摆、围巾飘动、眨眼、尾气、速度线、太阳光晕旋转、小鸟飞过。

## 产物：`giraffe-car.svg`

```xml
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
     viewBox="0 0 900 500" width="900" height="500" role="img" aria-labelledby="t d">
  <title id="t">长颈鹿开小车 · Giraffe driving a tiny car</title>
  <desc id="d">A giraffe driving a tiny red convertible; wheels spin, scenery scrolls by.</desc>

  <style>
    .rays  { animation: spin 26s linear infinite; transform-box: fill-box; transform-origin: center; }
    .cloud { animation: drift linear infinite; }
    .c1 { animation-duration: 50s; animation-delay: -18s; }
    .c2 { animation-duration: 66s; animation-delay: -44s; }
    .c3 { animation-duration: 57s; animation-delay: -30s; }
    .birds    { animation: flyby 34s linear infinite; animation-delay: -12s; }
    .hillsFar { animation: hillsFarMove 26s linear infinite; }
    .hills    { animation: hillsMove 16s linear infinite; }
    .trees    { animation: treesMove 4.5s linear infinite; }
    .lane     { animation: laneMove .3s linear infinite; }
    .ride     { animation: bob .55s ease-in-out infinite alternate; }
    .shadow   { animation: squash .55s ease-in-out infinite alternate; transform-box: fill-box; transform-origin: center; }
    .wheel    { animation: spin .6s linear infinite; transform-box: fill-box; transform-origin: center; }
    .puff     { animation: puff 1.35s ease-out infinite; transform-box: fill-box; transform-origin: center; opacity: 0; }
    .p2 { animation-delay: .45s; }  .p3 { animation-delay: .9s; }
    .speedline { animation: zoom .22s linear infinite; }
    .s2 { animation-delay: -.08s; } .s3 { animation-delay: -.15s; }

    @keyframes spin        { to { transform: rotate(360deg); } }
    @keyframes drift       { from { transform: translateX(960px); } to { transform: translateX(-300px); } }
    @keyframes flyby       { from { transform: translateX(960px); } to { transform: translateX(-180px); } }
    @keyframes hillsFarMove{ to { transform: translateX(-900px); } }
    @keyframes hillsMove   { to { transform: translateX(-900px); } }
    @keyframes treesMove   { to { transform: translateX(-1200px); } }
    @keyframes laneMove    { to { stroke-dashoffset: 80; } }
    @keyframes zoom        { to { stroke-dashoffset: -50; } }
    @keyframes bob         { to { transform: translateY(-3.5px); } }
    @keyframes squash      { to { transform: scaleX(.945); } }
    @keyframes puff {
      0%   { transform: translate(0,0) scale(.5); opacity: 0; }
      10%  { opacity: .75; }
      100% { transform: translate(-95px,-46px) scale(1.9); opacity: 0; }
    }
    @media (prefers-reduced-motion: reduce) { * { animation: none !important; } }
  </style>

  <defs>
    <linearGradient id="sky" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0" stop-color="#A9E0FF"/><stop offset="1" stop-color="#EAF9FF"/>
    </linearGradient>
    <g id="cloudL" fill="#FFFFFF">
      <ellipse cx="0" cy="0" rx="36" ry="21"/><ellipse cx="30" cy="7" rx="27" ry="15"/>
      <ellipse cx="-30" cy="8" rx="25" ry="13"/><ellipse cx="3" cy="-13" rx="26" ry="16"/>
    </g>
    <g id="cloudS" fill="#FFFFFF">
      <ellipse cx="0" cy="0" rx="26" ry="15"/><ellipse cx="21" cy="5" rx="19" ry="11"/>
      <ellipse cx="-21" cy="6" rx="18" ry="10"/><ellipse cx="2" cy="-9" rx="19" ry="12"/>
    </g>
    <g id="tree">
      <rect x="-5" y="-44" width="10" height="46" rx="3" fill="#8B5E34"/>
      <circle cx="0" cy="-62" r="26" fill="#4E9F3D"/>
      <circle cx="-19" cy="-48" r="17" fill="#5FB04A"/>
      <circle cx="18" cy="-50" r="18" fill="#3E8433"/>
    </g>
  </defs>

  <!-- 天空与太阳 -->
  <rect width="900" height="500" fill="url(#sky)"/>
  <g transform="translate(768,86)">
    <g class="rays" stroke="#FFCE3C" stroke-width="5" stroke-linecap="round">
      <line x1="0" y1="-40" x2="0" y2="-54"/>
      <line x1="0" y1="-40" x2="0" y2="-54" transform="rotate(45)"/>
      <line x1="0" y1="-40" x2="0" y2="-54" transform="rotate(90)"/>
      <line x1="0" y1="-40" x2="0" y2="-54" transform="rotate(135)"/>
      <line x1="0" y1="-40" x2="0" y2="-54" transform="rotate(180)"/>
      <line x1="0" y1="-40" x2="0" y2="-54" transform="rotate(225)"/>
      <line x1="0" y1="-40" x2="0" y2="-54" transform="rotate(270)"/>
      <line x1="0" y1="-40" x2="0" y2="-54" transform="rotate(315)"/>
    </g>
    <circle r="31" fill="#FFD93B" stroke="#FFCE3C" stroke-width="4"/>
  </g>

  <!-- 云 -->
  <g transform="translate(0,70)"><use href="#cloudL" xlink:href="#cloudL" class="cloud c1" opacity=".95"/></g>
  <g transform="translate(0,132)"><use href="#cloudS" xlink:href="#cloudS" class="cloud c2" opacity=".9"/></g>
  <g transform="translate(0,42)"><use href="#cloudS" xlink:href="#cloudS" class="cloud c3" opacity=".85"/></g>

  <!-- 小鸟 -->
  <g transform="translate(0,108)">
    <g class="birds" fill="none" stroke="#4A5568" stroke-width="3" stroke-linecap="round">
      <path d="M0 0 Q7 -7 14 0 Q21 -7 28 0">
        <animate attributeName="d" dur=".7s" repeatCount="indefinite"
          values="M0 0 Q7 -7 14 0 Q21 -7 28 0;M0 0 Q7 4 14 0 Q21 4 28 0;M0 0 Q7 -7 14 0 Q21 -7 28 0"/>
      </path>
      <g transform="translate(38,18) scale(.75)">
        <path d="M0 0 Q7 -7 14 0 Q21 -7 28 0">
          <animate attributeName="d" dur=".7s" begin="-.35s" repeatCount="indefinite"
            values="M0 0 Q7 -7 14 0 Q21 -7 28 0;M0 0 Q7 4 14 0 Q21 4 28 0;M0 0 Q7 -7 14 0 Q21 -7 28 0"/>
        </path>
      </g>
    </g>
  </g>

  <!-- 远山 / 近山（视差） -->
  <g class="hillsFar">
    <path id="hf" d="M0 452 L0 424 Q150 386 300 420 Q450 448 600 418 Q750 392 900 424 L900 452 Z" fill="#D8F0C4"/>
    <use href="#hf" xlink:href="#hf" x="900"/>
  </g>
  <g class="hills">
    <path id="hn" d="M0 452 L0 410 Q110 350 230 402 Q330 445 430 412 Q540 378 650 418 Q760 452 900 410 L900 452 Z" fill="#B4E09A"/>
    <use href="#hn" xlink:href="#hn" x="900"/>
  </g>

  <!-- 行道树（滚动，周期 1200px） -->
  <g class="trees">
    <use href="#tree" xlink:href="#tree" transform="translate(90,452)"/>
    <use href="#tree" xlink:href="#tree" transform="translate(370,452) scale(1.18)"/>
    <use href="#tree" xlink:href="#tree" transform="translate(640,452) scale(.88)"/>
    <use href="#tree" xlink:href="#tree" transform="translate(930,452) scale(1.05)"/>
    <use href="#tree" xlink:href="#tree" transform="translate(1290,452)"/>
    <use href="#tree" xlink:href="#tree" transform="translate(1570,452) scale(1.18)"/>
    <use href="#tree" xlink:href="#tree" transform="translate(1840,452) scale(.88)"/>
    <use href="#tree" xlink:href="#tree" transform="translate(2130,452) scale(1.05)"/>
  </g>

  <!-- 草地与马路 -->
  <rect x="0" y="446" width="900" height="8" fill="#8FCB7A"/>
  <rect x="0" y="452" width="900" height="48" fill="#59616D"/>
  <rect x="0" y="452" width="900" height="3" fill="#454C57"/>
  <line class="lane" x1="0" y1="480" x2="900" y2="480" stroke="#F7E9A5" stroke-width="6" stroke-dasharray="46 34"/>

  <!-- 车影 -->
  <ellipse class="shadow" cx="486" cy="457" rx="150" ry="8" fill="#1F2933" opacity=".18"/>

  <!-- 速度线 -->
  <g stroke="#FFFFFF" stroke-width="5" stroke-linecap="round" stroke-dasharray="24 26" opacity=".75">
    <line class="speedline s1" x1="350" y1="386" x2="228" y2="386"/>
    <line class="speedline s2" x1="354" y1="403" x2="206" y2="403"/>
    <line class="speedline s3" x1="348" y1="420" x2="246" y2="420"/>
  </g>

  <!-- 尾气 -->
  <g fill="#C7CED8">
    <circle class="puff p1" cx="340" cy="414" r="8"/>
    <circle class="puff p2" cx="340" cy="414" r="8"/>
    <circle class="puff p3" cx="340" cy="414" r="8"/>
  </g>

  <!-- ============ 车 + 长颈鹿（整体颠簸） ============ -->
  <g class="ride">
    <!-- 身体（下半截会被车门挡住） -->
    <path d="M466 390 L466 356 Q466 342 482 342 L538 342 Q556 342 556 358 L556 390 Z" fill="#F7B733"/>
    <circle cx="486" cy="362" r="7" fill="#B0722B"/>
    <circle cx="512" cy="368" r="6" fill="#B0722B"/>
    <circle cx="540" cy="357" r="5.5" fill="#B0722B"/>

    <!-- 脖子 + 头（SMIL 绕根部轻摆） -->
    <g>
      <animateTransform attributeName="transform" type="rotate"
        values="0 512 358;2.8 512 358;0 512 358;-2.2 512 358;0 512 358"
        keyTimes="0;0.28;0.5;0.78;1" dur="3.4s" repeatCount="indefinite"
        calcMode="spline" keySplines=".45 0 .55 1;.45 0 .55 1;.45 0 .55 1;.45 0 .55 1"/>
      <path d="M495 352 C503 298 523 238 555 186" stroke="#A66A1F" stroke-width="9"
            stroke-dasharray="9 5" fill="none" stroke-linecap="round"/>
      <path d="M512 356 C520 300 540 240 572 190" stroke="#F7B733" stroke-width="30"
            fill="none" stroke-linecap="round"/>
      <circle cx="522" cy="316" r="6" fill="#B0722B"/>
      <circle cx="532" cy="282" r="6.5" fill="#B0722B"/>
      <circle cx="544" cy="248" r="6" fill="#B0722B"/>
      <circle cx="556" cy="216" r="5.5" fill="#B0722B"/>
      <ellipse cx="560" cy="160" rx="12" ry="6" fill="#F7B733" transform="rotate(-35 560 160)"/>
      <ellipse cx="560" cy="160" rx="7" ry="3" fill="#E39B2D" transform="rotate(-35 560 160)"/>
      <path d="M574 156 L568 134" stroke="#C98A2B" stroke-width="5" stroke-linecap="round"/>
      <path d="M590 154 L590 132" stroke="#C98A2B" stroke-width="5" stroke-linecap="round"/>
      <circle cx="568" cy="132" r="4.5" fill="#8C5A16"/>
      <circle cx="590" cy="130" r="4.5" fill="#8C5A16"/>
      <ellipse cx="584" cy="174" rx="26" ry="20" fill="#F7B733" transform="rotate(-18 584 174)"/>
      <circle cx="568" cy="184" r="3" fill="#B0722B"/>
      <ellipse cx="612" cy="182" rx="17" ry="13" fill="#F9D28C" transform="rotate(-10 612 182)"/>
      <circle cx="620" cy="178" r="2.3" fill="#8C5A16"/>
      <path d="M605 191 Q613 197 621 191" stroke="#8C5A16" stroke-width="2.5" fill="none" stroke-linecap="round"/>
      <circle cx="592" cy="185" r="4.5" fill="#F08A7E" opacity=".5"/>
      <ellipse cx="580" cy="166" rx="4.5" ry="6" fill="#2E3440">
        <animate attributeName="ry" values="6;6;.5;6;6" keyTimes="0;.88;.92;.96;1" dur="4.2s" repeatCount="indefinite"/>
      </ellipse>
      <circle cx="581.6" cy="163.6" r="1.4" fill="#FFFFFF">
        <animate attributeName="opacity" values="1;1;0;1;1" keyTimes="0;.88;.92;.96;1" dur="4.2s" repeatCount="indefinite"/>
      </circle>
    </g>

    <!-- 围巾（SMIL 形变飘动） -->
    <path fill="#2AB7CA" d="M508 326 C486 318 466 330 444 322 L447 338 C468 344 488 336 510 340 Z">
      <animate attributeName="d" dur=".55s" repeatCount="indefinite"
        values="M508 326 C486 318 466 330 444 322 L447 338 C468 344 488 336 510 340 Z;
                M508 326 C486 334 464 316 442 330 L446 346 C470 338 488 348 510 340 Z;
                M508 326 C486 318 466 330 444 322 L447 338 C468 344 488 336 510 340 Z"/>
    </path>
    <circle cx="509" cy="333" r="6.5" fill="#1F93A5"/>

    <!-- 车身（敞篷小车） -->
    <path d="M362 424 L362 402 Q362 384 382 380 L438 374 Q447 373 448 364 L451 346
             Q452 338 460 338 Q468 338 469 346 L471 368 Q472 374 479 374 L550 374
             Q560 374 566 376 L596 380 Q612 384 612 402 L612 424 Q612 428 606 428
             L368 428 Q362 428 362 424 Z" fill="#E64545"/>
    <path d="M362 416 L612 416 L612 424 Q612 428 606 428 L368 428 Q362 428 362 424 Z" fill="#C93A3A" opacity=".6"/>
    <path d="M480 376 Q477 400 481 426" stroke="#C23A3A" stroke-width="2.5" fill="none"/>
    <rect x="523" y="387" width="13" height="4.5" rx="2" fill="#C23A3A"/>
    <circle cx="505" cy="401" r="13" fill="#FFF7EE" stroke="#C23A3A" stroke-width="2"/>
    <text x="505" y="406.5" font-family="Arial, Helvetica, sans-serif" font-size="16"
          font-weight="bold" fill="#C23A3A" text-anchor="middle">7</text>
    <circle cx="600" cy="393" r="7" fill="#FFF3C4" stroke="#D9A404" stroke-width="2.5"/>
    <rect x="359" y="393" width="6" height="11" rx="2" fill="#FF8E8E"/>
    <rect x="596" y="417" width="18" height="8" rx="4" fill="#B7BFC9"/>
    <rect x="354" y="417" width="16" height="8" rx="4" fill="#B7BFC9"/>
    <rect x="344" y="409" width="16" height="7" rx="3" fill="#8A93A3"/>

    <!-- 挡风玻璃 -->
    <path d="M560 373 L575 331 L581 333 L568 373 Z" fill="#BDE9FF" opacity=".85"
          stroke="#7FC4E8" stroke-width="2" stroke-linejoin="round"/>

    <!-- 方向盘与腿 -->
    <line x1="554" y1="374" x2="549" y2="350" stroke="#3A4149" stroke-width="5" stroke-linecap="round"/>
    <ellipse cx="548" cy="346" rx="5" ry="14" fill="none" stroke="#2E3440" stroke-width="5"
             transform="rotate(-15 548 346)"/>
    <path d="M534 368 Q550 366 552 340" stroke="#F7B733" stroke-width="11" fill="none" stroke-linecap="round"/>
    <circle cx="552" cy="337" r="6" fill="#8C5A16"/>
  </g>

  <!-- 车轮（不随车身颠簸，贴地旋转） -->
  <g transform="translate(414,430)">
    <g class="wheel">
      <circle r="26" fill="#2E3440"/>
      <circle r="13.5" fill="#E8EDF2"/>
      <g stroke="#98A2B0" stroke-width="4" stroke-linecap="round">
        <line x1="-11" y1="0" x2="11" y2="0"/><line x1="0" y1="-11" x2="0" y2="11"/>
      </g>
      <circle r="4" fill="#59616D"/>
    </g>
  </g>
  <g transform="translate(558,430)">
    <g class="wheel">
      <circle r="26" fill="#2E3440"/>
      <circle r="13.5" fill="#E8EDF2"/>
      <g stroke="#98A2B0" stroke-width="4" stroke-linecap="round">
        <line x1="-11" y1="0" x2="11" y2="0"/><line x1="0" y1="-11" x2="0" y2="11"/>
      </g>
      <circle r="4" fill="#59616D"/>
    </g>
  </g>
</svg>
```

## 运行方式

1. **直接打开**：将上面代码保存为 `giraffe-car.svg`，双击用任意现代浏览器打开即自动播放。
2. **HTML 嵌入（img 方式）**：

```html
<!doctype html>
<html><body style="margin:0;background:#222;display:grid;place-items:center;min-height:100vh">
  <img src="giraffe-car.svg" width="900" height="500" alt="长颈鹿开小车">
</body></html>
```

3. **内联方式**：直接把整段 `<svg>…</svg>` 粘进 HTML 页面 body 内亦可。

## 测试说明

**测试环境**：最新版 Chrome / Edge / Firefox / Safari（桌面或移动端均可）；不支持 IE。

**功能检查清单**：

| # | 检查点 | 预期结果 |
|---|--------|----------|
| 1 | 车轮 | 两轮持续顺时针旋转（辐条可见），约 0.6s/圈 |
| 2 | 路面虚线、行道树 | 向左匀速移动，速度一致（约 267px/s），循环无跳变 |
| 3 | 远山/近山/云 | 三层视差：山慢、树快、云最慢，循环无缝 |
| 4 | 车身 | 轻微上下颠簸，车影同步缩放；车轮不随之离地 |
| 5 | 长颈鹿 | 脖子+头每 3.4s 缓慢左右摇摆，斑点随脖子一起动 |
| 6 | 围巾 | 青色围巾持续波浪形飘动 |
| 7 | 眼睛 | 每约 4.2s 眨眼一次 |
| 8 | 尾气/速度线 | 烟团向左上飘散淡出（3 团交替）；车后白色速度线向后掠 |
| 9 | 嵌入测试 | 通过 `<img>` 引用时动画同样播放 |
| 10 | 控制台 | 无报错；W3C 校验器无结构性错误 |

**调参提示**（三处速度需联动修改以保持"轮胎不打滑"）：车道虚线 `laneMove .3s`、树木 `treesMove 4.5s`（周期 1200px）、车轮 `spin .6s`。若系统开启"减少动态效果"，CSS 动画会自动停止（SMIL 部分仍会轻微摆动，属预期）。