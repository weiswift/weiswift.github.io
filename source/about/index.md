---
title: 关于
date: 2023-10-10 16:27:03
layout: page
hide: true
comments: false
aside: false
footer: false
top_img: false
---

<div id="about-stage">
  <!-- 动态粒子背景（Antigravity 复刻，鼠标移上去形成光环） -->
  <div id="antigravity" class="antigravity"></div>

  <div class="about-overlay">
    <p class="about-kicker">ABOUT · WEISWIFT</p>
    <h1 class="about-title">俊泽<span class="dot">·</span>Jason</h1>
    <p class="about-lead">稳定而高效，不急功近利亦不一味盲从。</p>
    <div class="about-cards">
      <div class="about-card" data-tilt>
        <h3>理念</h3>
        <p>以平静之心做事，以笃定之姿前行。不追风口，只做长期正确的事。</p>
      </div>
      <div class="about-card" data-tilt>
        <h3>所爱</h3>
        <p>代码、音乐与光影。把理性与审美揉进每一个像素与每一段旋律。</p>
      </div>
      <div class="about-card" data-tilt>
        <h3>此站</h3>
        <p>一个用 Hexo 搭起的静态角落，记录思考、作品与那些不期而遇的灵感。</p>
      </div>
    </div>
    <footer class="about-foot">© 2026 Jason</footer>
  </div>
</div>

