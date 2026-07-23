<!-- <!DOCTYPE html> -->
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ball Tracking Robot with OpenCV | Vaideesh K</title>
<meta name="description" content="A Raspberry Pi robot that finds a red ball with OpenCV, pans a servo camera to keep it centred, and drives to it while avoiding obstacles. Built by Vaideesh K.">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Bricolage+Grotesque:opsz,wght@12..96,400;12..96,600;12..96,700;12..96,800&family=Inter:wght@400;450;500;600&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
<style>
:root{
  /* instrument housing — everything is built from the machine's own world */
  --void:#05090B;
  --surface:#0A1014;
  --raised:#101A1F;
  --edge:#1B272E;
  --edge-2:#243138;

  --bright:#F2F7F5;
  --text:#B9C7CC;
  --dim:#7A8B92;
  --faint:#546268;

  /* daylight — the documentation half of the page */
  --page:#FBFAF7;
  --card:#FFFFFF;
  --rule:#E6E2D8;
  --rule-2:#D6D1C4;
  --ink:#0D1417;
  --read-text:#39454C;
  --muted:#6F7D85;
  --green-ink:#0A8A4C;

  --ball:#FF3B2F;      /* the tracked object */
  --box:#29E07E;       /* the detection overlay */
  --amber:#F5B93B;     /* uncertain / in motion */

  --display:'Bricolage Grotesque',system-ui,sans-serif;
  --sans:'Inter',system-ui,-apple-system,sans-serif;
  --mono:'JetBrains Mono',ui-monospace,monospace;

  --shell:1180px;
  --read:790px;
}
*{box-sizing:border-box;margin:0;padding:0}
html{scroll-behavior:smooth;scroll-padding-top:74px;background:var(--page)}
body{background:var(--page);color:var(--read-text);font-family:var(--sans);
  font-size:17px;line-height:1.75;font-weight:400;
  -webkit-font-smoothing:antialiased;text-rendering:optimizeLegibility;
  overflow-x:hidden}
::selection{background:#BBF3D6;color:var(--ink)}

/* ================================================================
   BOOT — the machine powers up before the page exists
   ================================================================ */
#boot{position:fixed;inset:0;z-index:900;background:var(--page);
  display:flex;align-items:center;justify-content:center;
  transition:opacity .55s ease,visibility .55s ease}
#boot.gone{opacity:0;visibility:hidden;pointer-events:none}
.boot-in{font-family:var(--mono);font-size:12.5px;line-height:2.1;
  color:var(--green-ink);width:min(90vw,340px)}
.boot-in .l{opacity:0;transform:translateY(4px);
  transition:opacity .3s ease,transform .3s ease;display:flex;gap:10px}
.boot-in .l.up{opacity:1;transform:none}
.boot-in .l i{color:var(--muted);font-style:normal;flex:none;width:52px}
.boot-in .l b{color:var(--green-ink);font-weight:500;margin-left:auto}
.boot-rail{height:2px;background:var(--rule-2);margin-top:22px;overflow:hidden;
  border-radius:2px}
.boot-rail span{display:block;height:100%;width:0;background:var(--green-ink);
  transition:width 1.25s cubic-bezier(.5,0,.2,1)}

/* ================================================================
   RETICLE CURSOR — desktop only, pointer devices only
   ================================================================ */
#retic{position:fixed;top:0;left:0;width:26px;height:26px;pointer-events:none;
  z-index:800;opacity:0;transition:opacity .25s ease,width .2s ease,height .2s ease}
#retic i{position:absolute;width:8px;height:8px;border:1.5px solid var(--green-ink);
  transition:all .18s cubic-bezier(.4,0,.2,1)}
