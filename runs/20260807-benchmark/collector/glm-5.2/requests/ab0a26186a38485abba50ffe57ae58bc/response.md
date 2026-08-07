我来制作一个高级感的隐形眼镜盒 3D 模型。设计采用墨绿配金色的配色（避开蓝紫色），盒盖可点击开合，盒身和盒盖双面都有醒目的 L/R 金色标识，内部还有护理液、隐形眼镜等真实细节。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LUMINA · 隐形眼镜盒 3D 模型</title>

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@500;700&family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@300;400;500&display=swap" rel="stylesheet">

<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }

  :root {
    --bg-dark: #050a0c;
    --bg-mid: #0a1f1c;
    --bg-light: #143633;
    --accent: #f5a623;
    --accent-bright: #ffd966;
    --accent-2: #4ecdc4;
    --text: #e8f4f0;
    --text-dim: #6a8c87;
    --text-mute: #4a6c67;
    --border: rgba(245, 166, 35, 0.3);
  }

  body {
    background: radial-gradient(ellipse at 50% 30%, var(--bg-light) 0%, var(--bg-mid) 40%, var(--bg-dark) 80%);
    color: var(--text);
    font-family: 'Space Grotesk', system-ui, sans-serif;
    overflow: hidden;
    height: 100vh;
    position: relative;
  }

  body::after {
    content: '';
    position: fixed;
    inset: 0;
    background:
      radial-gradient(circle at 30% 70%, rgba(78, 205, 196, 0.08) 0%, transparent 50%),
      radial-gradient(circle at 70% 30%, rgba(245, 166, 35, 0.06) 0%, transparent 50%);
    pointer-events: none;
    z-index: 1;
  }

  #canvas-container {
    position: fixed;
    inset: 0;
    z-index: 2;
  }

  .header {
    position: fixed;
    top: 0; left: 0; right: 0;
    padding: 32px 