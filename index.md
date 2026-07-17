<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ball Tracking Robot — Preview</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
<style>
  :root{
    --ink:#0F1416;
    --paper:#FBFAF7;
    --panel:#0E1417;
    --panel-2:#141D22;
    --ball:#E63B2E;      /* the tracked ball */
    --box:#2FBF71;       /* the detection box */
    --muted:#6B7580;
    --line:#E4E0D8;
    --code-bg:#0E1417;
  }
  *{box-sizing:border-box;margin:0;padding:0}
  html{scroll-behavior:smooth}
  body{
    background:var(--paper);
    color:var(--ink);
    font-family:'Inter',system-ui,sans-serif;
    font-size:17px;
    line-height:1.7;
    -webkit-font-smoothing:antialiased;
  }

  /* ---------- HEADER / HUD BANNER ---------- */
  .page-header{
    position:relative;
    background:var(--panel);
    background-image:
      linear-gradient(rgba(47,191,113,.06) 1px,transparent 1px),
      linear-gradient(90deg,rgba(47,191,113,.06) 1px,transparent 1px);
    background-size:34px 34px;
    color:#EDEFEC;
    padding:78px 24px 84px;
    text-align:center;
    overflow:hidden;
    border-bottom:2px solid var(--box);
  }
  .hud-readout{
    font-family:'JetBrains Mono',monospace;
    font-size:12px;
    letter-spacing:.22em;
    text-transform:uppercase;
    color:var(--box);
    display:flex;
    gap:14px;
    align-items:center;
    justify-content:center;
    margin-bottom:26px;
  }
  .rec-dot{
    width:9px;height:9px;border-radius:50%;
    background:var(--ball);
    box-shadow:0 0 0 0 rgba(230,59,46,.6);
    animation:rec 1.8s ease-out infinite;
  }
  @keyframes rec{
    0%{box-shadow:0 0 0 0 rgba(230,59,46,.55)}
    70%{box-shadow:0 0 0 10px rgba(230,59,46,0)}
    100%{box-shadow:0 0 0 0 rgba(230,59,46,0)}
  }
  /* the signature: a machine-vision bounding box framing the title */
  .track-frame{
    position:relative;
    display:inline-block;
    padding:26px 40px;
    margin:0 auto;
  }
  .track-frame::before,
  .track-frame::after,
  .track-frame > .br-tl,
  .track-frame > .br-br{
    content:"";
    position:absolute;
    width:26px;height:26px;
    border:2px solid var(--box);
  }
  .track-frame::before{top:0;left:0;border-right:0;border-bottom:0}
  .track-frame::after{top:0;right:0;border-left:0;border-bottom:0}
  .track-frame > .br-tl{bottom:0;left:0;border-right:0;border-top:0}
  .track-frame > .br-br{bottom:0;right:0;border-left:0;border-top:0}
  .track-tag{
    position:absolute;
    top:-11px;left:50%;transform:translateX(-50%);
    background:var(--box);
    color:var(--panel);
    font-family:'JetBrains Mono',monospace;
    font-size:10px;font-weight:700;letter-spacing:.15em;
    padding:2px 8px;border-radius:2px;white-space:nowrap;
  }
  .project-name{
    font-family:'Space Grotesk',sans-serif;
    font-weight:700;
    font-size:clamp(30px,5vw,52px);
    line-height:1.05;
    letter-spacing:-.02em;
  }
  .project-name .accent{color:var(--ball)}
  .project-tagline{
    font-family:'JetBrains Mono',monospace;
    font-size:14px;
    color:#9AA6A0;
    margin-top:26px;
    letter-spacing:.02em;
  }
  .btns{margin-top:34px;display:flex;gap:14px;justify-content:center;flex-wrap:wrap}
  .btn{
    font-family:'JetBrains Mono',monospace;
    font-size:13px;font-weight:500;letter-spacing:.05em;
    text-decoration:none;
    color:#EDEFEC;
    border:1.5px solid rgba(255,255,255,.22);
    padding:11px 20px;border-radius:4px;
    transition:all .18s ease;
  }
  .btn:hover{border-color:var(--box);color:var(--box);transform:translateY(-1px)}
  .btn.primary{border-color:var(--box);color:var(--box)}
  .btn.primary:hover{background:var(--box);color:var(--panel)}

  /* ---------- CONTENT ---------- */
  .main-content{
    max-width:820px;
    margin:0 auto;
    padding:64px 24px 90px;
  }
  .intro{
    font-size:19px;
    color:#333B40;
    border-left:3px solid var(--ball);
    padding-left:20px;
    margin-bottom:16px;
  }

  h1.section{
    font-family:'Space Grotesk',sans-serif;
    font-size:clamp(26px,3.4vw,34px);
    font-weight:700;
    letter-spacing:-.02em;
    margin:70px 0 22px;
    padding-bottom:14px;
    border-bottom:1px solid var(--line);
    position:relative;
  }
  h1.section .idx{
    font-family:'JetBrains Mono',monospace;
    font-size:13px;font-weight:700;
    color:var(--box);
    letter-spacing:.1em;
    display:block;
    margin-bottom:6px;
  }
  .label{
    font-family:'Space Grotesk',sans-serif;
    font-weight:600;
    font-size:20px;
    margin:30px 0 8px;
    display:flex;align-items:center;gap:10px;
  }
  .label::before{
    content:"";
    width:8px;height:8px;
    background:var(--box);
    border-radius:1px;
    transform:rotate(45deg);
  }
  p{margin:14px 0;color:#2A3237}
  code.inl{
    font-family:'JetBrains Mono',monospace;
    font-size:.86em;
    background:#EDEAE2;
    color:#B4402F;
    padding:1px 6px;border-radius:4px;
  }

  /* code block styled like the robot's terminal */
  .code-wrap{
    background:var(--code-bg);
    border-radius:10px;
    margin:22px 0;
    overflow:hidden;
    box-shadow:0 14px 34px -18px rgba(15,20,22,.55);
    border:1px solid #1E2A30;
  }
  .code-bar{
    display:flex;align-items:center;gap:8px;
    padding:11px 16px;
    background:#0A0F11;
    border-bottom:1px solid #1E2A30;
  }
  .code-bar .dot{width:11px;height:11px;border-radius:50%}
  .d1{background:#E63B2E}.d2{background:#E8B23A}.d3{background:#2FBF71}
  .code-bar .fname{
    font-family:'JetBrains Mono',monospace;
    font-size:12px;color:#7C8A90;margin-left:8px;
  }
  pre{
    margin:0;padding:20px 22px;overflow-x:auto;
    font-family:'JetBrains Mono',monospace;
    font-size:13.5px;line-height:1.6;color:#D7DEE1;
  }
  pre .cmt{color:#5C6E63;font-style:italic}
  pre .kw{color:#E28C6B}
  pre .fn{color:#7FD1A6}
  pre .num{color:#E8B23A}

  table{
    width:100%;border-collapse:collapse;margin:24px 0;
    font-size:15px;
  }
  th{
    background:var(--panel);color:#EDEFEC;
    font-family:'JetBrains Mono',monospace;
    font-size:12px;letter-spacing:.06em;text-transform:uppercase;
    text-align:left;padding:12px 14px;
  }
  td{padding:12px 14px;border-bottom:1px solid var(--line)}
  tr:last-child td{border-bottom:none}
  tbody tr:hover{background:#F3F0E9}

  .note{
    margin-top:50px;padding:18px 20px;
    background:#F3F0E9;border-radius:8px;
    font-size:14px;color:#5A636A;
    font-family:'JetBrains Mono',monospace;
    line-height:1.6;
  }
  .note b{color:var(--ink)}
</style>
</head>
<body>

<header class="page-header">
  <div class="hud-readout"><span class="rec-dot"></span> REC · TRACKING · CV_ONLINE</div>
  <div class="track-frame">
    <span class="track-tag">BALL_DETECTED ● conf 0.98</span>
    <span class="br-tl"></span><span class="br-br"></span>
    <h1 class="project-name">Ball Tracking Robot<br>with <span class="accent">OpenCV</span></h1>
  </div>
  <div class="project-tagline">Vaideesh K · Computer Vision · Cupertino High School</div>
  <div class="btns">
    <a class="btn primary" href="#">View on GitHub</a>
    <a class="btn" href="#">Jump to Final Code</a>
  </div>
</header>

<main class="main-content">

  <p class="intro">The Ball Tracking Robot with OpenCV uses a Raspberry Pi 4, a 5MP camera, and Python to create a robot that avoids obstacles, moves independently, and decides where to navigate.</p>

  <table>
    <thead><tr><th>Engineer</th><th>School</th><th>Area of Interest</th><th>Grade</th></tr></thead>
    <tbody><tr><td>Vaideesh K</td><td>Cupertino High School</td><td>Electrical Engineering</td><td>Incoming Senior</td></tr></tbody>
  </table>

  <h1 class="section"><span class="idx">MILESTONE 03 / FINAL</span>Final Milestone</h1>

  <div class="label">Summary</div>
  <p>I installed a 5MP Raspberry Pi Camera and used OpenCV to run the vision code. The finished robot combines three ultrasonic sensors, the L9110 motor driver, and two motors so it can detect and follow a red ball while measuring distance to objects on the left, center, and right. To make sure it only follows the ball, I added a <code class="inl">circularity</code> check that measures how round each red object is.</p>

  <div class="label">Ball Detection Code</div>
  <p>It converts each frame to HSV, filters for red, cleans the mask so glare doesn't break the shape, then scores each object by roundness. Only round objects count as the ball.</p>

  <div class="code-wrap">
    <div class="code-bar">
      <span class="dot d1"></span><span class="dot d2"></span><span class="dot d3"></span>
      <span class="fname">ball_detection.py</span>
    </div>
<pre><span class="cmt"># Roundness threshold — raise it if it tracks non-ball red objects</span>
MIN_CIRCULARITY = <span class="num">0.4</span>

<span class="kw">for</span> c <span class="kw">in</span> contours:
    area = cv2.<span class="fn">contourArea</span>(c)
    <span class="kw">if</span> area &lt; <span class="num">300</span>:
        <span class="kw">continue</span>
    perimeter = cv2.<span class="fn">arcLength</span>(c, <span class="kw">True</span>)
    circularity = <span class="num">4</span> * np.pi * area / (perimeter * perimeter)</pre>
  </div>

  <h1 class="section"><span class="idx">EXTENSION / POST-FINAL</span>Modifications</h1>

  <div class="label">Summary</div>
  <p>After the final milestone I upgraded the robot with three improvements: variable <code class="inl">PWM</code> speed control, a servo that pans the camera to keep the ball centered, and a Flask web dashboard that streams the camera feed live and lets me start and stop the robot from my phone.</p>

  <div class="label">Servo Camera Panning</div>
  <p>The biggest change was adding a servo under the camera so it can turn on its own. The code measures how far the ball is from the center of the frame, and if that error passes a deadzone, it nudges the servo to re-center the ball while the body stays pointed forward.</p>

  <div class="note">
    <b>// this is a preview.</b> Your real site keeps all your markdown exactly as it is — this styling gets layered on top of the GitHub Pages theme. Colors, fonts, code blocks, header, and tables all upgrade automatically. The red ball + green bounding-box motif is pulled straight from your robot's own camera view.
  </div>

</main>
</body>
</html>