#retic i:nth-child(1){top:0;left:0;border-right:0;border-bottom:0}
#retic i:nth-child(2){top:0;right:0;border-left:0;border-bottom:0}
#retic i:nth-child(3){bottom:0;left:0;border-right:0;border-top:0}
#retic i:nth-child(4){bottom:0;right:0;border-left:0;border-top:0}
#retic.live{opacity:1}
#retic.lock{width:44px;height:44px}
#retic.lock i{border-color:var(--ball);width:11px;height:11px}
@media (hover:none),(pointer:coarse){#retic{display:none}}

/* ================================================================
   PROGRESS + STICKY NAV
   ================================================================ */
.progress{position:fixed;top:0;left:0;height:2.5px;width:0;
  background:var(--green-ink);z-index:200;transition:width .1s linear;
  box-shadow:0 0 10px rgba(10,138,76,.5)}
.stick{position:fixed;top:0;left:0;right:0;z-index:150;
  background:rgba(251,250,247,.86);backdrop-filter:blur(20px) saturate(160%);
  -webkit-backdrop-filter:blur(20px) saturate(160%);
  border-bottom:1px solid var(--rule-2);
  transform:translateY(-101%);transition:transform .36s cubic-bezier(.4,0,.2,1)}
.stick.on{transform:translateY(0)}
.stick-in{max-width:var(--shell);margin:0 auto;padding:0 24px;display:flex;
  align-items:center;gap:20px;height:58px}
.stick-mark{font-family:var(--mono);font-size:11px;letter-spacing:.17em;
  color:var(--green-ink);display:flex;align-items:center;gap:9px;flex:none}
.stick-links{display:flex;gap:1px;overflow-x:auto;scrollbar-width:none;margin-left:auto}
.stick-links::-webkit-scrollbar{display:none}
.stick-links a{font-family:var(--mono);font-size:11.5px;color:var(--muted);
  text-decoration:none;padding:7px 12px;border-radius:5px;white-space:nowrap;
  position:relative;transition:color .16s ease,background .16s ease}
.stick-links a:hover{color:var(--ink);background:rgba(13,20,23,.06)}
.stick-links a.here{color:var(--green-ink);background:rgba(10,138,76,.1)}
@media (max-width:820px){.stick-mark{display:none}.stick-links{margin-left:0}}

/* ================================================================
   HERO
   ================================================================ */
.hero{position:relative;padding:0 24px;overflow:hidden;
  background:linear-gradient(175deg,#FFFFFF 0%,var(--page) 55%,#F4F1E9 100%);
  color:var(--read-text);border-bottom:1px solid var(--rule-2)}
.hero::before{content:"";position:absolute;inset:0;pointer-events:none;
  background-image:linear-gradient(rgba(13,20,23,.045) 1px,transparent 1px),
                   linear-gradient(90deg,rgba(13,20,23,.045) 1px,transparent 1px);
  background-size:44px 44px;
  mask-image:radial-gradient(ellipse 92% 74% at 50% 44%,#000 18%,transparent 76%);
  -webkit-mask-image:radial-gradient(ellipse 92% 74% at 50% 44%,#000 18%,transparent 76%)}
/* a slow ambient sweep, like a sensor refreshing */
.hero::after{content:"";position:absolute;inset:0;pointer-events:none;
  background:linear-gradient(100deg,transparent 38%,rgba(10,138,76,.05) 50%,transparent 62%);
  background-size:280% 100%;animation:sweep 9s linear infinite}
@keyframes sweep{0%{background-position:180% 0}100%{background-position:-80% 0}}

.hero-in{position:relative;max-width:var(--shell);margin:0 auto;
  padding:86px 0 82px;display:grid;grid-template-columns:1.02fr 1fr;
  gap:64px;align-items:center;z-index:1}
@media (max-width:980px){
  .hero-in{grid-template-columns:1fr;gap:44px;padding:62px 0 66px}
}

/* staged entrance after boot */
.up{opacity:0;transform:translateY(22px)}
body.ready .up{opacity:1;transform:none;
  transition:opacity .7s cubic-bezier(.2,.6,.3,1),transform .7s cubic-bezier(.2,.6,.3,1)}
body.ready .d1{transition-delay:.05s}
body.ready .d2{transition-delay:.15s}
body.ready .d3{transition-delay:.25s}
body.ready .d4{transition-delay:.35s}
body.ready .d5{transition-delay:.45s}

.eyebrow{font-family:var(--mono);font-size:11px;letter-spacing:.25em;
  text-transform:uppercase;color:var(--green-ink);display:flex;align-items:center;
  gap:11px;margin-bottom:28px}
.rec-dot{width:8px;height:8px;border-radius:50%;background:var(--ball);
  box-shadow:0 0 0 0 rgba(255,59,47,.65);animation:rec 2s ease-out infinite;flex:none}
@keyframes rec{
  0%{box-shadow:0 0 0 0 rgba(255,59,47,.6)}
  70%{box-shadow:0 0 0 12px rgba(255,59,47,0)}
  100%{box-shadow:0 0 0 0 rgba(255,59,47,0)}
}

h1.title{font-family:var(--display);font-weight:800;
  font-size:clamp(40px,6vw,76px);line-height:.96;letter-spacing:-.04em;
  color:var(--ink)}
h1.title .cv{color:var(--ball);position:relative;display:inline-block}
h1.title .cv::before,h1.title .cv::after{content:"";position:absolute;
  width:15px;height:15px;border:2px solid var(--green-ink)}
h1.title .cv::before{top:-10px;left:-12px;border-right:0;border-bottom:0}
h1.title .cv::after{bottom:-10px;right:-12px;border-left:0;border-top:0}

.lede{font-size:18px;line-height:1.66;color:var(--read-text);max-width:45ch;
  margin-top:28px}
.byline{font-family:var(--mono);font-size:12.5px;color:var(--muted);margin-top:24px}
.byline b{color:var(--ink);font-weight:500}

.cta{display:flex;gap:12px;flex-wrap:wrap;margin-top:36px}
.cta a{font-family:var(--mono);font-size:12.5px;letter-spacing:.04em;
  text-decoration:none;padding:13px 22px;border-radius:7px;
  border:1.5px solid var(--rule-2);color:var(--ink);background:var(--card);
  transition:all .2s cubic-bezier(.4,0,.2,1)}
.cta a.go{border-color:var(--ink);color:var(--card);background:var(--ink);font-weight:700}
.cta a.go:hover{transform:translateY(-2px);
  box-shadow:0 14px 30px -12px rgba(13,20,23,.55)}
.cta a.ghost:hover{border-color:var(--green-ink);color:var(--green-ink);
  transform:translateY(-2px);box-shadow:0 12px 26px -14px rgba(20,28,33,.3)}

/* ---------------- SIGNATURE: the live tracker ---------------- */
.scope{position:relative;border:1px solid var(--edge);border-radius:14px;
  background:#04070A;overflow:hidden;
  box-shadow:0 34px 70px -26px rgba(20,28,33,.45),
             0 6px 16px -8px rgba(20,28,33,.28),
             0 0 0 1px rgba(13,20,23,.06)}
.scope-bar{display:flex;align-items:center;gap:10px;padding:11px 15px;
  background:#070C0F;border-bottom:1px solid var(--edge);
  font-family:var(--mono);font-size:10.5px;letter-spacing:.13em;color:var(--faint)}
.scope-bar .live{color:var(--box)}
.scope-bar .spacer{flex:1}
.viewbtn{font-family:var(--mono);font-size:10px;letter-spacing:.09em;
  background:transparent;border:1px solid var(--edge-2);color:var(--dim);
  padding:5px 10px;border-radius:5px;cursor:pointer;transition:all .16s ease}
.viewbtn:hover{color:var(--box);border-color:var(--box)}
.viewbtn.on{color:var(--void);background:var(--box);border-color:var(--box);font-weight:700}
#scopeCanvas{display:block;width:100%;height:auto;background:#04070A;cursor:crosshair}
.scope-feet{display:flex;flex-wrap:wrap;border-top:1px solid var(--edge);background:#070C0F}
.foot{flex:1;min-width:86px;padding:12px 15px;border-right:1px solid var(--edge)}
.foot:last-child{border-right:0}
.foot .k{font-family:var(--mono);font-size:9px;letter-spacing:.17em;color:var(--faint)}
.foot .v{font-family:var(--mono);font-size:14px;color:var(--box);margin-top:4px;
  font-weight:500;font-variant-numeric:tabular-nums}
.foot .v.warn{color:var(--amber)}
.scope-hint{font-family:var(--mono);font-size:10.5px;color:var(--muted);
  text-align:center;margin-top:14px;letter-spacing:.05em}

/* ================================================================
   SPEC STRIP
   ================================================================ */
.specs{background:var(--card);border-bottom:1px solid var(--rule-2)}
.specs-in{max-width:var(--shell);margin:0 auto;padding:0 24px;
  display:grid;grid-template-columns:repeat(4,1fr)}
.spec{padding:30px 10px 30px 0}
.spec + .spec{padding-left:30px;border-left:1px solid var(--rule)}
.spec .n{font-family:var(--display);font-size:38px;font-weight:700;
  letter-spacing:-.035em;color:var(--ink);line-height:1;
  font-variant-numeric:tabular-nums}
.spec .n small{font-size:16px;color:var(--green-ink);font-weight:600;margin-left:3px}
.spec .l{font-family:var(--mono);font-size:10px;letter-spacing:.16em;
  text-transform:uppercase;color:var(--muted);margin-top:10px}
@media (max-width:780px){
  .specs-in{grid-template-columns:1fr 1fr}
  .spec:nth-child(3){padding-left:0;border-left:0}
  .spec:nth-child(3),.spec:nth-child(4){border-top:1px solid var(--rule)}
}

/* ================================================================
   THE LAB — interactive HSV playground
   ================================================================ */
.lab{border-top:1px solid var(--edge);border-bottom:1px solid var(--rule);
  background:var(--void);color:var(--text);padding:80px 24px 84px}
.lab-in{max-width:var(--shell);margin:0 auto}
.lab-head{max-width:var(--read);margin-bottom:38px}
.kicker{font-family:var(--mono);font-size:11px;letter-spacing:.19em;
  text-transform:uppercase;color:var(--box);margin-bottom:14px}
.lab-head h2{font-family:var(--display);font-size:clamp(27px,3.6vw,40px);
  font-weight:700;letter-spacing:-.035em;color:var(--bright);line-height:1.06}
.lab-head p{color:var(--dim);margin-top:16px;font-size:17px}
.lab-grid{display:grid;grid-template-columns:1fr 320px;gap:26px;align-items:start}
@media (max-width:900px){.lab-grid{grid-template-columns:1fr}}

.lab-stage{border:1px solid var(--edge);border-radius:13px;overflow:hidden;
  background:#04070A;box-shadow:0 30px 70px -34px rgba(0,0,0,.9)}
.lab-stage .scope-bar{border-radius:0}
#labCanvas{display:block;width:100%;height:auto;background:#04070A}

.panel{border:1px solid var(--edge);border-radius:13px;background:var(--surface);
  padding:22px;position:sticky;top:78px}
.panel h3{font-family:var(--mono);font-size:10.5px;letter-spacing:.17em;
  text-transform:uppercase;color:var(--faint);margin-bottom:20px;font-weight:500}
.knob{margin-bottom:20px}
.knob-top{display:flex;justify-content:space-between;align-items:baseline;
  font-family:var(--mono);font-size:11.5px;margin-bottom:9px}
.knob-top span{color:var(--text)}
.knob-top b{color:var(--box);font-weight:500;font-variant-numeric:tabular-nums}
input[type=range]{-webkit-appearance:none;appearance:none;width:100%;height:3px;
  background:var(--edge-2);border-radius:3px;outline:none;cursor:pointer}
input[type=range]::-webkit-slider-thumb{-webkit-appearance:none;appearance:none;
  width:15px;height:15px;border-radius:50%;background:var(--box);cursor:pointer;
  border:3px solid var(--surface);box-shadow:0 0 0 1px var(--box);
  transition:transform .14s ease}
input[type=range]::-webkit-slider-thumb:hover{transform:scale(1.22)}
input[type=range]::-moz-range-thumb{width:13px;height:13px;border-radius:50%;
  background:var(--box);cursor:pointer;border:3px solid var(--surface);
  box-shadow:0 0 0 1px var(--box)}
.verdict{margin-top:24px;padding-top:20px;border-top:1px solid var(--edge);
  font-family:var(--mono);font-size:11.5px;line-height:1.85}
.verdict div{display:flex;justify-content:space-between;gap:12px}
.verdict .g{color:var(--box)}
.verdict .a{color:var(--amber)}
.verdict .r{color:var(--ball)}
.verdict em{color:var(--faint);font-style:normal}
.reset{width:100%;margin-top:18px;font-family:var(--mono);font-size:10.5px;
  letter-spacing:.11em;background:transparent;border:1px solid var(--edge-2);
  color:var(--dim);padding:10px;border-radius:6px;cursor:pointer;
  transition:all .16s ease}
.reset:hover{color:var(--box);border-color:var(--box)}

/* ================================================================
   CONTENT
   ================================================================ */
.main{max-width:var(--read);margin:0 auto;padding:74px 24px 110px}
.intro{font-size:19.5px;line-height:1.66;color:var(--ink);
  border-left:3px solid var(--ball);padding-left:24px;margin-bottom:12px;
  font-weight:450}

h1.section{font-family:var(--display);font-size:clamp(29px,3.8vw,42px);
  font-weight:700;letter-spacing:-.036em;color:var(--ink);
  margin:104px 0 24px;padding-bottom:18px;border-bottom:1px solid var(--rule-2);
  line-height:1.04;position:relative}
h1.section::after{content:"";position:absolute;left:0;bottom:-1px;
  width:0;height:2px;background:var(--green-ink);
  transition:width 1.1s cubic-bezier(.2,.7,.3,1)}
h1.section.seen::after{width:74px}
h1.section .idx{font-family:var(--mono);font-size:11px;font-weight:700;
  color:var(--green-ink);letter-spacing:.18em;display:block;margin-bottom:12px}

.label{font-family:var(--display);font-weight:600;font-size:21px;
  color:var(--ink);margin:38px 0 9px;display:flex;align-items:center;gap:12px;
  letter-spacing:-.015em}
.label::before{content:"";width:7px;height:7px;background:var(--green-ink);
  border-radius:1px;transform:rotate(45deg);flex:none;
  box-shadow:0 0 0 3px rgba(10,138,76,.13)}

p{margin:16px 0;color:var(--read-text)}
ul{margin:16px 0 16px 2px;list-style:none}
ul li{position:relative;padding-left:26px;margin:9px 0;color:var(--read-text)}
ul li::before{content:"";position:absolute;left:3px;top:.78em;width:8px;height:1.5px;
  background:var(--green-ink)}
strong{color:var(--ink);font-weight:600}
em{color:var(--read-text)}
/* the dark instrument sections keep their own text colours */
.hero p,.hero ul li,.lab p{color:inherit}

code.inl{font-family:var(--mono);font-size:.85em;background:#F4EFE7;
  color:#B5301F;padding:2px 7px;border-radius:5px;white-space:nowrap;
  border:1px solid #E7DFD2}
a.link{color:var(--green-ink);text-decoration:none;
  border-bottom:1px solid rgba(10,138,76,.4);transition:all .16s ease}
a.link:hover{border-bottom-color:var(--green-ink);background:rgba(10,138,76,.07)}

/* ---------------- CODE ---------------- */
.code-wrap{background:var(--surface);border-radius:12px;margin:26px 0;
  overflow:hidden;border:1px solid var(--edge);
  box-shadow:0 26px 56px -30px rgba(0,0,0,.85);transition:border-color .25s ease}
.code-wrap:hover{border-color:var(--edge-2)}
.code-bar{display:flex;align-items:center;gap:8px;padding:12px 16px;
  background:#070C0F;border-bottom:1px solid var(--edge)}
.code-bar .dot{width:10px;height:10px;border-radius:50%;flex:none}
.d1{background:#FF5F57}.d2{background:#FEBC2E}.d3{background:#28C840}
.code-bar .fname{font-family:var(--mono);font-size:11.5px;color:var(--dim);
  margin-left:10px;flex:1}
.copy{font-family:var(--mono);font-size:10px;letter-spacing:.11em;color:var(--dim);
  background:transparent;border:1px solid var(--edge-2);border-radius:5px;
  padding:5px 12px;cursor:pointer;transition:all .16s ease;flex:none}
.copy:hover{color:var(--box);border-color:var(--box)}
.copy.done{color:var(--void);background:var(--box);border-color:var(--box);font-weight:700}
pre{margin:0;padding:22px 24px;overflow-x:auto;
  scrollbar-width:thin;scrollbar-color:var(--edge-2) transparent}
pre::-webkit-scrollbar{height:9px}
pre::-webkit-scrollbar-thumb{background:var(--edge-2);border-radius:5px}
pre::-webkit-scrollbar-track{background:transparent}
pre code{font-family:var(--mono);font-size:12.8px;line-height:1.66;
  color:#C3D0D5;white-space:pre}

/* ---------------- TABLE ---------------- */
table{width:100%;border-collapse:collapse;margin:28px 0;font-size:15px}
th{background:var(--void);color:var(--bright);font-family:var(--mono);
  font-size:10.5px;letter-spacing:.14em;text-transform:uppercase;text-align:left;
  padding:14px 16px;font-weight:500}
td{padding:14px 16px;border-bottom:1px solid var(--rule);vertical-align:top;
  color:var(--read-text);background:var(--card)}
tr:last-child td{border-bottom:none}
tbody tr{transition:background .16s ease}
tbody tr:hover td{background:#F5F2EA}
td a{color:var(--green-ink);text-decoration:none;font-family:var(--mono);font-size:12.5px}
td a:hover{text-decoration:underline}

/* ---------------- MEDIA ---------------- */
.video{position:relative;padding-bottom:56.25%;height:0;margin:26px 0;
  border-radius:12px;overflow:hidden;border:1px solid var(--rule-2);
  box-shadow:0 22px 48px -28px rgba(20,28,33,.35)}
.video iframe{position:absolute;top:0;left:0;width:100%;height:100%;border:0}
.imgrow{display:flex;flex-wrap:wrap;gap:14px;margin:24px 0}
.imgrow img{border-radius:10px;max-width:100%;border:1px solid var(--rule-2);
  transition:transform .3s cubic-bezier(.2,.7,.3,1),border-color .3s ease,
             box-shadow .3s ease}
.imgrow img:hover{transform:translateY(-4px);border-color:var(--green-ink);
  box-shadow:0 18px 36px -18px rgba(20,28,33,.4)}
.single-img{border-radius:12px;max-width:100%;border:1px solid var(--rule-2);margin:24px 0}

/* a dense schematic needs more width than the reading column allows,
   so it breaks out on either side and opens full size in a new tab */
.wide{width:calc(100% + 340px);margin-left:-170px;max-width:calc(100vw - 48px)}
.wide img{width:100%;height:auto;display:block;border-radius:12px;
  border:1px solid var(--rule-2);background:#fff;
  box-shadow:0 26px 60px -32px rgba(20,28,33,.45);
  transition:box-shadow .3s ease,border-color .3s ease}
.wide a:hover img{border-color:var(--green-ink);
  box-shadow:0 30px 66px -30px rgba(20,28,33,.55)}
.wide figcaption{font-family:var(--mono);font-size:11.5px;color:var(--muted);
  margin-top:12px;text-align:center;letter-spacing:.03em}
@media (max-width:1180px){.wide{width:100%;margin-left:0}}
.headshot{width:100%;max-width:440px;height:auto;border-radius:13px;
  border:2px solid var(--green-ink);margin:28px 0;display:block;
  box-shadow:0 24px 54px -30px rgba(20,28,33,.45)}

/* ---------------- FOOTER ---------------- */
.footer{border-top:1px solid var(--rule-2);margin-top:104px;padding-top:34px;
  font-family:var(--mono);font-size:12.5px;color:var(--muted);
  display:flex;flex-wrap:wrap;gap:14px;justify-content:space-between;
  align-items:baseline}
.footer .sig{color:var(--ink);font-weight:500}
.footer .top{color:var(--muted);text-decoration:none;transition:color .16s ease}
.footer .top:hover{color:var(--green-ink)}

/* ---------------- REVEAL + A11Y ---------------- */
.rise{opacity:0;transform:translateY(22px);
  transition:opacity .66s cubic-bezier(.2,.6,.3,1),transform .66s cubic-bezier(.2,.6,.3,1)}
.rise.seen{opacity:1;transform:none}
a:focus-visible,button:focus-visible,input:focus-visible{outline:2px solid var(--box);
  outline-offset:3px;border-radius:4px}
@media (prefers-reduced-motion:reduce){
  *{animation-duration:.01ms !important;animation-iteration-count:1 !important;
    transition-duration:.01ms !important;scroll-behavior:auto !important}
  .rise,.up{opacity:1;transform:none}
  .hero::after{display:none}
  #boot{display:none}
}
</style>
</head>
<body>

<!-- ============ BOOT ============ -->
<div id="boot">
  <div class="boot-in">
    <div class="l" data-l><i>[ 0.00 ]</i> gpio interface <b>READY</b></div>
    <div class="l" data-l><i>[ 0.34 ]</i> l9110 driver <b>READY</b></div>
    <div class="l" data-l><i>[ 0.61 ]</i> hc-sr04 ×3 <b>READY</b></div>
    <div class="l" data-l><i>[ 0.88 ]</i> servo pan <b>90°</b></div>
    <div class="l" data-l><i>[ 1.12 ]</i> picamera2 <b>480×360</b></div>
    <div class="l" data-l><i>[ 1.30 ]</i> opencv <b>ONLINE</b></div>
    <div class="boot-rail"><span id="bootRail"></span></div>
  </div>
</div>

<!-- ============ RETICLE ============ -->
<div id="retic"><i></i><i></i><i></i><i></i></div>

<div class="progress" id="progress"></div>

<nav class="stick" id="stick">
  <div class="stick-in">
    <div class="stick-mark"><span class="rec-dot"></span> CV_ONLINE</div>
    <div class="stick-links">
      <a href="#lab">Try it</a>
      <a href="#mods">Modifications</a>
      <a href="#final">Final</a>
      <a href="#m2">Milestone 2</a>
      <a href="#m1">Milestone 1</a>
      <a href="#schematics">Schematics</a>
      <a href="#bom">Materials</a>
      <a href="#starter">Starter</a>
      <a href="#resources">Resources</a>
    </div>
  </div>
</nav>

<header class="hero">
  <div class="hero-in">

    <div class="hero-copy">
      <div class="eyebrow up d1"><span class="rec-dot"></span> Computer Vision · Raspberry Pi 4</div>
      <h1 class="title up d2">Ball Tracking<br>Robot with <span class="cv">OpenCV</span></h1>
      <p class="lede up d3">A robot that finds a red ball in a camera frame, pans its
        camera to keep the ball centred, and drives toward it while reading distance
        off three ultrasonic sensors.</p>
      <div class="byline up d4"><b>Vaideesh K</b> · Cupertino High School · Electrical Engineering</div>
      <div class="cta up d5">
        <a class="go" href="#lab" data-lock>Try the detector</a>
        <a class="ghost" href="#final" data-lock>See the final build</a>
      </div>
    </div>

    <div class="up d3">
      <div class="scope">
        <div class="scope-bar">
          <span class="live">● REC</span>
          <span>480 × 360</span>
          <span class="spacer"></span>
          <button class="viewbtn on" id="btnCam" type="button" data-lock>CAMERA</button>
          <button class="viewbtn" id="btnMask" type="button" data-lock>HSV MASK</button>
        </div>
        <canvas id="scopeCanvas" width="640" height="430"></canvas>
        <div class="scope-feet">
          <div class="foot"><div class="k">STATUS</div><div class="v" id="fStatus">LOCKED</div></div>
          <div class="foot"><div class="k">ERROR X</div><div class="v" id="fErr">0 px</div></div>
          <div class="foot"><div class="k">RADIUS</div><div class="v" id="fRad">0 px</div></div>
          <div class="foot"><div class="k">SERVO</div><div class="v" id="fServo">90°</div></div>
        </div>
      </div>
      <div class="scope-hint">move your cursor near the ball to push it</div>
    </div>

  </div>
</header>

<section class="specs">
  <div class="specs-in">
    <div class="spec"><div class="n" data-count="3">0</div><div class="l">Milestones</div></div>
    <div class="spec"><div class="n" data-count="3">0</div><div class="l">Ultrasonic sensors</div></div>
    <div class="spec"><div class="n" data-count="140">0</div><div class="l">Degree pan range</div></div>
    <div class="spec"><div class="n" data-count="40">0</div><div class="l">Wired connections</div></div>
  </div>
</section>

<!-- ============ THE LAB ============ -->
<section class="lab" id="lab">
  <div class="lab-in">
    <div class="lab-head">
      <div class="kicker">Interactive · the hard part</div>
      <h2>The robot doesn't see a ball.<br>It sees a range of colour.</h2>
      <p>Every frame gets converted to HSV and filtered down to a black-and-white
        mask. Whatever survives the filter is what the robot chases. Getting these
        three numbers wrong is why it spent an afternoon following my face instead
        of the ball — skin is dull red, and the saturation floor was too low to
        tell them apart. Drag the sliders and watch it happen.</p>
    </div>

    <div class="lab-grid">
      <div class="lab-stage">
        <div class="scope-bar">
          <span class="live">● MASK OUTPUT</span>
          <span class="spacer"></span>
          <span id="labVerdict">2 objects pass</span>
        </div>
        <canvas id="labCanvas" width="760" height="380"></canvas>
      </div>

      <div class="panel">
        <h3>HSV Threshold</h3>

        <div class="knob">
          <div class="knob-top"><span>Hue window</span><b id="vHue">±10</b></div>
          <input type="range" id="sHue" min="4" max="60" value="10">
        </div>

        <div class="knob">
          <div class="knob-top"><span>Saturation floor</span><b id="vSat">170</b></div>
          <input type="range" id="sSat" min="40" max="240" value="170">
        </div>

        <div class="knob">
          <div class="knob-top"><span>Value floor</span><b id="vVal">80</b></div>
          <input type="range" id="sVal" min="20" max="200" value="80">
        </div>

        <div class="knob">
          <div class="knob-top"><span>Min roundness</span><b id="vFill">0.40</b></div>
          <input type="range" id="sFill" min="0" max="90" value="40">
        </div>

        <div class="verdict">
          <div><em>Red ball</em><span id="rBall" class="g">TRACKED</span></div>
          <div><em>Skin tone</em><span id="rFace" class="g">rejected</span></div>
          <div><em>Red mug</em><span id="rMug" class="g">rejected</span></div>
          <div><em>Orange</em><span id="rOrange" class="g">rejected</span></div>
        </div>

        <button class="reset" id="resetLab" type="button" data-lock>RESET TO MY VALUES</button>
      </div>
    </div>
  </div>
</section>

<main class="main">

  <p class="intro">The Ball Tracking Robot with OpenCV uses a Raspberry Pi 4 computer, a 5MP camera, and Python to create a robot that avoids obstacles, moves independently, and makes decisions about where to navigate. This project involves building circuits to make connections, integrating hardware and software, and programming.</p>

  <table>
    <thead><tr><th>Engineer</th><th>School</th><th>Area of Interest</th><th>Grade</th></tr></thead>
    <tbody><tr><td>Vaideesh K</td><td>Cupertino High School</td><td>Electrical Engineering</td><td>Incoming Senior</td></tr></tbody>
  </table>

  <img class="headshot" src="Vaideesh%20K.jpg" alt="Vaideesh K">

  <!-- ============ MODIFICATIONS ============ -->
  <h1 class="section" id="mods"><span class="idx">EXTENSION / POST-FINAL</span>Modifications</h1>

  <div class="label">Summary</div>
  <p>After finishing my final milestone, I went back and upgraded the robot with three big improvements: variable speed control using PWM, a servo that pans the camera to keep the ball centered, and a web dashboard that streams the camera feed live and lets me start and stop the robot from my phone. In the original code the motors were either fully on or fully off, the camera was fixed in place, and I could only see what the robot saw if I was plugged into a monitor. With these modifications the robot now slows down smoothly as it gets close to the ball, physically turns the camera to follow the ball instead of just steering the whole body, and I can control and watch everything from a browser on any device connected to the same network. I also made the ball detection steadier so the tracking is less jumpy.</p>

  <div class="label">PWM Speed Control</div>
  <p>In my final milestone the motors only knew two states — full power or off — so the robot moved in jerky bursts. I switched every motor pin over to PWM (Pulse Width Modulation), which rapidly turns the pin on and off to control how much power the motor actually gets, so now I can set any speed from 0 to 100 percent. I use this to make the robot cruise fast when the ball is far away and automatically slow down as it gets closer, so it eases up to the ball instead of slamming into it. The <code class="inl">speed_for()</code> function does this by mapping the distance to a speed: far away returns full cruise speed, close returns the minimum creep speed, and anything in between scales smoothly between the two. I also added an <code class="inl">INSIDE</code> factor to the turn functions so that when turning, the inside wheel spins slower than the outside wheel, which gives a smoother curve instead of a sharp pivot.</p>

  <div class="label">Servo Camera Panning</div>
  <p>The biggest change was adding a servo motor under the camera so the camera can physically turn left and right on its own. Before, if the ball moved to the side, the whole robot had to rotate to keep it in view. Now the camera pans to follow the ball while the body stays pointed forward, which makes tracking much smoother. The code figures out how far the ball is from the center of the frame (the "error"), and if that error is bigger than a deadzone, it nudges the servo a small step in that direction to re-center the ball. The deadzone stops the servo from twitching constantly when the ball is basically centered. There is a <code class="inl">SERVO_DIR</code> setting I can flip if the servo turns the wrong way, and min/max limits so it cannot try to turn past its physical range.</p>

  <div class="label">Web Control Dashboard</div>
  <p>To make the robot easier to use and to show it off, I added a web interface using Flask. The program runs two things at the same time using threading: one thread is the "brain" that captures frames, detects the ball, reads the sensors, and drives the motors and servo, and the other thread runs a small web server. The web page has a live video feed of what the camera sees, plus START and STOP buttons. The video is streamed as MJPEG, which is basically a fast sequence of JPEG images, and I compress each frame to 60 percent quality so it streams smoothly without lag. Now I can open a browser on my phone, go to the Pi's IP address, and watch and control the robot with no monitor or keyboard plugged in.</p>

  <div class="label">Steadier Detection</div>
  <p>I also cleaned up the ball detection. Instead of just using the biggest red blob, the code now scores each candidate by how well it fills a circle (using <code class="inl">minEnclosingCircle</code>) combined with its size, and picks the best one. On top of that I added smoothing so the tracked position is a blend of the old position and the new one (a 60/40 mix), which stops the box from jumping around frame to frame. I also added a "lost hold" so if the ball disappears for a few frames the robot does not instantly give up — it holds the last known position for a short time in case the ball just flickered out.</p>

  <div class="label">Modified Code — PWM, Servo, and Web Control</div>
  <p>This is the full upgraded program. It combines the camera, motors, servo, and ultrasonic sensors, adds PWM speed control and servo panning, and serves a live video stream with START/STOP buttons to a web page. All the settings I tune the most are grouped at the top so they are easy to adjust.</p>
  <div class="code-wrap">
<div class="code-bar"><span class="dot d1"></span><span class="dot d2"></span><span class="dot d3"></span><span class="fname">nano_demo.py</span></div>
<pre><code>from flask import Flask, Response, render_template_string
from picamera2 import Picamera2
import cv2
import numpy as np
import RPi.GPIO as GPIO
import time
import threading

# ================= TUNING =================
# motors
MIN_SPEED    = 55
CRUISE_SPEED = 100
TURN_SPEED   = 90
AVOID_SPEED  = 85
SLOW_FROM    = 60
ARRIVE_AT    = 15
OBSTACLE_AT  = 20
SIDE_EVERY   = 8

# detection
MIN_AREA   = 400
MIN_RADIUS = 20
MAX_RADIUS = 100
MIN_FILL   = 0.55
LOST_HOLD  = 10

# servo camera pan
SERVO_DIR = -1        # flip to 1 if camera turns away from the ball
SERVO_GAIN = 0.03
SERVO_MAX_STEP = 3
SERVO_DEADZONE = 55
SERVO_MIN = 20
SERVO_MAX = 160

# ================= MOTORS =================
GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)
A1A = 6; A1B = 5; B1A = 22; B2A = 23
for p in [A1A, A1B, B1A, B2A]:
    GPIO.setup(p, GPIO.OUT)
pA1A = GPIO.PWM(A1A, 1000); pA1B = GPIO.PWM(A1B, 1000)
pB1A = GPIO.PWM(B1A, 1000); pB2A = GPIO.PWM(B2A, 1000)
for p in [pA1A, pA1B, pB1A, pB2A]:
    p.start(0)

INSIDE = 0.4
def forward(s):
    pA1A.ChangeDutyCycle(0); pA1B.ChangeDutyCycle(s)
    pB1A.ChangeDutyCycle(0); pB2A.ChangeDutyCycle(s)
def left(s):
    pA1A.ChangeDutyCycle(0); pA1B.ChangeDutyCycle(int(s * INSIDE))
    pB1A.ChangeDutyCycle(0); pB2A.ChangeDutyCycle(s)
def right(s):
    pA1A.ChangeDutyCycle(0); pA1B.ChangeDutyCycle(s)
    pB1A.ChangeDutyCycle(0); pB2A.ChangeDutyCycle(int(s * INSIDE))
def stop():
    for p in [pA1A, pA1B, pB1A, pB2A]:
        p.ChangeDutyCycle(0)

# ================= SERVO =================
SERVO = 18
GPIO.setup(SERVO, GPIO.OUT)
servo_pwm = GPIO.PWM(SERVO, 50)
servo_pwm.start(0)
cam_angle = 90.0
def drive_servo(a):
    a = max(SERVO_MIN, min(SERVO_MAX, a))
    duty = 2.5 + (a / 180.0) * 10.0
    servo_pwm.ChangeDutyCycle(duty)
    time.sleep(0.02)
    servo_pwm.ChangeDutyCycle(0)
    return a
cam_angle = drive_servo(cam_angle)

# ================= SENSORS =================
SENSORS = {&quot;LEFT&quot;: (19, 26), &quot;CENTER&quot;: (16, 20), &quot;RIGHT&quot;: (11, 12)}
for trig, echo in SENSORS.values():
    GPIO.setup(trig, GPIO.OUT)
    GPIO.setup(echo, GPIO.IN)
    GPIO.output(trig, False)
def measure(trig, echo):
    start = time.time(); stop_t = time.time()
    GPIO.output(trig, True); time.sleep(0.00001); GPIO.output(trig, False)
    t = time.time() + 0.006
    while GPIO.input(echo) == 0 and time.time() &lt; t: start = time.time()
    t = time.time() + 0.006
    while GPIO.input(echo) == 1 and time.time() &lt; t: stop_t = time.time()
    d = round((stop_t - start) * 34300 / 2, 1)
    return d if 0 &lt; d &lt; 400 else 400

# ================= CAMERA =================
picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={&quot;format&quot;: &quot;RGB888&quot;, &quot;size&quot;: (480, 360)}))
picam2.start()
time.sleep(2)

W = 480
CENTER = W // 2
LEFT_EDGE  = W // 3
RIGHT_EDGE = W * 2 // 3
kernel = np.ones((5, 5), np.uint8)

running = False
latest = None
lock = threading.Lock()
dL = dR = 400
frame_count = 0
sx = None
lost = 0

def speed_for(d):
    if d &gt;= SLOW_FROM: return CRUISE_SPEED
    if d &lt;= ARRIVE_AT: return MIN_SPEED
    frac = (d - ARRIVE_AT) / float(SLOW_FROM - ARRIVE_AT)
    return int(MIN_SPEED + frac * (CRUISE_SPEED - MIN_SPEED))

def brain():
    global latest, dL, dR, frame_count, sx, lost, cam_angle
    while True:
        frame_count += 1
        frame = picam2.capture_array()
        frame = cv2.flip(frame, -1)

        dC = measure(*SENSORS[&quot;CENTER&quot;])
        if frame_count % SIDE_EVERY == 0:
            dL = measure(*SENSORS[&quot;LEFT&quot;])
            dR = measure(*SENSORS[&quot;RIGHT&quot;])

        # ---- DETECTION ----
        blurred = cv2.GaussianBlur(frame, (5, 5), 0)
        hsv = cv2.cvtColor(blurred, cv2.COLOR_BGR2HSV)
        lo1 = np.array([0, 150, 90]);   hi1 = np.array([10, 255, 255])
        lo2 = np.array([170, 150, 90]); hi2 = np.array([180, 255, 255])
        mask = cv2.inRange(hsv, lo1, hi1) + cv2.inRange(hsv, lo2, hi2)
        mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN,  kernel)
        mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel, iterations=2)
        mask = cv2.copyMakeBorder(mask, 1, 1, 1, 1, cv2.BORDER_CONSTANT, value=0)

        contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
        best = None; best_score = 0
        for c in contours:
            area = cv2.contourArea(c)
            if area &lt; MIN_AREA:
                continue
            (mx, my), mr = cv2.minEnclosingCircle(c)
            if mr &lt; MIN_RADIUS or mr &gt; MAX_RADIUS:
                continue
            fill = area / (np.pi * mr * mr) if mr &gt; 0 else 0
            if fill &lt; MIN_FILL:
                continue
            score = fill * area
            if score &gt; best_score:
                best_score = score
                best = (int(mx), int(my), int(mr))

        # smooth + hold
        if best is not None:
            mx = best[0]
            sx = mx if sx is None else 0.6 * sx + 0.4 * mx
            lost = 0
        else:
            lost += 1
            if lost &gt; LOST_HOLD:
                sx = None

        ball = sx is not None
        pos = &quot;NONE&quot;
        if ball:
            cx = int(sx)
            if cx &lt; LEFT_EDGE:    pos = &quot;LEFT&quot;
            elif cx &gt; RIGHT_EDGE: pos = &quot;RIGHT&quot;
            else:                 pos = &quot;CENTER&quot;
            if best is not None:
                bx, by, br = best
                cv2.circle(frame, (bx, by), br, (0, 255, 0), 3)

            # ---- SERVO: pan camera to keep ball centered ----
            err = cx - CENTER
            if abs(err) &gt; SERVO_DEADZONE:
                stepc = SERVO_DIR * SERVO_GAIN * err
                stepc = max(-SERVO_MAX_STEP, min(SERVO_MAX_STEP, stepc))
                cam_angle = cam_angle + stepc
        cam_angle = drive_servo(cam_angle)

        # ---- DRIVE ----
        if not running:
            stop(); action = &quot;STOPPED&quot;
        elif dL &lt; OBSTACLE_AT and dL &lt; dR:
            right(AVOID_SPEED); action = &quot;avoid&quot;
        elif dR &lt; OBSTACLE_AT:
            left(AVOID_SPEED);  action = &quot;avoid&quot;
        elif ball:
            if 0 &lt; dC &lt;= ARRIVE_AT:
                stop(); action = &quot;ARRIVED&quot;
            elif pos == &quot;LEFT&quot;:
                left(TURN_SPEED);  action = &quot;left&quot;
            elif pos == &quot;RIGHT&quot;:
                right(TURN_SPEED); action = &quot;right&quot;
            else:
                s = speed_for(dC)
                forward(s); action = f&quot;fwd {s}%&quot;
        else:
            stop(); action = &quot;no ball&quot;

        cv2.line(frame, (LEFT_EDGE, 0),  (LEFT_EDGE, 360),  (80, 80, 80), 1)
        cv2.line(frame, (RIGHT_EDGE, 0), (RIGHT_EDGE, 360), (80, 80, 80), 1)
        cv2.putText(frame, f&quot;{pos} | {action}&quot;, (10, 30),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)
        cv2.putText(frame, f&quot;C{dC}  cam={int(cam_angle)}&quot;, (10, 60),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)

        ok, jpg = cv2.imencode(&#x27;.jpg&#x27;, frame, [int(cv2.IMWRITE_JPEG_QUALITY), 60])
        if ok:
            with lock:
                latest = jpg.tobytes()

threading.Thread(target=brain, daemon=True).start()

app = Flask(__name__)
PAGE = &quot;&quot;&quot;
&lt;html&gt;&lt;head&gt;&lt;title&gt;Ball Tracking Robot&lt;/title&gt;
&lt;style&gt;
 body{background:#111;color:#eee;font-family:sans-serif;text-align:center}
 img{border:3px solid #444;border-radius:8px;margin-top:15px;width:90%;max-width:600px}
 button{font-size:20px;padding:14px 34px;margin:10px;border:none;border-radius:6px;cursor:pointer}
 .go{background:#2a7;color:#fff} .no{background:#a33;color:#fff}
&lt;/style&gt;&lt;/head&gt;
&lt;body&gt;
 &lt;h1&gt;Ball Tracking Robot&lt;/h1&gt;
 &lt;button class=&quot;go&quot; onclick=&quot;fetch(&#x27;/start&#x27;)&quot;&gt;START&lt;/button&gt;
 &lt;button class=&quot;no&quot; onclick=&quot;fetch(&#x27;/stop&#x27;)&quot;&gt;STOP&lt;/button&gt;
 &lt;br&gt;&lt;img src=&quot;/video&quot;&gt;
&lt;/body&gt;&lt;/html&gt;
&quot;&quot;&quot;

def gen():
    while True:
        with lock:
            d = latest
        if d is None:
            time.sleep(0.03); continue
        yield (b&#x27;--frame\r\nContent-Type: image/jpeg\r\n\r\n&#x27; + d + b&#x27;\r\n&#x27;)
        time.sleep(0.03)

@app.route(&#x27;/&#x27;)
def index(): return render_template_string(PAGE)

@app.route(&#x27;/start&#x27;)
def go():
    global running; running = True; return &quot;started&quot;

@app.route(&#x27;/stop&#x27;)
def halt():
    global running; running = False; stop(); return &quot;stopped&quot;

@app.route(&#x27;/video&#x27;)
def video():
    return Response(gen(), mimetype=&#x27;multipart/x-mixed-replace; boundary=frame&#x27;)

try:
    app.run(host=&#x27;0.0.0.0&#x27;, port=5000, threaded=True)
finally:
    stop(); servo_pwm.stop(); GPIO.cleanup()</code></pre>
</div>

  <!-- ============ FINAL MILESTONE ============ -->
  <h1 class="section" id="final"><span class="idx">MILESTONE 03 / FINAL</span>Final Milestone</h1>
  <div class="video"><iframe src="https://www.youtube.com/embed/IsikH-t7laU" title="Vaideesh K. Milestone 3" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe></div>

  <div class="label">Summary</div>
  <p>My third and final milestone is the last portion of my project. In this part I installed a 5MP Raspberry Pi Camera and used OpenCV to run the vision code. The finished robot combines the three ultrasonic sensors, the L9110 motor driver, and the two motors so that it can detect and follow a red ball while using the three ultrasonic sensors to measure the distance to objects on the left, center, and right, and to detect and avoid obstacles in real time. The camera sees the ball and decides whether it is on the left, center, or right, and the robot turns or drives forward to follow it, stopping when it gets close. To make sure the robot only follows the ball and not any other red object, I added a circularity check that measures how round each red object is, so only round objects like the ball are tracked.</p>

  <div class="label">Challenges</div>
  <p>A major challenge I faced was getting the camera to be detected by the Raspberry Pi 4 Model B. I ran <code class="inl">rpicam-hello</code> to turn on the camera and check for a live preview, but it did not work. My next step was to completely power down the Pi by unplugging the USB-C cable and reseating the camera ribbon cable at both the camera module and the connector on the Raspberry Pi. I rebooted and ran it again, but it still did not work, so I tried three different cameras of the same model with different ribbon cables. I ran <code class="inl">rpicam-hello --list-cameras</code> to see if anything would show up, but no cameras were available. I then ran an update to see if that would fix the issue, but nothing changed. As a last resort I created a camera test — I made a file, wrote the camera code, saved it, and ran it, and it finally worked.</p>
  <p>Another challenge was making the robot track only the red ball and not every red object in the room. At first the code just picked the largest red blob, so it would follow red shirts or anything else red. I fixed this by adding a circularity calculation that compares each object's area to its perimeter to measure how round it is. I also had to clean up the mask with morphological operations, because glare on the ball punched holes in the detected shape and ruined the roundness math. I also had to fix the camera orientation, since the image was coming in upside down.</p>

  <div class="label">Camera Test Code</div>
  <p>This is the camera test I used to confirm the camera was working with OpenCV. It grabs frames from the Pi Camera using picamera2 and displays them in a live window. Getting this to run was the fix that finally got my camera working after it wouldn't show a preview.</p>
  <div class="code-wrap">
<div class="code-bar"><span class="dot d1"></span><span class="dot d2"></span><span class="dot d3"></span><span class="fname">camera_test.py</span></div>
<pre><code>from picamera2 import Picamera2
import cv2

picam2 = Picamera2()
config = picam2.create_preview_configuration(main={&quot;format&quot;: &quot;RGB888&quot;, &quot;size&quot;: (320, 240)})
picam2.configure(config)
picam2.start()

print(&quot;Camera running - press q in the window to quit&quot;)
while True:
    frame = picam2.capture_array()
    cv2.imshow(&quot;Camera&quot;, frame)
    if cv2.waitKey(1) &amp; 0xFF == ord(&#x27;q&#x27;):
        break

cv2.destroyAllWindows()
picam2.stop()</code></pre>
</div>

  <div class="label">Ball Detection Code</div>
  <p>After the camera worked, I wrote this code to detect the red ball. It converts each frame to HSV, filters for red, cleans up the mask so glare doesn't break the shape, and then calculates the circularity of each red object. Only round objects count as the ball, so other red objects get ignored. The code draws a green box when it finds the ball and prints whether the ball is on the left, center, or right.</p>
  <div class="code-wrap">
<div class="code-bar"><span class="dot d1"></span><span class="dot d2"></span><span class="dot d3"></span><span class="fname">ball_detection.py</span></div>
<pre><code>from picamera2 import Picamera2
import cv2
import numpy as np

picam2 = Picamera2()
config = picam2.create_preview_configuration(main={&quot;format&quot;: &quot;RGB888&quot;, &quot;size&quot;: (320, 240)})
picam2.configure(config)
picam2.start()

kernel = np.ones((5, 5), np.uint8)

# Roundness threshold - raise it if it tracks non-ball red objects,
# lower it if it misses the ball
MIN_CIRCULARITY = 0.4

print(&quot;Detecting red ball - press q to quit&quot;)
while True:
    frame = picam2.capture_array()
    frame = cv2.flip(frame, -1)

    blurred = cv2.GaussianBlur(frame, (5, 5), 0)
    hsv = cv2.cvtColor(blurred, cv2.COLOR_BGR2HSV)

    lower1 = np.array([0, 150, 80])
    upper1 = np.array([10, 255, 255])
    lower2 = np.array([170, 150, 80])
    upper2 = np.array([180, 255, 255])
    mask = cv2.inRange(hsv, lower1, upper1) + cv2.inRange(hsv, lower2, upper2)

    # Clean the mask - glare punches holes in the ball and wrecks the
    # perimeter math, so closing fills them back in
    mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)
    mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel, iterations=3)
    mask = cv2.dilate(mask, kernel, iterations=1)

    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

    best = None
    best_area = 0
    fallback = None
    fallback_area = 0

    for c in contours:
        area = cv2.contourArea(c)
        if area &lt; 300:
            continue

        perimeter = cv2.arcLength(c, True)
        if perimeter == 0:
            continue
        circularity = 4 * np.pi * area / (perimeter * perimeter)

        # Track the biggest round one
        if circularity &gt;= MIN_CIRCULARITY and area &gt; best_area:
            best = (c, circularity)
            best_area = area

        # Also track the biggest blob overall, in case nothing is round
        if area &gt; fallback_area:
            fallback = (c, circularity)
            fallback_area = area

    chosen = best if best is not None else fallback

    if chosen is not None:
        c, circ = chosen
        area = cv2.contourArea(c)
        x, y, w, h = cv2.boundingRect(c)
        cx = x + w // 2

        # Green box = passed roundness (it&#x27;s the ball)
        # Red box = failed roundness (probably not the ball)
        color = (0, 255, 0) if circ &gt;= MIN_CIRCULARITY else (0, 0, 255)
        cv2.rectangle(frame, (x, y), (x+w, y+h), color, 2)

        if cx &lt; 107:
            pos = &quot;LEFT&quot;
        elif cx &gt; 213:
            pos = &quot;RIGHT&quot;
        else:
            pos = &quot;CENTER&quot;

        label = f&quot;{pos}  circ={circ:.2f}&quot;
        cv2.putText(frame, label, (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 0.7, color, 2)
        print(f&quot;Ball: {pos}  area: {int(area)}  circularity: {circ:.2f}&quot;)

    cv2.imshow(&quot;Ball Tracking&quot;, frame)

    if cv2.waitKey(1) &amp; 0xFF == ord(&#x27;q&#x27;):
        break

cv2.destroyAllWindows()
picam2.stop()</code></pre>
</div>

  <div class="label">Final Code — Full Ball Tracking Robot</div>
  <p>This is the complete program that combines the camera, the motors, and the ultrasonic sensors. The camera detects the red ball, checks that it is round so other red objects are ignored, and decides if the ball is on the left, center, or right. Based on that, the robot turns or drives forward to follow the ball, and the center ultrasonic sensor stops the robot when it gets close so it does not crash into the ball.</p>
  <div class="code-wrap">
<div class="code-bar"><span class="dot d1"></span><span class="dot d2"></span><span class="dot d3"></span><span class="fname">ball_tracking_robot.py</span></div>
<pre><code>from picamera2 import Picamera2
import cv2
import numpy as np
import RPi.GPIO as GPIO
import time

# ---------- MOTOR SETUP ----------
GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

A1A = 6; A1B = 5; B1A = 22; B2A = 23
for p in [A1A, A1B, B1A, B2A]:
    GPIO.setup(p, GPIO.OUT)

def forward():
    GPIO.output(A1A, GPIO.LOW); GPIO.output(A1B, GPIO.HIGH)
    GPIO.output(B1A, GPIO.LOW); GPIO.output(B2A, GPIO.HIGH)
def backward():
    GPIO.output(A1A, GPIO.HIGH); GPIO.output(A1B, GPIO.LOW)
    GPIO.output(B1A, GPIO.HIGH); GPIO.output(B2A, GPIO.LOW)
def left():
    GPIO.output(A1A, GPIO.LOW); GPIO.output(A1B, GPIO.LOW)
    GPIO.output(B1A, GPIO.LOW); GPIO.output(B2A, GPIO.HIGH)
def right():
    GPIO.output(A1A, GPIO.LOW); GPIO.output(A1B, GPIO.HIGH)
    GPIO.output(B1A, GPIO.LOW); GPIO.output(B2A, GPIO.LOW)
def stop():
    for p in [A1A, A1B, B1A, B2A]: GPIO.output(p, GPIO.LOW)

# ---------- SENSOR SETUP ----------
sensors = [(&quot;LEFT&quot;, 19, 26), (&quot;CENTER&quot;, 16, 20), (&quot;RIGHT&quot;, 11, 12)]
for name, trig, echo in sensors:
    GPIO.setup(trig, GPIO.OUT)
    GPIO.setup(echo, GPIO.IN)
    GPIO.output(trig, False)

def measure(trig, echo):
    start = time.time()
    stop_t = time.time()
    GPIO.output(trig, True)
    time.sleep(0.00001)
    GPIO.output(trig, False)
    t = time.time() + 0.05
    while GPIO.input(echo) == 0 and time.time() &lt; t:
        start = time.time()
    t = time.time() + 0.05
    while GPIO.input(echo) == 1 and time.time() &lt; t:
        stop_t = time.time()
    return round((stop_t - start) * 34300 / 2, 1)

# ---------- CAMERA SETUP ----------
picam2 = Picamera2()
config = picam2.create_preview_configuration(main={&quot;format&quot;: &quot;RGB888&quot;, &quot;size&quot;: (320, 240)})
picam2.configure(config)
picam2.start()
time.sleep(2)

kernel = np.ones((5, 5), np.uint8)

# Roundness threshold - raise it if it tracks non-ball red objects,
# lower it if it misses the ball
MIN_CIRCULARITY = 0.4

print(&quot;Ball tracking robot running - press q to quit&quot;)

try:
    while True:
        frame = picam2.capture_array()
        frame = cv2.flip(frame, -1)

        blurred = cv2.GaussianBlur(frame, (5, 5), 0)
        hsv = cv2.cvtColor(blurred, cv2.COLOR_BGR2HSV)

        lower1 = np.array([0, 150, 80])
        upper1 = np.array([10, 255, 255])
        lower2 = np.array([170, 150, 80])
        upper2 = np.array([180, 255, 255])
        mask = cv2.inRange(hsv, lower1, upper1) + cv2.inRange(hsv, lower2, upper2)

        # Clean the mask - fills glare holes so the roundness math works
        mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)
        mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel, iterations=3)
        mask = cv2.dilate(mask, kernel, iterations=1)

        center_dist = measure(16, 20)

        contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        # Find the biggest ROUND red blob (the ball)
        best = None
        best_area = 0
        fallback = None
        fallback_area = 0

        for c in contours:
            area = cv2.contourArea(c)
            if area &lt; 300:
                continue
            perimeter = cv2.arcLength(c, True)
            if perimeter == 0:
                continue
            circularity = 4 * np.pi * area / (perimeter * perimeter)

            if circularity &gt;= MIN_CIRCULARITY and area &gt; best_area:
                best = (c, circularity)
                best_area = area

            if area &gt; fallback_area:
                fallback = (c, circularity)
                fallback_area = area

        chosen = best if best is not None else fallback

        ball_found = False
        pos = &quot;NONE&quot;
        circ = 0.0

        if chosen is not None:
            c, circ = chosen
            # Only treat it as the ball if it passed the roundness test
            if circ &gt;= MIN_CIRCULARITY:
                ball_found = True

            area = cv2.contourArea(c)
            x, y, w, h = cv2.boundingRect(c)
            cx = x + w // 2

            # Green box = it&#x27;s the ball. Red box = red thing, not round enough.
            color = (0, 255, 0) if ball_found else (0, 0, 255)
            cv2.rectangle(frame, (x, y), (x+w, y+h), color, 2)

            if cx &lt; 107:
                pos = &quot;LEFT&quot;
            elif cx &gt; 213:
                pos = &quot;RIGHT&quot;
            else:
                pos = &quot;CENTER&quot;

        # ---------- DECISION LOGIC ----------
        if ball_found:
            if 0 &lt; center_dist &lt; 15:
                stop()
                action = &quot;ARRIVED - stopped&quot;
            elif pos == &quot;LEFT&quot;:
                left()
                action = &quot;turning left&quot;
            elif pos == &quot;RIGHT&quot;:
                right()
                action = &quot;turning right&quot;
            else:
                forward()
                action = &quot;driving forward&quot;
        else:
            stop()
            action = &quot;searching (no ball)&quot;

        cv2.putText(frame, f&quot;{pos} | {action}&quot;, (10, 30),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)
        cv2.putText(frame, f&quot;dist: {center_dist} cm  circ: {circ:.2f}&quot;, (10, 60),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)
        cv2.imshow(&quot;Ball Tracking Robot&quot;, frame)

        print(f&quot;Ball: {pos} | {action} | dist: {center_dist} cm | circ: {circ:.2f}&quot;)

        if cv2.waitKey(1) &amp; 0xFF == ord(&#x27;q&#x27;):
            break

except KeyboardInterrupt:
    pass

stop()
cv2.destroyAllWindows()
picam2.stop()
GPIO.cleanup()
print(&quot;Stopped and cleaned up&quot;)</code></pre>
</div>

  <!-- ============ SECOND MILESTONE ============ -->
  <h1 class="section" id="m2"><span class="idx">MILESTONE 02</span>Second Milestone</h1>
  <div class="video"><iframe src="https://www.youtube.com/embed/lQya6fe888A" title="Vaideesh K. Milestone 2" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe></div>

  <div class="label">Summary</div>
  <p>My second milestone was the biggest portion of my project. I had to get the motors spinning, which in turn makes the wheels spin, and I had to get all three ultrasonic sensors working. I wrote code for both — the motor code drives the robot forward, backward, left, and right, and the sensor code uses the Pi to send a signal to the Trigger pin, releasing a burst of ultrasonic sound at approximately 40,000 Hz. The sound travels out, hits an object, and bounces back, and the time it takes to return is used to calculate the distance.</p>

  <div class="label">Challenges</div>
  <p>The biggest challenge I faced was getting the motors running. I tried four different L298N motor driver boards because the first one arrived defective — it was missing all of its screw terminals and had pads that were already soldered shut, which made it very difficult to connect the battery and motor wires reliably and safely.</p>
  <p>The problem that took about two weeks to solve was supplying power to the motors. The board would not power on through the +12V input while I was using the L298N. I tried 6V, 7.5V, and 9V battery packs, and nothing worked. I had to systematically test each part — the battery holder, the batteries, the motors, the Pi, and the wiring — to isolate exactly where the problem was.</p>
  <p>Another issue was unreliable connections, which was the hardest part because everything <em>looked</em> correct, but I eventually realized some wires weren't making solid contact with the Raspberry Pi or the breadboard. I was also missing a few connections because I was progressing too quickly.</p>
  <p>After all of these problems, there was one simple fix that would have saved a lot of time: switching to the L9110 motor driver. The L9110 uses a single power input instead of separate logic and motor inputs, which makes everything much simpler — and once I switched, the motors finally spun.</p>

  <div class="label">Motor Driver Code</div>
  <p>This code uses basic WASD controls to move the robot, which is useful for testing the most basic mechanics of the motors. It also confirms that the wiring to the motor driver and the Raspberry Pi is correct. The HIGH/LOW combinations create different patterns, which cause the changes in direction.</p>
  <div class="code-wrap">
<div class="code-bar"><span class="dot d1"></span><span class="dot d2"></span><span class="dot d3"></span><span class="fname">motor_test.py</span></div>
<pre><code>import RPi.GPIO as GPIO
GPIO.setmode(GPIO.BCM)

A1A = 6; A1B = 5; B1A = 22; B2A = 23
for p in [A1A, A1B, B1A, B2A]:
    GPIO.setup(p, GPIO.OUT)

def forward():
    GPIO.output(A1A, GPIO.LOW); GPIO.output(A1B, GPIO.HIGH)
    GPIO.output(B1A, GPIO.LOW); GPIO.output(B2A, GPIO.HIGH)
def backward():
    GPIO.output(A1A, GPIO.HIGH); GPIO.output(A1B, GPIO.LOW)
    GPIO.output(B1A, GPIO.HIGH); GPIO.output(B2A, GPIO.LOW)
def left():
    GPIO.output(A1A, GPIO.LOW); GPIO.output(A1B, GPIO.LOW)
    GPIO.output(B1A, GPIO.LOW); GPIO.output(B2A, GPIO.HIGH)
def right():
    GPIO.output(A1A, GPIO.LOW); GPIO.output(A1B, GPIO.HIGH)
    GPIO.output(B1A, GPIO.LOW); GPIO.output(B2A, GPIO.LOW)
def stop():
    for p in [A1A, A1B, B1A, B2A]: GPIO.output(p, GPIO.LOW)

print(&quot;w=forward s=back a=left d=right x=stop q=quit&quot;)
try:
    while True:
        u = input(&quot;move: &quot;)
        if u == &#x27;w&#x27;: forward()
        elif u == &#x27;s&#x27;: backward()
        elif u == &#x27;a&#x27;: left()
        elif u == &#x27;d&#x27;: right()
        elif u == &#x27;x&#x27;: stop()
        elif u == &#x27;q&#x27;: break
except KeyboardInterrupt:
    pass
GPIO.cleanup()</code></pre>
</div>

  <div class="label">Ultrasonic Sensor Code</div>
  <p>This code tests whether the ultrasonic sensors work. It measures the distance to an object by firing a pulse from each sensor, timing how long the echo takes to return, and using the speed of sound to calculate the distance. It reads all three sensors — left, center, and right — and prints their distances.</p>
  <div class="code-wrap">
<div class="code-bar"><span class="dot d1"></span><span class="dot d2"></span><span class="dot d3"></span><span class="fname">ultrasonic_test.py</span></div>
<pre><code>import RPi.GPIO as GPIO
import time
GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

# All 3 sensors: (name, TRIG, ECHO)
sensors = [
    (&quot;LEFT&quot;,   19, 26),
    (&quot;CENTER&quot;, 16, 20),
    (&quot;RIGHT&quot;,  11, 12),
]

for name, trig, echo in sensors:
    GPIO.setup(trig, GPIO.OUT)
    GPIO.setup(echo, GPIO.IN)
    GPIO.output(trig, False)

time.sleep(2)

def measure(trig, echo):
    start = time.time()
    stop = time.time()
    GPIO.output(trig, True)
    time.sleep(0.00001)
    GPIO.output(trig, False)
    timeout = time.time() + 0.05
    while GPIO.input(echo) == 0 and time.time() &lt; timeout:
        start = time.time()
    timeout = time.time() + 0.05
    while GPIO.input(echo) == 1 and time.time() &lt; timeout:
        stop = time.time()
    return round((stop - start) * 34300 / 2, 1)

try:
    while True:
        for name, trig, echo in sensors:
            d = measure(trig, echo)
            print(name, &quot;:&quot;, d, &quot;cm&quot;)
        print(&quot;-----&quot;)
        time.sleep(0.5)
except KeyboardInterrupt:
    GPIO.cleanup()</code></pre>
</div>

  <div class="label">What's Next</div>
  <p>Connect the camera to the Raspberry Pi and add code so it can track the ball. I will write the OpenCV code so the robot can detect the red ball, then combine the camera, sensors, and motors so it can track the ball and avoid obstacles in its path.</p>

  <!-- ============ FIRST MILESTONE ============ -->
  <h1 class="section" id="m1"><span class="idx">MILESTONE 01</span>First Milestone</h1>
  <div class="video"><iframe src="https://www.youtube.com/embed/zEN702sMDmo" title="Vaideesh K. Milestone 1" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe></div>

  <div class="label">Summary</div>
  <p>The goal of the first milestone was to build the foundation of the ball-tracking robot: assembling the robot chassis, getting the Raspberry Pi running with the necessary software installed, and completing all of the electronic connections (wiring the two motors to the L9110 motor driver, the three ultrasonic sensors, and the power). The Pi drives the motors, and the camera is added so the robot can track the red ball and follow it.</p>

  <div class="label">Components Used</div>
  <ul>
    <li><strong>Raspberry Pi 4 Model B</strong> — the brain behind everything. It runs the code and controls the robot, from the sensors to the motors.</li>
    <li><strong>PiCamera (OV5647)</strong> — the camera that detects the ball (used in later milestones).</li>
    <li><strong>L9110 Motor Driver</strong> — an H-bridge board that lets the Pi control the motors. I switched to this after using the L298N.</li>
    <li><strong>2× Yellow TT Motors</strong> — spin the wheels to move the robot.</li>
    <li><strong>2× Wheels + Front Caster</strong> — the wheels drive the robot, and the caster helps balance it and mount the front sensors.</li>
    <li><strong>3× HC-SR04 Ultrasonic Sensors</strong> — measure distance for obstacle detection.</li>
    <li><strong>Resistors (1kΩ and 2kΩ)</strong> — build the voltage dividers that protect the Pi's 3.3V pins from the sensors' 5V signals.</li>
    <li><strong>Breadboard</strong> — the main hub for connecting the voltage dividers, power, and sensor wiring.</li>
    <li><strong>Jumper Wires</strong> — connect all the components together.</li>
    <li><strong>4×AA Battery Pack (6V)</strong> — powers the motors.</li>
    <li><strong>USB-C Powerbank</strong> — powers the Pi.</li>
    <li><strong>Clear Acrylic 2WD Chassis</strong> — the frame that holds everything together.</li>
  </ul>

  <div class="label">Challenges</div>
  <p>The challenges I faced were finding diagrams to help me make the electrical connections and making all of the connections myself. There were nearly 40 connections to make, and the wires kept getting tangled, the resistors kept getting unplugged, and everything was disorganized. I had to reseat and redo cables multiple times, but in the end I got them organized. I also ran into a problem where some cables were dead, so I used a multimeter to find and replace them.</p>

  <div class="label">What's Next</div>
  <p>Make the motors work when connected to the motor driver, and make the ultrasonic sensors detect the distance of an object placed in front of them.</p>

  <!-- ============ SCHEMATICS ============ -->
  <h1 class="section" id="schematics"><span class="idx">REFERENCE</span>Schematics</h1>
  <div class="label">Ball Tracking Robot Diagram</div>
  <p>This is the complete wiring diagram for the robot. Every Raspberry Pi connection is labelled
    by physical board pin number with the BCM GPIO number in brackets, and the three Echo voltage
    dividers are drawn hole by hole on the breadboard. A filled dot marks a real electrical
    junction; a small arc means two wires cross without touching.</p>
  <figure class="wide">
    <a href="Screenshot%202026-07-23%20012431.png" target="_blank" rel="noopener" data-lock>
      <img src="Screenshot%202026-07-23%20012431.png"
           alt="Complete wiring schematic: Raspberry Pi 4B, three HC-SR04 sensors on 1k and 2k echo voltage dividers, L298N motor driver, two DC motors, 6V battery pack and a separate USB-C power bank.">
    </a>
    <figcaption>Click to open the full-size version</figcaption>
  </figure>
  <div class="label">Robot Build Photos</div>
  <div class="imgrow">
    <img src="IMG_6778.jpeg" alt="Build photo 1" style="width:250px">
    <img src="IMG_6781.jpeg" alt="Build photo 2" style="width:250px">
    <img src="IMG_6782.jpeg" alt="Build photo 3" style="width:250px">
  </div>

  <!-- ============ BILL OF MATERIALS ============ -->
  <h1 class="section" id="bom"><span class="idx">REFERENCE</span>Bill of Materials</h1>
  <table>
    <thead><tr><th>Part</th><th>Note</th><th>Price</th><th>Link</th></tr></thead>
    <tbody>
      <tr><td>Raspberry Pi 4 Model B</td><td>Small computer used for controlling the robot and writing code</td><td>$79.97</td><td><a href="https://www.amazon.com/Raspberry-Model-2019-Quad-Bluetooth/dp/B07TC2BK1X">Link</a></td></tr>
      <tr><td>Raspberry Pi Camera Module</td><td>Camera used for live video and seeing the ball</td><td>$14.99</td><td><a href="https://www.amazon.com/Arducam-Autofocus-Raspberry-Motorized-Software/dp/B07SN8GYGD">Link</a></td></tr>
      <tr><td>L298N Driver Board</td><td>A basic motor driver used to drive the wheels forward and backward</td><td>$8.99</td><td><a href="https://www.amazon.com/Qunqi-2Packs-Controller-Stepper-Arduino/dp/B01M29YK5U">Link</a></td></tr>
      <tr><td>Motors and Board Kit</td><td>Basic hardware pieces that help with the assembly of the robot</td><td>$13.59</td><td><a href="https://www.amazon.com/Smart-Chassis-Motors-Encoder-Battery/dp/B01LXY7CM3">Link</a></td></tr>
      <tr><td>Powerbank</td><td>Supplies power to the Raspberry Pi 4</td><td>$21.98</td><td><a href="https://www.amazon.com/Anker-Ultra-Compact-High-Speed-VoltageBoost-Technology/dp/B07QXV6N1B">Link</a></td></tr>
      <tr><td>HC-SR04 Sensors (5 pcs)</td><td>Used for distance calculations of objects and obstacles</td><td>$8.99</td><td><a href="https://www.amazon.com/Organizer-Ultrasonic-Distance-MEGA2560-ElecRight/dp/B07RGB4W8V">Link</a></td></tr>
      <tr><td>HDMI to Micro HDMI Cable</td><td>Connects the Raspberry Pi 4 to a laptop for the OBS video stream</td><td>$8.99</td><td><a href="https://www.amazon.com/UGREEN-Adapter-Ethernet-Compatible-Raspberry/dp/B06WWQ7KLV">Link</a></td></tr>
      <tr><td>Video Capture Card</td><td>Necessary to display the Pi's output on laptops</td><td>$16.99</td><td><a href="https://www.amazon.com/Capture-Streaming-Broadcasting-Conference-Teaching/dp/B09FLN63B3">Link</a></td></tr>
      <tr><td>SD Card Reader</td><td>Necessary to flash the microSD card and install an OS</td><td>$4.99</td><td><a href="https://www.amazon.com/Reader-Adapter-Camera-Memory-Wansurs/dp/B0B9QZ4W4Y">Link</a></td></tr>
      <tr><td>Wired Mouse and Keyboard</td><td>Needed to operate the Raspberry Pi 4</td><td>$25.99</td><td><a href="https://www.amazon.com/Wireless-Keyboard-Trueque-Cordless-Computer/dp/B09J4RQFK7">Link</a></td></tr>
      <tr><td>Basic Connections Components Kit</td><td>Includes male-to-male, female-to-female, and male-to-female jumper wires, resistors, and LEDs</td><td>$11.47</td><td><a href="https://www.amazon.com/Smraza-Breadboard-Resistors-Mega2560-Raspberry/dp/B01HRR7EBG">Link</a></td></tr>
      <tr><td>Soldering Kit</td><td>Used for the motor connections</td><td>$13.60</td><td><a href="https://www.amazon.com/Soldering-Interchangeable-Adjustable-Temperature-Enthusiast/dp/B087767KNW">Link</a></td></tr>
    </tbody>
  </table>

  <!-- ============ STARTER PROJECT ============ -->
  <h1 class="section" id="starter"><span class="idx">STARTER PROJECT</span>Retro Arcade</h1>
  <div class="video"><iframe src="https://www.youtube.com/embed/BF2v-AP0EPY" title="Retro Arcade Starter Project" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe></div>

  <div class="label">Summary</div>
  <p>My starter project was the Retro Arcade Console. It works by receiving input from the buttons on the front display, which is processed through the integrated circuits and displayed on an LCD screen. It also includes a buzzer for sound effects and runs on batteries. After I soldered all of the electronic components onto the circuit board, I could play games like Tetris and Snake, with the system keeping track of the player's score. This project taught me how to solder electronics to circuit boards, identify electronic components, and troubleshoot electrical connections. Building the Retro Arcade Console helped prepare me for my main project, the Ball Tracking Robot.</p>

  <div class="label">Components Used</div>
  <ul>
    <li>Microcontroller</li>
    <li>Small LCD Screen</li>
    <li>Buzzer</li>
    <li>Capacitors</li>
    <li>4×AA Battery Holder / AA Batteries</li>
    <li>PCB (Circuit Board)</li>
    <li>Solder Kit</li>
    <li>Header Pins</li>
    <li>Acrylic Case</li>
    <li>Variety of Colored Buttons</li>
  </ul>

  <div class="label">Challenges Faced</div>
  <p>There were several challenges, some harder than others. My first challenge was soldering the board — every time I soldered, the solder kept bridging to other holes, which could cause a short circuit and damage the board. Another problem was getting the red and black battery wires to sit neatly in two tiny holes and holding them in place so I could solder them properly. I had to unsolder many parts multiple times because the console simply would not turn on. But after all of these hardships, I managed to fix every one of them and get the console working properly.</p>

  <!-- ============ RESOURCES ============ -->
  <h1 class="section" id="resources"><span class="idx">REFERENCE</span>Resources</h1>
  <p>These are the references and documentation I used while building the robot. The OpenCV and picamera2 docs were the most useful for the vision code, and the GPIO documentation was what I kept going back to while wiring the motors and sensors.</p>

  <div class="label">Documentation</div>
  <ul>
    <li><a class="link" href="https://docs.opencv.org/4.x/">OpenCV Documentation</a> — the main reference for the vision code: HSV color filtering, contours, and morphological operations.</li>
    <li><a class="link" href="https://datasheets.raspberrypi.com/camera/picamera2-manual.pdf">Picamera2 Manual</a> — how to configure and capture frames from the Raspberry Pi Camera in Python.</li>
    <li><a class="link" href="https://sourceforge.net/p/raspberry-gpio-python/wiki/Home/">RPi.GPIO Documentation</a> — controlling the GPIO pins and using PWM for the motors and servo.</li>
    <li><a class="link" href="https://www.raspberrypi.com/documentation/">Raspberry Pi Documentation</a> — general setup, flashing the OS, and enabling the camera.</li>
    <li><a class="link" href="https://flask.palletsprojects.com/">Flask Documentation</a> — used to build the web dashboard and stream video to a browser.</li>
  </ul>

  <div class="label">Guides I Used</div>
  <ul>
    <li><a class="link" href="https://docs.opencv.org/4.x/df/d9d/tutorial_py_colorspaces.html">OpenCV: Changing Colorspaces</a> — explains BGR to HSV conversion and why HSV is better for color tracking.</li>
    <li><a class="link" href="https://docs.opencv.org/4.x/d9/d61/tutorial_py_morphological_ops.html">OpenCV: Morphological Transformations</a> — how opening and closing clean up a mask, which is what fixed the glare holes in the ball.</li>
    <li><a class="link" href="https://docs.opencv.org/4.x/dd/d49/tutorial_py_contour_features.html">OpenCV: Contour Features</a> — area, perimeter, and minimum enclosing circle, which I used for the roundness check.</li>
    <li><a class="link" href="https://projects.raspberrypi.org/en/projects/physical-computing">Raspberry Pi Physical Computing</a> — basics of wiring components to the GPIO pins.</li>
  </ul>

  <div class="label">Components</div>
  <ul>
    <li><a class="link" href="https://components101.com/sensors/ultrasonic-sensor-working-pinout-datasheet">HC-SR04 Datasheet</a> — pinout and timing for the ultrasonic sensors.</li>
    <li><a class="link" href="https://components101.com/modules/l9110-2-channel-motor-driver-module">L9110 Motor Driver</a> — pinout and wiring for the H-bridge that drives the motors.</li>
  </ul>

  <div class="footer">
    <span><span class="sig">Vaideesh K</span> · Ball Tracking Robot with OpenCV</span>
    <span>Cupertino High School · Electrical Engineering</span>
    <a class="top" href="#">Back to top ↑</a>
  </div>

</main>

<script>
(function () {
  var reduce = window.matchMedia &&
    window.matchMedia('(prefers-reduced-motion: reduce)').matches;

  /* ==================================================================
     BOOT — lines tick in, rail fills, then the page arrives
     ================================================================== */
  (function boot() {
    var el = document.getElementById('boot');
    var rail = document.getElementById('bootRail');
    if (!el || reduce) {
      if (el) el.classList.add('gone');
      document.body.classList.add('ready');
      return;
    }
    var lines = document.querySelectorAll('[data-l]');
    lines.forEach(function (l, i) {
      setTimeout(function () { l.classList.add('up'); }, 90 + i * 155);
    });
    setTimeout(function () { rail.style.width = '100%'; }, 120);
    setTimeout(function () {
      el.classList.add('gone');
      document.body.classList.add('ready');
    }, 1500);
  })();

  /* ==================================================================
     RETICLE CURSOR — a detection box that follows the pointer
     ================================================================== */
  (function reticle() {
    if (reduce) return;
    if (!window.matchMedia || !window.matchMedia('(hover:hover)').matches) return;
    var r = document.getElementById('retic');
    if (!r) return;
    var tx = 0, ty = 0, x = 0, y = 0, on = false;

    window.addEventListener('pointermove', function (e) {
      tx = e.clientX; ty = e.clientY;
      if (!on) { on = true; x = tx; y = ty; r.classList.add('live'); }
      var hit = e.target.closest(
        'a,button,input,.code-wrap,.imgrow img,.scope,.lab-stage,[data-lock]');
      r.classList.toggle('lock', !!hit);
    });
    window.addEventListener('pointerleave', function () {
      on = false; r.classList.remove('live');
    });

    (function loop() {
      x += (tx - x) * 0.22;
      y += (ty - y) * 0.22;
      var s = r.classList.contains('lock') ? 22 : 13;
      r.style.transform = 'translate(' + (x - s) + 'px,' + (y - s) + 'px)';
      requestAnimationFrame(loop);
    })();
  })();

  /* ==================================================================
     HERO TRACKER — the robot's loop, running in the browser
     ================================================================== */
  (function tracker() {
    var cv = document.getElementById('scopeCanvas');
    if (!cv || !cv.getContext) return;
    var g = cv.getContext('2d');
    var W = cv.width, H = cv.height, CX = W / 2, DEAD = 62;
    var maskView = false;

    var ball = { x: W * 0.3, y: H * 0.44, vx: 1.9, vy: 1.2, r: 38 };
    var est = { x: ball.x, y: ball.y, r: ball.r };
    var servo = 90, t = 0, raf = null;

    var elStatus = document.getElementById('fStatus'),
        elErr = document.getElementById('fErr'),
        elRad = document.getElementById('fRad'),
        elServo = document.getElementById('fServo');

    var bCam = document.getElementById('btnCam'),
        bMask = document.getElementById('btnMask');
    function setView(m) {
      maskView = m;
      bCam.classList.toggle('on', !m);
      bMask.classList.toggle('on', m);
    }
    bCam.addEventListener('click', function () { setView(false); });
    bMask.addEventListener('click', function () { setView(true); });

    cv.addEventListener('pointermove', function (e) {
      var b = cv.getBoundingClientRect();
      var mx = (e.clientX - b.left) * (W / b.width);
      var my = (e.clientY - b.top) * (H / b.height);
      var dx = ball.x - mx, dy = ball.y - my;
      var d = Math.sqrt(dx * dx + dy * dy);
      if (d < 135 && d > 0.1) {
        ball.vx += (dx / d) * 0.6;
        ball.vy += (dy / d) * 0.6;
      }
    });

    function step() {
      t += 0.016;
      ball.x += ball.vx; ball.y += ball.vy;
      ball.vx += Math.sin(t * 0.7) * 0.016;
      ball.vy += Math.cos(t * 0.53) * 0.013;

      var pad = ball.r + 12;
      if (ball.x < pad) { ball.x = pad; ball.vx = Math.abs(ball.vx); }
      if (ball.x > W - pad) { ball.x = W - pad; ball.vx = -Math.abs(ball.vx); }
      if (ball.y < pad) { ball.y = pad; ball.vy = Math.abs(ball.vy); }
      if (ball.y > H - pad - 22) { ball.y = H - pad - 22; ball.vy = -Math.abs(ball.vy); }

      var sp = Math.sqrt(ball.vx * ball.vx + ball.vy * ball.vy);
      if (sp > 3.0) { ball.vx *= 3.0 / sp; ball.vy *= 3.0 / sp; }
      if (sp < 1.0) { ball.vx *= 1.03; ball.vy *= 1.03; }
      ball.r = 33 + Math.sin(t * 0.42) * 8;

      /* the same 0.6 / 0.4 smoothing the robot runs */
      est.x = est.x * 0.6 + ball.x * 0.4;
      est.y = est.y * 0.6 + ball.y * 0.4;
      est.r = est.r * 0.6 + ball.r * 0.4;

      var err = est.x - CX;
      if (Math.abs(err) > DEAD) {
        var d = Math.max(-1.6, Math.min(1.6, 0.02 * err));
        servo = Math.max(20, Math.min(160, servo + d));
      }
      draw(err);
      raf = requestAnimationFrame(step);
    }

    function draw(err) {
      var locked = Math.abs(err) <= DEAD;

      if (maskView) {
        g.fillStyle = '#000'; g.fillRect(0, 0, W, H);
      } else {
        g.fillStyle = '#04070A'; g.fillRect(0, 0, W, H);
        g.strokeStyle = 'rgba(41,224,126,.05)'; g.lineWidth = 1;
        for (var x = 0; x <= W; x += 40) {
          g.beginPath(); g.moveTo(x + .5, 0); g.lineTo(x + .5, H); g.stroke();
        }
        for (var y = 0; y <= H; y += 40) {
          g.beginPath(); g.moveTo(0, y + .5); g.lineTo(W, y + .5); g.stroke();
        }
      }

      if (maskView) {
        g.fillStyle = '#fff';
        g.beginPath(); g.arc(ball.x, ball.y, ball.r, 0, 6.2832); g.fill();
        g.fillStyle = '#000';
        g.beginPath();
        g.arc(ball.x - ball.r * .3, ball.y - ball.r * .34, ball.r * .19, 0, 6.2832); g.fill();
        g.beginPath();
        g.arc(ball.x + ball.r * .12, ball.y - ball.r * .46, ball.r * .1, 0, 6.2832); g.fill();
        g.fillStyle = '#fff';
        g.beginPath(); g.arc(ball.x + ball.r * 1.7, ball.y - ball.r * 1.2, 4, 0, 6.2832); g.fill();
        g.beginPath(); g.arc(ball.x - ball.r * 1.9, ball.y + ball.r * 1.1, 2.6, 0, 6.2832); g.fill();
      } else {
        var gr = g.createRadialGradient(
          ball.x - ball.r * .34, ball.y - ball.r * .38, ball.r * .1,
          ball.x, ball.y, ball.r);
        gr.addColorStop(0, '#FF8A7E');
        gr.addColorStop(.45, '#FF3B2F');
        gr.addColorStop(1, '#B21B12');
        g.fillStyle = gr;
        g.beginPath(); g.arc(ball.x, ball.y, ball.r, 0, 6.2832); g.fill();
        g.fillStyle = 'rgba(255,255,255,.5)';
        g.beginPath();
        g.arc(ball.x - ball.r * .33, ball.y - ball.r * .36, ball.r * .17, 0, 6.2832); g.fill();
      }

      g.strokeStyle = 'rgba(120,140,150,.26)';
      g.setLineDash([5, 6]); g.lineWidth = 1;
      g.beginPath(); g.moveTo(CX - DEAD, 0); g.lineTo(CX - DEAD, H - 30); g.stroke();
      g.beginPath(); g.moveTo(CX + DEAD, 0); g.lineTo(CX + DEAD, H - 30); g.stroke();
      g.setLineDash([]);

      g.strokeStyle = 'rgba(41,224,126,.5)'; g.lineWidth = 1;
      g.beginPath(); g.moveTo(CX, H / 2 - 13); g.lineTo(CX, H / 2 + 13); g.stroke();
      g.beginPath(); g.moveTo(CX - 13, H / 2); g.lineTo(CX + 13, H / 2); g.stroke();

      var col = locked ? '#29E07E' : '#F5B93B';
      var bx = est.x - est.r - 13, by = est.y - est.r - 13;
      var bw = (est.r + 13) * 2, bh = (est.r + 13) * 2, c = 17;
      g.strokeStyle = col; g.lineWidth = 2.4; g.lineCap = 'square';
      g.beginPath();
      g.moveTo(bx, by + c); g.lineTo(bx, by); g.lineTo(bx + c, by);
      g.moveTo(bx + bw - c, by); g.lineTo(bx + bw, by); g.lineTo(bx + bw, by + c);
      g.moveTo(bx + bw, by + bh - c); g.lineTo(bx + bw, by + bh); g.lineTo(bx + bw - c, by + bh);
      g.moveTo(bx + c, by + bh); g.lineTo(bx, by + bh); g.lineTo(bx, by + bh - c);
      g.stroke();

      g.fillStyle = col;
      g.beginPath(); g.arc(est.x, est.y, 3, 0, 6.2832); g.fill();

      g.strokeStyle = locked ? 'rgba(41,224,126,.45)' : 'rgba(245,185,59,.65)';
      g.lineWidth = 1.4; g.setLineDash([3, 4]);
      g.beginPath(); g.moveTo(CX, est.y); g.lineTo(est.x, est.y); g.stroke();
      g.setLineDash([]);

      var tag = locked ? 'BALL · LOCKED' : 'BALL · TRACKING';
      g.font = '600 11px JetBrains Mono, monospace';
      g.fillStyle = col;
      g.fillRect(bx, by - 21, g.measureText(tag).width + 14, 16);
      g.fillStyle = '#04070A';
      g.fillText(tag, bx + 7, by - 9);

      var pct = (servo - 20) / 140;
      g.fillStyle = 'rgba(255,255,255,.07)';
      g.fillRect(26, H - 18, W - 52, 3);
      g.fillStyle = '#29E07E';
      g.fillRect(26 + (W - 52) * pct - 13, H - 21, 26, 9);
      g.font = '9px JetBrains Mono, monospace';
      g.fillStyle = '#546268';
      g.fillText('SERVO 20°', 26, H - 26);
      g.textAlign = 'right';
      g.fillText('160°', W - 26, H - 26);
      g.textAlign = 'left';

      elStatus.textContent = locked ? 'LOCKED' : 'TRACKING';
      elStatus.className = locked ? 'v' : 'v warn';
      elErr.textContent = Math.round(err) + ' px';
      elRad.textContent = Math.round(est.r) + ' px';
      elServo.textContent = Math.round(servo) + '\u00B0';
    }

    if (reduce) { draw(est.x - CX); return; }
    if ('IntersectionObserver' in window) {
      new IntersectionObserver(function (en) {
        en.forEach(function (e) {
          if (e.isIntersecting && !raf) raf = requestAnimationFrame(step);
          else if (!e.isIntersecting && raf) { cancelAnimationFrame(raf); raf = null; }
        });
      }, { threshold: 0 }).observe(cv);
    } else { raf = requestAnimationFrame(step); }
  })();

  /* ==================================================================
     THE LAB — real HSV thresholding, live
     Four objects with real-ish HSV values. Change the filter, watch
     which ones survive it.
     ================================================================== */
  (function lab() {
    var cv = document.getElementById('labCanvas');
    if (!cv || !cv.getContext) return;
    var g = cv.getContext('2d');
    var W = cv.width, H = cv.height;

    /* h/s/v are approximate OpenCV values. fill = how round the blob is. */
    var objs = [
      { id: 'rBall',  name: 'red ball',  h: 3,   s: 232, v: 205, fill: .93,
        x: 150, y: 178, r: 62, css: '#FF3B2F', shape: 'circle' },
      { id: 'rFace',  name: 'skin tone', h: 8,   s: 118, v: 196, fill: .74,
        x: 330, y: 150, r: 52, css: '#D89A78', shape: 'face' },
      { id: 'rMug',   name: 'red mug',   h: 2,   s: 214, v: 168, fill: .58,
        x: 500, y: 200, r: 48, css: '#D6342A', shape: 'mug' },
      { id: 'rOrange',name: 'orange',    h: 16,  s: 236, v: 226, fill: .90,
        x: 648, y: 150, r: 42, css: '#FF9426', shape: 'circle' }
    ];

    var cfg = { hue: 10, sat: 170, val: 80, fill: 40 };
    var DEF = { hue: 10, sat: 170, val: 80, fill: 40 };

    var el = {
      hue: document.getElementById('sHue'), sat: document.getElementById('sSat'),
      val: document.getElementById('sVal'), fill: document.getElementById('sFill'),
      vHue: document.getElementById('vHue'), vSat: document.getElementById('vSat'),
      vVal: document.getElementById('vVal'), vFill: document.getElementById('vFill'),
      verdict: document.getElementById('labVerdict')
    };

    /* red wraps the hue circle at 0/180, so measure the shorter way round */
    function hueGap(h) {
      var d = Math.abs(h - 0);
      return Math.min(d, 180 - d);
    }

    function passes(o) {
      var colour = hueGap(o.h) <= cfg.hue && o.s >= cfg.sat && o.v >= cfg.val;
      var round = o.fill >= cfg.fill / 100;
      return { colour: colour, round: round, ok: colour && round };
    }

    function paint() {
      g.fillStyle = '#000'; g.fillRect(0, 0, W, H);

      var passing = 0;

      objs.forEach(function (o) {
        var p = passes(o);
        if (p.ok) passing++;

        /* left half of each object: what the camera sees.
           the mask beneath it: what survives the filter. */
        if (p.colour) {
          g.fillStyle = '#fff';
          blob(o, false);
          /* glare hole, same thing that broke the real roundness maths */
          g.fillStyle = '#000';
          g.beginPath();
          g.arc(o.x - o.r * .3, o.y - o.r * .32, o.r * .16, 0, 6.2832);
          g.fill();
        } else {
          /* filtered out - draw a faint ghost so you can see what you lost */
          g.fillStyle = 'rgba(255,255,255,.055)';
          blob(o, false);
        }

        /* verdict bracket */
        var col = p.ok ? '#29E07E' : (p.colour ? '#F5B93B' : 'rgba(120,135,142,.4)');
        var bx = o.x - o.r - 14, by = o.y - o.r - 14;
        var bw = (o.r + 14) * 2, bh = (o.r + 14) * 2, c = 14;
        g.strokeStyle = col; g.lineWidth = 2; g.lineCap = 'square';
        g.beginPath();
        g.moveTo(bx, by + c); g.lineTo(bx, by); g.lineTo(bx + c, by);
        g.moveTo(bx + bw - c, by); g.lineTo(bx + bw, by); g.lineTo(bx + bw, by + c);
        g.moveTo(bx + bw, by + bh - c); g.lineTo(bx + bw, by + bh); g.lineTo(bx + bw - c, by + bh);
        g.moveTo(bx + c, by + bh); g.lineTo(bx, by + bh); g.lineTo(bx, by + bh - c);
        g.stroke();

        /* label */
        var msg = p.ok ? 'TRACKED'
                : p.colour ? 'not round enough'
                : 'filtered out';
        g.font = '500 10.5px JetBrains Mono, monospace';
        g.fillStyle = col;
        g.fillText(o.name.toUpperCase(), bx, by - 20);
        g.fillStyle = p.ok ? '#29E07E' : 'rgba(140,155,162,.75)';
        g.font = '10px JetBrains Mono, monospace';
        g.fillText(msg, bx, by - 7);

        /* hsv readout under each */
        g.fillStyle = 'rgba(120,135,142,.5)';
        g.font = '9.5px JetBrains Mono, monospace';
        g.fillText('H' + o.h + ' S' + o.s + ' V' + o.v + '  fill ' + o.fill.toFixed(2),
          bx, by + bh + 16);

        /* update the side panel */
        var out = document.getElementById(o.id);
        if (out) {
          out.textContent = p.ok ? 'TRACKED' : (p.colour ? 'passes colour' : 'rejected');
          out.className = p.ok ? (o.id === 'rBall' ? 'g' : 'r')
                               : (o.id === 'rBall' ? 'r' : 'g');
        }
      });

      if (el.verdict) {
        el.verdict.textContent = passing === 1 ? '1 object passes'
          : passing + ' objects pass';
        el.verdict.style.color = passing === 1 ? '#29E07E' : '#F5B93B';
      }
    }

    function blob(o, outline) {
      g.beginPath();
      if (o.shape === 'circle') {
        g.arc(o.x, o.y, o.r, 0, 6.2832);
      } else if (o.shape === 'face') {
        g.ellipse(o.x, o.y, o.r * .78, o.r, 0, 0, 6.2832);
      } else {
        /* mug: a rounded body with a handle - deliberately not round */
        g.moveTo(o.x - o.r * .7, o.y - o.r);
        g.lineTo(o.x + o.r * .55, o.y - o.r);
        g.lineTo(o.x + o.r * .55, o.y + o.r);
        g.lineTo(o.x - o.r * .7, o.y + o.r);
        g.closePath();
        g.moveTo(o.x + o.r * .55, o.y - o.r * .4);
        g.arc(o.x + o.r * .62, o.y, o.r * .42, -1.2, 1.2);
      }
      g.fill();
    }

    function sync() {
      cfg.hue = +el.hue.value; cfg.sat = +el.sat.value;
      cfg.val = +el.val.value; cfg.fill = +el.fill.value;
      el.vHue.textContent = '\u00B1' + cfg.hue;
      el.vSat.textContent = cfg.sat;
      el.vVal.textContent = cfg.val;
      el.vFill.textContent = (cfg.fill / 100).toFixed(2);
      paint();
    }

    ['hue', 'sat', 'val', 'fill'].forEach(function (k) {
      el[k].addEventListener('input', sync);
    });

    var rst = document.getElementById('resetLab');
    if (rst) rst.addEventListener('click', function () {
      el.hue.value = DEF.hue; el.sat.value = DEF.sat;
      el.val.value = DEF.val; el.fill.value = DEF.fill;
      sync();
    });

    sync();
  })();

  /* ==================================================================
     COUNTERS
     ================================================================== */
  (function counters() {
    var nums = document.querySelectorAll('[data-count]');
    if (!nums.length) return;
    if (reduce || !('IntersectionObserver' in window)) {
      Array.prototype.forEach.call(nums, function (n) {
        n.textContent = n.getAttribute('data-count');
      });
      return;
    }
    var io = new IntersectionObserver(function (en) {
      en.forEach(function (e) {
        if (!e.isIntersecting) return;
        var n = e.target, end = +n.getAttribute('data-count'), s = null;
        function tick(ts) {
          if (!s) s = ts;
          var p = Math.min((ts - s) / 1100, 1);
          var eased = 1 - Math.pow(1 - p, 3);
          n.textContent = Math.round(end * eased);
          if (p < 1) requestAnimationFrame(tick);
        }
        requestAnimationFrame(tick);
        io.unobserve(n);
      });
    }, { threshold: .5 });
    Array.prototype.forEach.call(nums, function (n) { io.observe(n); });
  })();

  /* ==================================================================
     PROGRESS + STICKY NAV + ACTIVE SECTION + REVEALS
     ================================================================== */
  var bar = document.getElementById('progress');
  var stick = document.getElementById('stick');
  var hero = document.querySelector('.hero');

  function onScroll() {
    var h = document.documentElement.scrollHeight - window.innerHeight;
    bar.style.width = (h > 0 ? (window.scrollY / h) * 100 : 0) + '%';
    stick.classList.toggle('on', window.scrollY > hero.offsetHeight - 60);
  }
  window.addEventListener('scroll', onScroll, { passive: true });
  onScroll();

  var links = Array.prototype.slice.call(document.querySelectorAll('.stick-links a'));
  var targets = links
    .map(function (a) { return document.querySelector(a.getAttribute('href')); })
    .filter(Boolean);

  if ('IntersectionObserver' in window) {
    var spy = new IntersectionObserver(function (en) {
      en.forEach(function (e) {
        if (!e.isIntersecting) return;
        links.forEach(function (a) {
          a.classList.toggle('here', a.getAttribute('href') === '#' + e.target.id);
        });
      });
    }, { rootMargin: '-14% 0px -72% 0px' });
    targets.forEach(function (x) { spy.observe(x); });

    var blocks = document.querySelectorAll(
      '.main h1.section, .main .label, .main p, .main ul, .main table,' +
      '.main .code-wrap, .main .video, .main .imgrow, .main .single-img, .main .headshot, .main figure');
    if (!reduce) {
      Array.prototype.forEach.call(blocks, function (el) { el.classList.add('rise'); });
    }
    var reveal = new IntersectionObserver(function (en) {
      en.forEach(function (e) {
        if (e.isIntersecting) { e.target.classList.add('seen'); reveal.unobserve(e.target); }
      });
    }, { rootMargin: '0px 0px -7% 0px', threshold: .04 });
    Array.prototype.forEach.call(blocks, function (el) { reveal.observe(el); });
  }

  /* ==================================================================
     COPY BUTTONS
     ================================================================== */
  Array.prototype.forEach.call(document.querySelectorAll('.code-wrap'), function (wrap) {
    var pre = wrap.querySelector('pre'), cbar = wrap.querySelector('.code-bar');
    if (!pre || !cbar) return;
    var btn = document.createElement('button');
    btn.className = 'copy'; btn.type = 'button'; btn.textContent = 'COPY';
    btn.setAttribute('data-lock', '');
    btn.addEventListener('click', function () {
      function ok() {
        btn.textContent = 'COPIED'; btn.classList.add('done');
        setTimeout(function () {
          btn.textContent = 'COPY'; btn.classList.remove('done');
        }, 1600);
      }
      if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(pre.innerText).then(ok, function () {});
      } else {
        var ta = document.createElement('textarea');
        ta.value = pre.innerText;
        document.body.appendChild(ta); ta.select();
        try { document.execCommand('copy'); ok(); } catch (err) {}
        document.body.removeChild(ta);
      }
    });
    cbar.appendChild(btn);
  });
})();
</script>

</body>
</html>