{% raw %}
<style>
  :root{
    --bg:#06060a;
    --ink:#f4f4f7;
    --muted:#9a9aab;
    --accent:#FF9FFC;
  }
  *{box-sizing:border-box;}
  html,body{margin:0;padding:0;background:var(--bg);color:var(--ink);
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,"PingFang SC","Microsoft YaHei",sans-serif;
    -webkit-font-smoothing:antialiased;overflow-x:hidden;}
  /* about-stage 全屏固定铺满，作为背景+内容容器；z-index 低于 header，使导航栏浮于其上 */
  #about-stage{position:fixed;inset:0;z-index:1;width:100vw;height:100vh;overflow:hidden;
    background:radial-gradient(120% 120% at 50% 0%,#160d22 0%,#06060a 60%);}

  /* 粒子画布作为 about-stage 的 absolute 全屏背景 */
  .antigravity{position:absolute;inset:0;z-index:0;pointer-events:none;width:100%;height:100%;}
  .antigravity canvas{display:block;width:100%!important;height:100%!important;}

  /* 内容层相对定位，在 about-stage 内垂直水平居中；顶部留出导航栏安全区 */
  .about-overlay{position:relative;z-index:2;min-height:100vh;
    display:flex;flex-direction:column;align-items:center;justify-content:center;
    text-align:center;padding:90px 24px 60px;gap:20px;pointer-events:none;}
  .about-overlay > *{pointer-events:auto;}

  .about-kicker{letter-spacing:.55em;font-size:12px;color:var(--muted);margin:0;}
  .about-title{font-size:clamp(28px,5vw,44px);line-height:1.1;margin:0;font-weight:700;
    background:linear-gradient(120deg,#fff,#FF9FFC 45%,#7c5cff 80%);
    -webkit-background-clip:text;background-clip:text;color:transparent;
    background-size:200% 200%;animation:hue 9s ease infinite;}
  .about-title .dot{color:var(--accent);-webkit-text-fill-color:var(--accent);}
  @keyframes hue{0%{background-position:0% 50%}50%{background-position:100% 50%}100%{background-position:0% 50%}}
  .about-lead{color:var(--muted);font-size:clamp(15px,2.4vw,20px);margin:6px auto 0;max-width:560px;}

  .about-cards{display:grid;grid-template-columns:repeat(3,1fr);gap:18px;max-width:980px;width:100%;margin-top:24px;}
  @media(max-width:760px){.about-cards{grid-template-columns:1fr;max-width:420px;}}
  .about-card{padding:26px 24px;border-radius:18px;background:rgba(255,255,255,.04);
    border:1px solid rgba(255,255,255,.08);backdrop-filter:blur(10px);text-align:left;
    transition:transform .5s cubic-bezier(.16,1,.3,1),background .5s,border-color .5s,box-shadow .5s;
    transform:perspective(800px) rotateX(0) rotateY(0);}
  .about-card:hover{background:rgba(255,159,252,.08);border-color:rgba(255,159,252,.5);
    box-shadow:0 24px 60px -28px rgba(255,159,252,.6);}
  .about-card h3{margin:0 0 10px;font-size:18px;font-weight:700;color:#fff;}
  .about-card p{margin:0;font-size:14px;line-height:1.7;color:var(--muted);}

  .about-foot{margin-top:40px;color:var(--muted);font-size:13px;letter-spacing:.04em;}

  /* 隐藏侧边栏、页面默认标题、主题页脚等干扰，仅保留顶部导航 */
  body{max-width:none!important;background:var(--bg)!important;overflow-x:hidden!important;}
  #page .page-title,.page-title{display:none!important;}
  #page .aside-content,#recent-posts,#page-header .mask,
  #pagination,.comment-head,.console-card-group,#sideNav,#sidebar-menus,#sidebar,
  #rightside,#toggle-sidebar,#web_bg,
  #footer-wrap,#footer /* 主题页脚盖住卡片，隐藏改用页面内 own 页脚 */{display:none!important;}
  /* about-stage 已 fixed 全屏，将主题页面容器压到背景层之下、不挡导航 */
  #page,#page .page-content,#page #article-container,.layout,.layout_page{background:transparent!important;box-shadow:none!important;position:relative!important;z-index:0!important;padding:0!important;margin:0!important;}
  #body-wrap,#content-inner{background:transparent!important;position:relative!important;z-index:0!important;}
  #page-header,#nav{z-index:10!important;}
  /* 导航栏透明浮于粒子背景之上，文字浅色保证可读 */
  #page-header{background:transparent!important;box-shadow:none!important;}
  #nav,#nav a,#nav .site-name,#nav .menus_item > a,
  #nav .menus_items .menus_item > a,#nav .site-page{color:#f4f4f7!important;text-shadow:0 1px 8px rgba(0,0,0,.6);background:transparent!important;}
  #nav .menus_item > a:hover,#nav .site-page:hover{color:var(--accent)!important;}
  #toggle-menu{color:#f4f4f7!important;}
  #about-stage{position:fixed!important;inset:0!important;width:100vw!important;height:100vh!important;margin:0!important;z-index:1!important;overflow-y:auto!important;overflow-x:hidden!important;}
  .antigravity{position:absolute!important;inset:0!important;width:100%!important;height:100%!important;z-index:0!important;pointer-events:none!important;}
  .about-overlay{position:relative!important;z-index:2!important;min-height:100vh;padding-top:90px!important;padding-bottom:60px!important;}
  @media(max-width:900px){#about-stage{margin:0!important;}}
</style>

<script>
(function(){
  let THREE;
  function loadThree(cb){
    if(window.THREE){ cb(window.THREE); return; }
    const s = document.createElement("script");
    s.src = "/js/three.min.js";
    s.onload = ()=>{ if(window.THREE) cb(window.THREE); else showFallback(); };
    s.onerror = ()=>{ showFallback(); };
    document.head.appendChild(s);
  }

  function showFallback(){
    const c = document.getElementById("antigravity");
    c.innerHTML = '<div style="display:grid;place-items:center;height:100vh;color:#9a9aab">此浏览器无法加载 Three.js，动态粒子效果已跳过</div>';
  }

  loadThree(function(T){
    run(T);
  });

  function run(THREE){
    const container = document.getElementById("antigravity");
    if(!container){ return; }

    // ---- Antigravity 参数（与 React Bits 组件一致） ----
    const props = {
      count: 300,
      magnetRadius: 6,
      ringRadius: 7,
      waveSpeed: 0.4,
      waveAmplitude: 1,
      particleSize: 1.5,
      lerpSpeed: 0.05,
      color: '#FF9FFC',
      autoAnimate: true,
      particleVariance: 1,
      rotationSpeed: 0,
      depthFactor: 1,
      pulseSpeed: 3,
      particleShape: 'capsule',   // capsule | sphere | box | tetrahedron
      fieldStrength: 10
    };
    const { count, magnetRadius, ringRadius, waveSpeed, waveAmplitude, particleSize,
      lerpSpeed, color, autoAnimate, particleVariance, rotationSpeed, depthFactor,
      pulseSpeed, particleShape, fieldStrength } = props;

    let width = container.clientWidth || window.innerWidth;
    let height = container.clientHeight || window.innerHeight;

    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(35, width/height, 0.1, 1000);
    camera.position.set(0,0,50);

    // 计算 z=0 平面相机可视范围（世界单位），用于把屏幕坐标映射到画面内
    function viewSize(){
      const dist = camera.position.z;                       // 相机到 z=0 平面的距离
      const vH = 2 * Math.tan((camera.fov*Math.PI/180)/2) * dist; // 可视高度
      const vW = vH * camera.aspect;                        // 可视宽度
      return { vW, vH };
    }
    let { vW, vH } = viewSize();

    const renderer = new THREE.WebGLRenderer({ alpha:true, antialias:true });
    renderer.setPixelRatio(Math.min(window.devicePixelRatio||1, 2));
    renderer.setSize(width, height);
    renderer.setClearColor(0x000000, 0);
    container.appendChild(renderer.domElement);

    // ---- 几何体 / 材质 ----
    let geo;
    if(particleShape === 'sphere') geo = new THREE.SphereGeometry(0.2, 16, 16);
    else if(particleShape === 'box') geo = new THREE.BoxGeometry(0.3,0.3,0.3);
    else if(particleShape === 'tetrahedron') geo = new THREE.TetrahedronGeometry(0.3);
    else geo = new THREE.CapsuleGeometry(0.1, 0.4, 4, 8);

    const mat = new THREE.MeshBasicMaterial({ color: new THREE.Color(color) });
    const mesh = new THREE.InstancedMesh(geo, mat, count);
    mesh.instanceMatrix.setUsage(THREE.DynamicDrawUsage);
    scene.add(mesh);

    const dummy = new THREE.Object3D();

    // ---- 鼠标状态 ----
    const ndc = { x:0, y:0 };          // 归一化设备坐标 (-1..1)
    const lastMousePos = { x:0, y:0 };
    let lastMoveTime = 0;
    const virtualMouse = { x:0, y:0 };

    function onPointerMove(clientX, clientY){
      const rect = renderer.domElement.getBoundingClientRect();
      ndc.x = ((clientX - rect.left)/rect.width)*2 - 1;
      ndc.y = -(((clientY - rect.top)/rect.height)*2 - 1);
      const md = Math.hypot(ndc.x-lastMousePos.x, ndc.y-lastMousePos.y);
      if(md > 0.001){ lastMoveTime = Date.now(); lastMousePos.x = ndc.x; lastMousePos.y = ndc.y; }
    }
    window.addEventListener("mousemove", e=>onPointerMove(e.clientX, e.clientY));
    window.addEventListener("touchmove", e=>{
      if(e.touches[0]) onPointerMove(e.touches[0].clientX, e.touches[0].clientY);
    }, {passive:true});

    // ---- 初始化粒子（使用相机可视范围内的世界坐标） ----
    const particles = [];
    for(let i=0;i<count;i++){
      const t = Math.random()*100;
      const speed = 0.01 + Math.random()/200;
      // 归一化 -1..1，再映射到可视范围 ±vW/2、±vH/2 内
      const x = (Math.random()-0.5) * vW * 1.2;
      const y = (Math.random()-0.5) * vH * 1.2;
      const z = (Math.random()-0.5)*20;
      const randomRadiusOffset = (Math.random()-0.5)*2;
      particles.push({ t, speed, mx:x, my:y, mz:z, cx:x, cy:y, cz:z, randomRadiusOffset });
    }

    const clock = new THREE.Clock();

    function animate(){
      requestAnimationFrame(animate);
      const elapsed = clock.getElapsedTime();

      // 目标鼠标位置（世界坐标）：ndc ∈ [-1,1] → 可视范围 ±vW/2、±vH/2
      let destX = ndc.x * (vW/2);
      let destY = ndc.y * (vH/2);

      if(autoAnimate && Date.now() - lastMoveTime > 2000){
        destX = Math.sin(elapsed*0.5) * (vW/4);
        destY = Math.cos(elapsed*0.5*2) * (vH/4);
      }

      // 平滑跟随
      const smooth = 0.05;
      virtualMouse.x += (destX - virtualMouse.x)*smooth;
      virtualMouse.y += (destY - virtualMouse.y)*smooth;

      const targetX = virtualMouse.x;
      const targetY = virtualMouse.y;
      const globalRotation = elapsed * rotationSpeed;

      particles.forEach((p, i)=>{
        p.t += p.speed/2;
        const t = p.t;

        const projectionFactor = 1 - p.cz/50;
        const projTX = targetX * projectionFactor;
        const projTY = targetY * projectionFactor;

        const dx = p.mx - projTX;
        const dy = p.my - projTY;
        const dist = Math.hypot(dx, dy);

        let tgX = p.mx, tgY = p.my, tgZ = p.mz * depthFactor;

        if(dist < magnetRadius){
          const angle = Math.atan2(dy, dx) + globalRotation;
          const wave = Math.sin(t*waveSpeed + angle) * (0.5*waveAmplitude);
          const deviation = p.randomRadiusOffset * (5/(fieldStrength+0.1));
          const curRing = ringRadius + wave + deviation;
          tgX = projTX + curRing*Math.cos(angle);
          tgY = projTY + curRing*Math.sin(angle);
          tgZ = p.mz*depthFactor + Math.sin(t)*(1*waveAmplitude*depthFactor);
        }

        p.cx += (tgX - p.cx)*lerpSpeed;
        p.cy += (tgY - p.cy)*lerpSpeed;
        p.cz += (tgZ - p.cz)*lerpSpeed;

        dummy.position.set(p.cx, p.cy, p.cz);
        dummy.lookAt(projTX, projTY, p.cz);
        dummy.rotateX(Math.PI/2);

        const curDist = Math.hypot(p.cx - projTX, p.cy - projTY);
        const distFromRing = Math.abs(curDist - ringRadius);
        let scaleFactor = 1 - distFromRing/10;
        scaleFactor = Math.max(0, Math.min(1, scaleFactor));

        const finalScale = scaleFactor * (0.8 + Math.sin(t*pulseSpeed)*0.2*particleVariance) * particleSize;
        dummy.scale.set(finalScale, finalScale, finalScale);
        dummy.updateMatrix();
        mesh.setMatrixAt(i, dummy.matrix);
      });

      mesh.instanceMatrix.needsUpdate = true;
      renderer.render(scene, camera);
    }
    animate();

    function resize(){
      width = container.clientWidth || window.innerWidth;
      height = container.clientHeight || window.innerHeight;
      camera.aspect = width/height;
      camera.updateProjectionMatrix();
      renderer.setSize(width, height);
      const vs = viewSize();
      vW = vs.vW; vH = vs.vH;
    }
    window.addEventListener("resize", resize);
  }
})();
</script>
{% endraw %}
