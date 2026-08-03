---
title: Music
date: 2026-08-03 00:00:00
layout: page
hide: true
comments: false
aside: false
footer: false
top_img: false
---

<div id="music-stage">
  <!-- HERO -->
  <section class="hero">
    <div class="hero-inner">
      <p class="hero-kicker">SOUND · MOTION · LIGHT</p>
      <h1 class="hero-title">MUSIC<span class="dot">.</span></h1>
      <p class="hero-sub">A fluid journey through sound and gradient — scroll to feel the motion.</p>
      <div class="hero-scroll">SCROLL ↓</div>
    </div>
  </section>

  <!-- PROJECTS -->
  <section class="projects" id="projects">
    <!-- 10 个卡片由 JS 生成 -->
  </section>

  <!-- STRANDS FOOTER -->
  <section class="strands-section">
    <div class="strands-head">
      <h2>Feel the strands</h2>
      <p>Live WebGL — built with OGL (React Bits · Strands)</p>
    </div>
    <div id="strands" class="strands-canvas"></div>
    <footer class="site-foot">
      <span>© 2026 weiswift</span>
      <span>weiswift.github.io</span>
    </footer>
  </section>
</div>

<style>
  :root{
    --bg:#08080c;
    --ink:#f4f4f7;
    --muted:#8b8b99;
    --accent:#ff5e7a;
  }
  *{box-sizing:border-box;}
  html,body{margin:0;padding:0;background:var(--bg);color:var(--ink);
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,"PingFang SC","Microsoft YaHei",sans-serif;
    -webkit-font-smoothing:antialiased;overflow-x:hidden;}
  #music-stage{position:relative;z-index:1;}

  /* HERO */
  .hero{min-height:100vh;display:flex;align-items:center;justify-content:center;
    background:radial-gradient(120% 120% at 50% 0%,#1a1030 0%,#08080c 60%);
    text-align:center;padding:0 24px;position:relative;overflow:hidden;}
  .hero::after{content:"";position:absolute;inset:0;
    background:radial-gradient(60% 40% at 50% 100%,rgba(255,94,122,.18),transparent 70%);}
  .hero-inner{position:relative;z-index:2;max-width:760px;}
  .hero-kicker{letter-spacing:.5em;font-size:12px;color:var(--muted);margin:0 0 18px;}
  .hero-title{font-size:clamp(64px,16vw,180px);line-height:.9;margin:0;font-weight:800;
    background:linear-gradient(120deg,#fff,#ff5e7a 40%,#7c5cff 75%,#06b6d4);
    -webkit-background-clip:text;background-clip:text;color:transparent;
    background-size:200% 200%;animation:hue 8s ease infinite;}
  .hero-title .dot{color:var(--accent);-webkit-text-fill-color:var(--accent);}
  @keyframes hue{0%{background-position:0% 50%}50%{background-position:100% 50%}100%{background-position:0% 50%}}
  .hero-sub{color:var(--muted);font-size:clamp(14px,2.2vw,18px);margin:22px auto 0;max-width:520px;}
  .hero-scroll{margin-top:48px;font-size:12px;letter-spacing:.3em;color:var(--muted);animation:bob 2s ease-in-out infinite;}
  @keyframes bob{0%,100%{transform:translateY(0)}50%{transform:translateY(8px)}}

  /* PROJECTS */
  .projects{max-width:1200px;margin:0 auto;padding:120px 24px;}
  .projects-head{margin-bottom:60px;}
  .projects-head h2{font-size:clamp(32px,6vw,64px);margin:0;font-weight:800;letter-spacing:-.02em;}
  .projects-head p{color:var(--muted);margin:12px 0 0;}
  .grid{display:grid;grid-template-columns:repeat(2,1fr);gap:22px;}
  @media(max-width:760px){.grid{grid-template-columns:1fr;}}

  .project{position:relative;border-radius:18px;overflow:hidden;aspect-ratio:16/11;
    background:#0e0e15;cursor:pointer;isolation:isolate;
    transform:translateY(40px);opacity:0;transition:transform .9s cubic-bezier(.16,1,.3,1),opacity .9s;}
  .project.in{transform:none;opacity:1;}
  .project-img{position:absolute;inset:0;background-size:cover;background-position:center;
    transform:scale(1.08);transition:transform 1.1s cubic-bezier(.16,1,.3,1),filter 1.1s;
    filter:saturate(.9) brightness(.8);}
  .project::after{content:"";position:absolute;inset:0;z-index:1;
    background:linear-gradient(180deg,rgba(8,8,12,0) 30%,rgba(8,8,12,.85) 100%);}
  .project:hover .project-img{transform:scale(1);filter:saturate(1.25) brightness(1.05);}
  .project-foot{position:absolute;left:0;right:0;bottom:0;z-index:2;padding:22px 24px;}
  .project-line1{font-size:12px;letter-spacing:.18em;color:var(--muted);text-transform:uppercase;}
  .project-line2{display:flex;align-items:center;gap:10px;margin-top:8px;}
  .project-name{font-size:clamp(22px,3.4vw,34px);font-weight:800;display:flex;line-height:1;
    overflow:hidden;height:1em;}
  .col-reel{display:flex;flex-direction:column;transform:translateY(0);
    transition:transform .5s cubic-bezier(.16,1,.3,1);}
  .col-reel span{height:1em;display:block;}
  .project:hover .col-reel{transform:translateY(-100%);}
  .project-tag{margin-left:auto;font-size:12px;color:var(--ink);opacity:.0;
    border:1px solid rgba(255,255,255,.35);padding:6px 12px;border-radius:999px;
    transition:opacity .4s,background .4s,color .4s;}
  .project:hover .project-tag{opacity:1;background:#fff;color:#08080c;}
  .project-arrow{width:34px;height:34px;border-radius:50%;border:1px solid rgba(255,255,255,.3);
    display:grid;place-items:center;position:absolute;top:18px;right:18px;z-index:3;
    opacity:0;transform:translateY(-6px);transition:.45s cubic-bezier(.16,1,.3,1);}
  .project:hover .project-arrow{opacity:1;transform:none;}

  /* STRANDS */
  .strands-section{position:relative;padding:120px 24px 0;text-align:center;
    background:linear-gradient(180deg,#08080c,#0a0612);}
  .strands-head h2{font-size:clamp(30px,6vw,58px);margin:0;font-weight:800;}
  .strands-head p{color:var(--muted);margin:14px 0 0;}
  .strands-canvas{position:relative;width:100%;height:560px;margin-top:40px;border-radius:20px;
    overflow:hidden;background:transparent;}
  @media(max-width:760px){.strands-canvas{height:380px;}}
  .strands-canvas canvas{display:block;width:100%!important;height:100%!important;}
  .site-foot{display:flex;justify-content:space-between;padding:60px 0 40px;
    color:var(--muted);font-size:13px;border-top:1px solid rgba(255,255,255,.06);margin-top:60px;}

  /* 隐藏 butterfly 默认页眉/标题/侧栏干扰，实现沉浸式 */
  body{max-width:none!important;}
  #page .page-title,.page-title{display:none!important;}
  #page .aside-content,#recent-posts,#page-header .mask,
  #pagination,.comment-head,.console-card-group{display:none!important;}
  #music-stage{margin-top:-62px;}
  @media(max-width:900px){#music-stage{margin-top:-56px;}}
</style>

<script>
/* ---------- 项目数据（图片用 Unsplash 免费图，可替换为你自己的） ---------- */
const PROJECTS = [
  {name:"NOVA",   line1:"music • visual • webgl",         img:"https://images.unsplash.com/photo-1493225457124-a3eb161ffa5f?w=1200&q=80"},
  {name:"ECHO",   line1:"audio • design • motion",        img:"https://images.unsplash.com/photo-1511671782779-c97d3d27a1d4?w=1200&q=80"},
  {name:"PULSE",  line1:"beat • code • 3d",               img:"https://images.unsplash.com/photo-1459749411175-04bf5292ceea?w=1200&q=80"},
  {name:"FLUX",   line1:"synth • art • dev",              img:"https://images.unsplash.com/photo-1470225620780-dba8ba36b745?w=1200&q=80"},
  {name:"ORBIT",  line1:"space • sound • web",            img:"https://images.unsplash.com/photo-1419242902214-272b3f66ee7a?w=1200&q=80"},
  {name:"DRIFT",  line1:"ambient • ux • code",            img:"https://images.unsplash.com/photo-1458560871784-56d23406c091?w=1200&q=80"},
  {name:"PRISM",  line1:"light • music • lab",            img:"https://images.unsplash.com/photo-1507838153414-b4b713384a76?w=1200&q=80"},
  {name:"WAVE",   line1:"ocean • audio • 3d",             img:"https://images.unsplash.com/photo-1500462918059-b1a0cb512f1d?w=1200&q=80"},
  {name:"EMBER",  line1:"fire • sound • dev",             img:"https://images.unsplash.com/photo-1499415479124-43c32433a620?w=1200&q=80"},
  {name:"LUMA",   line1:"glow • visual • web",            img:"https://images.unsplash.com/photo-1506157786151-b8491531f063?w=1200&q=80"}
];

function nameReel(n){
  // 每个字母一列，hover 时整列上滚（复刻 lusion 的字母滚动）
  return n.split("").map(ch=>`<div class="col-reel"><span>${ch}</span><span>${ch}</span></div>`).join("");
}

const grid = document.createElement("div");
grid.className = "grid";
grid.innerHTML = PROJECTS.map((p,i)=>`
  <article class="project" data-i="${i}" style="transition-delay:${i*0.06}s">
    <div class="project-img" style="background-image:url('${p.img}')"></div>
    <div class="project-arrow">→</div>
    <div class="project-foot">
      <div class="project-line1">${p.line1}</div>
      <div class="project-line2">
        <div class="project-name">${nameReel(p.name)}</div>
        <span class="project-tag">VIEW</span>
      </div>
    </div>
  </article>`).join("");

const projects = document.getElementById("projects");
const head = document.createElement("div");
head.className = "projects-head";
head.innerHTML = `<h2>Selected Works</h2><p>Ten fluid pieces — hover to feel the motion.</p>`;
projects.appendChild(head);
projects.appendChild(grid);

/* 进入视口逐个浮现 */
const io = new IntersectionObserver((entries)=>{
  entries.forEach(e=>{ if(e.isIntersecting){ e.target.classList.add("in"); io.unobserve(e.target);} });
},{threshold:.15});
document.querySelectorAll(".project").forEach(el=>io.observe(el));

/* ---------- Strands 复刻 React Bits 的 Strands 组件（原生 WebGL2，零外部依赖） ---------- */
(function(){
  const MAX_STRANDS = 12, MAX_COLORS = 8;
    const VERT = `#version 300 es
in vec2 position; void main(){ gl_Position=vec4(position,0.0,1.0); }`;
    const FRAG = `#version 300 es
precision highp float;
uniform float uTime; uniform vec2 uResolution;
uniform vec3 uColors[${MAX_COLORS}]; uniform int uColorCount; uniform int uStrandCount;
uniform float uSpeed,uAmplitude,uWaviness,uThickness,uGlow,uTaper,uSpread,uHueShift,uIntensity,uOpacity,uScale,uSaturation;
out vec4 fragColor;
const float PI=3.14159265;
vec3 spectrum(float t){ return 0.5+0.5*cos(2.0*PI*(t+vec3(0.0,0.33,0.67))); }
vec3 samplePalette(float t){ t=fract(t); float s=t*float(uColorCount); int i=int(floor(s)); float b=fract(s);
  int n=i+1; if(n>=uColorCount)n=0; return mix(uColors[i],uColors[n],b); }
vec3 strandColor(float t){ return uColorCount>0?samplePalette(t):spectrum(t); }
void main(){
  vec2 uv=(gl_FragCoord.xy-0.5*uResolution)/uResolution.y; uv/=max(uScale,0.0001);
  float e=0.06+uIntensity*0.94; float env=pow(max(cos(uv.x*PI*1.3),0.0),uTaper);
  vec3 col=vec3(0.0);
  for(int i=0;i<${MAX_STRANDS};i++){ if(i>=uStrandCount)break;
    float fi=float(i); float ph=fi*1.7*uSpread; float freq=(2.0+fi*0.35)*uWaviness; float spd=1.4+fi*1.2;
    float tt=uTime*uSpeed;
    float w=sin(uv.x*freq+tt*spd+ph)*0.60+sin(uv.x*freq*1.1-tt*spd*0.7+ph*1.7)*0.40;
    float amp=(0.1+0.02*e)*env*uAmplitude; float y=w*amp;
    float d=abs(uv.y-y); float thick=(0.001+0.05*e)*(0.35+env)*uThickness;
    float g=thick/(d+thick*0.45); g=g*g;
    float h=fi/float(uStrandCount)+uv.x*0.30+uTime*0.04+uHueShift;
    col+=strandColor(h)*g*env;
  }
  col*=0.45+0.7*e; col=1.0-exp(-col*uGlow);
  float gray=dot(col,vec3(0.2126,0.7152,0.0722)); col=max(mix(vec3(gray),col,uSaturation),0.0);
  float lum=max(max(col.r,col.g),col.b); float alpha=clamp(lum,0.0,1.0)*uOpacity;
  fragColor=vec4(col*uOpacity,alpha);
}`;
    /* hex -> [r,g,b] (0..1)，替代 ogl 的 Color */
    function hexToRGB(hex){
      let h = hex.replace("#","");
      if(h.length===3) h = h.split("").map(c=>c+c).join("");
      const n = parseInt(h,16);
      return [ ((n>>16)&255)/255, ((n>>8)&255)/255, (n&255)/255 ];
    }
    function buildPalette(colors){
      const filled = (colors&&colors.length)?colors:["#ffffff"]; const out=[];
      for(let i=0;i<MAX_COLORS;i++){ const hex=filled[i]??filled[filled.length-1]; out.push(hexToRGB(hex)); }
      return out;
    }

    const ctn = document.getElementById("strands");
    const canvas = document.createElement("canvas");
    canvas.style.width="100%"; canvas.style.height="100%"; canvas.style.display="block";
    canvas.style.backgroundColor="transparent";
    ctn.appendChild(canvas);
    const gl = canvas.getContext("webgl2",{alpha:true,premultipliedAlpha:true,antialias:true});
    if(!gl){
      ctn.innerHTML = '<div style="display:grid;place-items:center;height:100%;color:#8b8b99">此浏览器不支持 WebGL2，Strands 效果已跳过</div>';
      return;
    }
    gl.clearColor(0,0,0,0); gl.enable(gl.BLEND); gl.blendFunc(gl.ONE, gl.ONE_MINUS_SRC_ALPHA);

    function compile(type,src){
      const s = gl.createShader(type); gl.shaderSource(s,src); gl.compileShader(s);
      if(!gl.getShaderParameter(s,gl.COMPILE_STATUS)){ console.error(gl.getShaderInfoLog(s)); return null; }
      return s;
    }
    const vs = compile(gl.VERTEX_SHADER, VERT), fs = compile(gl.FRAGMENT_SHADER, FRAG);
    if(!vs||!fs){ ctn.innerHTML='<div style="display:grid;place-items:center;height:100%;color:#8b8b99">着色器编译失败</div>'; return; }
    const prog = gl.createProgram(); gl.attachShader(prog,vs); gl.attachShader(prog,fs); gl.linkProgram(prog);
    if(!gl.getProgramParameter(prog,gl.LINK_STATUS)){ console.error(gl.getProgramInfoLog(prog)); return; }
    gl.useProgram(prog);

    /* 全屏三角形（等价于 ogl 的 Triangle） */
    const buf = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, buf);
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-1,-1, 3,-1, -1,3]), gl.STATIC_DRAW);
    const aPos = gl.getAttribLocation(prog,"position");
    gl.enableVertexAttribArray(aPos); gl.vertexAttribPointer(aPos,2,gl.FLOAT,false,0,0);

    /* 缓存 uniform 位置 */
    const U = {};
    ["uTime","uResolution","uColors","uColorCount","uStrandCount","uSpeed","uAmplitude","uWaviness",
     "uThickness","uGlow","uTaper","uSpread","uHueShift","uIntensity","uOpacity","uScale","uSaturation"]
      .forEach(n=>{ U[n]=gl.getUniformLocation(prog,n); });

    const colors = ["#F97316","#7C3AED","#06B6D4","#EAB308"];
    const P = { colors, count:4, speed:0.5, amplitude:1, waviness:1, thickness:0.7, glow:2.6,
      taper:3, spread:1, hueShift:0, intensity:0.7, saturation:1.5, opacity:1, scale:1.5 };

    function resize(){
      const dpr = Math.min(window.devicePixelRatio||1, 2);
      const w = Math.max(1, Math.floor(ctn.offsetWidth*dpr));
      const h = Math.max(1, Math.floor(ctn.offsetHeight*dpr));
      canvas.width=w; canvas.height=h;
      gl.viewport(0,0,w,h);
    }
    window.addEventListener("resize", resize); resize();

    let raf=0;
    const update = t=>{
      raf = requestAnimationFrame(update);
      gl.useProgram(prog);
      gl.uniform1f(U.uTime, t*0.001);
      gl.uniform2f(U.uResolution, canvas.width, canvas.height);
      const pal = buildPalette(P.colors);
      gl.uniform3fv(U.uColors, new Float32Array(pal.flat()));
      gl.uniform1i(U.uColorCount, Math.min(P.colors.length, MAX_COLORS));
      gl.uniform1i(U.uStrandCount, Math.min(Math.max(P.count,1), MAX_STRANDS));
      gl.uniform1f(U.uSpeed, P.speed); gl.uniform1f(U.uAmplitude, P.amplitude);
      gl.uniform1f(U.uWaviness, P.waviness); gl.uniform1f(U.uThickness, P.thickness);
      gl.uniform1f(U.uGlow, P.glow); gl.uniform1f(U.uTaper, P.taper);
      gl.uniform1f(U.uSpread, P.spread); gl.uniform1f(U.uHueShift, P.hueShift);
      gl.uniform1f(U.uIntensity, P.intensity); gl.uniform1f(U.uOpacity, P.opacity);
      gl.uniform1f(U.uScale, P.scale); gl.uniform1f(U.uSaturation, P.saturation);
      gl.clear(gl.COLOR_BUFFER_BIT);
      gl.drawArrays(gl.TRIANGLES, 0, 3);
    };
    raf = requestAnimationFrame(update);
})();
</script>
