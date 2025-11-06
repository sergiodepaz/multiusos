<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Herramienta Todo-en-Uno</title>
  <style>
    :root{
      --bg:#0f1724; --card:#0b1220; --accent:#6ee7b7; --muted:#94a3b8; --glass: rgba(255,255,255,0.03);
      --glass-2: rgba(255,255,255,0.02);
      font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    *{box-sizing:border-box}
    body{margin:0;background:linear-gradient(180deg,#071022 0%,#0b1220 100%);color:#e6eef8;min-height:100vh}
    .app{max-width:1100px;margin:28px auto;padding:20px}
    header{display:flex;align-items:center;gap:16px;margin-bottom:18px}
    .logo{width:56px;height:56px;border-radius:12px;background:linear-gradient(135deg,var(--accent),#60a5fa);display:flex;align-items:center;justify-content:center;font-weight:700;color:#052024}
    h1{font-size:1.35rem;margin:0}
    p.lead{margin:0;color:var(--muted);font-size:0.95rem}

    /* grid layout */
    .grid{display:grid;grid-template-columns:260px 1fr;gap:18px}
    nav{background:var(--card);padding:12px;border-radius:12px;min-height:320px}
    nav button{display:block;width:100%;background:transparent;border:0;color:inherit;padding:10px 12px;text-align:left;border-radius:8px;margin-bottom:6px;cursor:pointer}
    nav button.active{background:linear-gradient(90deg,rgba(255,255,255,0.02),rgba(255,255,255,0.015));box-shadow:0 6px 18px rgba(2,6,23,0.6)}

    .main{background:var(--glass);padding:16px;border-radius:12px;min-height:320px}

    /* toolbar */
    .tools-head{display:flex;justify-content:space-between;align-items:center;margin-bottom:12px}
    .tools-head .hint{color:var(--muted);font-size:0.9rem}

    /* individual panes */
    .pane{display:none}
    .pane.active{display:block}

    /* calculator */
    .calc{display:grid;grid-template-columns:repeat(4,1fr);gap:8px;max-width:360px}
    .calc input{grid-column:1/-1;padding:10px;border-radius:8px;border:0;background:var(--card);color:inherit;font-size:1.1rem}
    .calc button{padding:14px;border-radius:8px;border:0;background:var(--glass-2);cursor:pointer}

    /* todo */
    .todo-form{display:flex;gap:8px}
    .todo-list{margin-top:12px}
    .todo-item{display:flex;justify-content:space-between;gap:12px;padding:10px;border-radius:8px;background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent);margin-bottom:8px}

    /* contact form */
    form .row{display:flex;gap:8px}
    input,textarea,select{width:100%;padding:10px;border-radius:8px;border:0;background:var(--card);color:inherit}
    .actions{display:flex;gap:8px;justify-content:flex-end;margin-top:8px}

    /* image tool */
    .img-preview{max-width:320px;border-radius:8px;overflow:hidden;background:#071022;padding:8px}
    .img-preview img{display:block;max-width:100%;height:auto}

    /* password */
    .pw-row{display:flex;gap:8px;align-items:center}

    footer{margin-top:18px;color:var(--muted);font-size:0.85rem}

    @media (max-width:880px){.grid{grid-template-columns:1fr;}.logo{width:44px;height:44px}}
  </style>
</head>
<body>
  <div class="app">
    <header>
      <div class="logo">TU</div>
      <div>
        <h1>Herramienta Todo-en-Uno</h1>
        <p class="lead">Calculadora, lista de tareas, formulario, editor simple de imágenes y más — todo en una página.</p>
      </div>
    </header>

    <div class="grid">
      <nav aria-label="Herramientas">
        <button class="tab-btn active" data-tab="calc">🧮 Calculadora</button>
        <button class="tab-btn" data-tab="todo">✅ Lista de tareas</button>
        <button class="tab-btn" data-tab="form">✉️ Formulario</button>
        <button class="tab-btn" data-tab="image">🖼️ Imagen (preview + filtros)</button>
        <button class="tab-btn" data-tab="pw">🔐 Generador de contraseñas</button>
        <button class="tab-btn" data-tab="timer">⏱️ Temporizador</button>
        <button class="tab-btn" data-tab="notes">🗒️ Notas (localStorage)</button>
      </nav>

      <main class="main">
        <div class="tools-head">
          <div class="hint">Selecciona una herramienta en la columna izquierda.</div>
          <div id="status" class="status" aria-live="polite"></div>
        </div>

        <!-- CALCULADORA -->
        <section id="calc" class="pane active" aria-labelledby="calc">
          <h2>Calculadora</h2>
          <div class="calc">
            <input id="calc-screen" readonly value="0" aria-label="pantalla de calculadora">
            <button data-val="7">7</button>
            <button data-val="8">8</button>
            <button data-val="9">9</button>
            <button data-op="/">÷</button>
            <button data-val="4">4</button>
            <button data-val="5">5</button>
            <button data-val="6">6</button>
            <button data-op="*">×</button>
            <button data-val="1">1</button>
            <button data-val="2">2</button>
            <button data-val="3">3</button>
            <button data-op="-">−</button>
            <button data-val="0">0</button>
            <button data-val=".">.</button>
            <button id="calc-eq">=</button>
            <button data-op="+">+</button>
            <button id="calc-clear" style="grid-column:1/-1;background:#3b4252;color:#fff">Limpiar</button>
          </div>
        </section>

        <!-- TODO -->
        <section id="todo" class="pane" aria-labelledby="todo">
          <h2>Lista de tareas</h2>
          <div class="todo-form">
            <input id="todo-input" placeholder="Nueva tarea...">
            <button id="todo-add">Añadir</button>
          </div>
          <div class="todo-list" id="todo-list" aria-live="polite"></div>
          <div style="margin-top:10px;display:flex;gap:8px;justify-content:flex-end">
            <button id="todo-clear">Borrar completadas</button>
            <button id="todo-reset">Reset total</button>
          </div>
        </section>

        <!-- FORM -->
        <section id="form" class="pane" aria-labelledby="form">
          <h2>Formulario de contacto (simulado)</h2>
          <form id="contact-form">
            <div class="row">
              <input id="name" placeholder="Nombre" required>
              <input id="email" type="email" placeholder="Correo" required>
            </div>
            <div style="margin-top:8px">
              <select id="subject">
                <option value="general">General</option>
                <option value="bug">Reportar un error</option>
                <option value="feature">Sugerencia</option>
              </select>
            </div>
            <div style="margin-top:8px">
              <textarea id="message" rows="5" placeholder="Tu mensaje..."></textarea>
            </div>
            <div class="actions">
              <button type="submit">Enviar (simulado)</button>
              <button type="button" id="form-reset">Reset</button>
            </div>
          </form>
        </section>

        <!-- IMAGE TOOL -->
        <section id="image" class="pane" aria-labelledby="image">
          <h2>Editor de imagen simple</h2>
          <div style="display:flex;gap:16px;flex-wrap:wrap">
            <div>
              <input id="img-file" type="file" accept="image/*">
              <div style="margin-top:8px;display:flex;gap:8px">
                <button id="rotate">Rotar 90º</button>
                <button id="grayscale">Toggle B/N</button>
                <button id="download-img">Descargar</button>
              </div>
              <p style="color:var(--muted);font-size:0.9rem;margin-top:8px">Carga una imagen para verla y aplicar filtros simples.</p>
            </div>
            <div class="img-preview" id="img-preview"> <div style="color:var(--muted)">No hay imagen.</div> </div>
          </div>
        </section>

        <!-- PASSWORD -->
        <section id="pw" class="pane" aria-labelledby="pw">
          <h2>Generador de contraseñas</h2>
          <div class="pw-row">
            <input id="pw-length" type="number" min="4" max="64" value="16">
            <label style="color:var(--muted)">longitud</label>
            <label><input id="pw-upper" type="checkbox" checked> Mayúsculas</label>
            <label><input id="pw-lower" type="checkbox" checked> Minúsculas</label>
            <label><input id="pw-digits" type="checkbox" checked> Dígitos</label>
            <label><input id="pw-symbols" type="checkbox" checked> Símbolos</label>
            <button id="pw-gen">Generar</button>
          </div>
          <div style="margin-top:12px;display:flex;gap:8px;align-items:center">
            <input id="pw-output" readonly style="flex:1">
            <button id="pw-copy">Copiar</button>
          </div>
        </section>

        <!-- TIMER -->
        <section id="timer" class="pane" aria-labelledby="timer">
          <h2>Temporizador simple</h2>
          <div style="display:flex;gap:8px;align-items:center">
            <input id="timer-mins" type="number" value="1" min="0" style="width:80px">
            <label style="color:var(--muted)">minutos</label>
            <button id="timer-start">Iniciar</button>
            <button id="timer-stop">Parar</button>
            <button id="timer-reset">Reset</button>
          </div>
          <div style="margin-top:12px;font-size:1.6rem" id="timer-display">00:00</div>
        </section>

        <!-- NOTES -->
        <section id="notes" class="pane" aria-labelledby="notes">
          <h2>Notas (guardadas en tu navegador)</h2>
          <textarea id="notes-area" rows="8" placeholder="Escribe algo..."></textarea>
          <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:8px">
            <button id="notes-save">Guardar</button>
            <button id="notes-clear">Borrar</button>
          </div>
        </section>

        <footer>Proyecto de demostración — todo se ejecuta localmente en tu navegador. Los datos se guardan en localStorage.</footer>
      </main>
    </div>
  </div>

  <script>
    // --- Tabs ---
    document.querySelectorAll('.tab-btn').forEach(b=>b.addEventListener('click',()=>{
      document.querySelectorAll('.tab-btn').forEach(x=>x.classList.remove('active'));
      b.classList.add('active');
      const tab = b.dataset.tab;
      document.querySelectorAll('.pane').forEach(p=>p.classList.remove('active'));
      document.getElementById(tab).classList.add('active');
      document.getElementById('status').textContent = `Herramienta: ${b.textContent.trim()}`;
    }));

    // --- Calculadora ---
    const screen = document.getElementById('calc-screen');
    let expr = '';
    document.querySelectorAll('.calc button').forEach(btn=>{
      btn.addEventListener('click',()=>{
        const v = btn.dataset.val; const op = btn.dataset.op;
        if(v) { if(expr==='0') expr = v; else expr += v; screen.value = expr }
        if(op){ expr += op; screen.value = expr }
      });
    });
    document.getElementById('calc-clear').addEventListener('click',()=>{expr='';screen.value='0'});
    document.getElementById('calc-eq').addEventListener('click',()=>{
      try{ const res = Function('return '+expr)(); screen.value = String(res); expr=String(res) }
      catch(e){screen.value='Error'; expr=''}
    });

    // --- TODO (localStorage) ---
    const todoInput = document.getElementById('todo-input');
    const todoListEl = document.getElementById('todo-list');
    let todos = JSON.parse(localStorage.getItem('todos-v1')||'[]');
    function renderTodos(){ todoListEl.innerHTML=''; todos.forEach((t,i)=>{
      const div = document.createElement('div'); div.className='todo-item';
      div.innerHTML = `<div><input type="checkbox" ${t.done?'checked':''} data-i="${i}"> <strong>${t.text}</strong></div>
        <div><button data-del="${i}">Eliminar</button></div>`;
      todoListEl.appendChild(div);
    });
    }
    renderTodos();
    todoListEl.addEventListener('change',e=>{
      if(e.target.matches('input[type=checkbox]')){ const i = +e.target.dataset.i; todos[i].done = e.target.checked; saveTodos(); renderTodos(); }
    });
    todoListEl.addEventListener('click',e=>{ if(e.target.dataset.del){ todos.splice(+e.target.dataset.del,1); saveTodos(); renderTodos(); }});
    document.getElementById('todo-add').addEventListener('click',()=>{ const v=todoInput.value.trim(); if(!v) return; todos.push({text:v,done:false}); todoInput.value=''; saveTodos(); renderTodos();});
    document.getElementById('todo-clear').addEventListener('click',()=>{ todos = todos.filter(t=>!t.done); saveTodos(); renderTodos(); });
    document.getElementById('todo-reset').addEventListener('click',()=>{ if(confirm('¿Borrar todas las tareas?')){ todos=[]; saveTodos(); renderTodos(); }});
    function saveTodos(){ localStorage.setItem('todos-v1', JSON.stringify(todos)); }

    // --- Contact form (simulated) ---
    document.getElementById('contact-form').addEventListener('submit',e=>{
      e.preventDefault();
      const data = {name:document.getElementById('name').value, email:document.getElementById('email').value, subject:document.getElementById('subject').value, message:document.getElementById('message').value};
      console.log('Simulated submit',data);
      alert('Formulario enviado (simulado). Revisa la consola para ver los datos.');
    });
    document.getElementById('form-reset').addEventListener('click',()=>document.getElementById('contact-form').reset());

    // --- Image preview & simple edits ---
    let imgState = {angle:0,gr:false,img:null};
    const imgPreview = document.getElementById('img-preview');
    document.getElementById('img-file').addEventListener('change',e=>{
      const f = e.target.files && e.target.files[0]; if(!f) return; const url = URL.createObjectURL(f);
      imgState.img = new Image(); imgState.img.onload = ()=>{ imgState.angle=0; imgState.gr=false; showImage(); }
      imgState.img.src = url;
    });
    function showImage(){ imgPreview.innerHTML=''; const c = document.createElement('canvas'); const ctx=c.getContext('2d'); let w=imgState.img.width, h=imgState.img.height;
      // scale to max width
      const maxW=600; if(w>maxW){ const ratio = maxW/w; w = maxW; h = h*ratio; }
      c.width=w; c.height=h; ctx.drawImage(imgState.img,0,0,w,h);
      if(imgState.gr){ const imgd=ctx.getImageData(0,0,w,h); const d=imgd.data; for(let i=0;i<d.length;i+=4){ const avg=(d[i]+d[i+1]+d[i+2])/3; d[i]=d[i+1]=d[i+2]=avg } ctx.putImageData(imgd,0,0); }
      if(imgState.angle!==0){ /* rotate canvas visually using CSS transform */ }
      const imgEl = document.createElement('img'); imgEl.src=c.toDataURL('image/png'); imgEl.style.transform = `rotate(${imgState.angle}deg)`;
      imgPreview.appendChild(imgEl);
    }
    document.getElementById('rotate').addEventListener('click',()=>{ if(!imgState.img) return alert('Carga primero una imagen'); imgState.angle = (imgState.angle+90)%360; showImage(); });
    document.getElementById('grayscale').addEventListener('click',()=>{ if(!imgState.img) return alert('Carga primero una imagen'); imgState.gr = !imgState.gr; showImage(); });
    document.getElementById('download-img').addEventListener('click',()=>{
      if(!imgPreview.querySelector('img')) return alert('Nada para descargar');
      const a=document.createElement('a'); a.href = imgPreview.querySelector('img').src; a.download = 'imagen-editada.png'; a.click();
    });

    // --- Password generator ---
    function genPassword(length, opts){ const upper = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'; const lower='abcdefghijklmnopqrstuvwxyz'; const digits='0123456789'; const syms='!@#$%^&*()-_=+[]{};:,.<>?'; let chars=''; if(opts.upper) chars+=upper; if(opts.lower) chars+=lower; if(opts.digits) chars+=digits; if(opts.symbols) chars+=syms; if(!chars) return '';
      let out=''; const cryptoObj = window.crypto || window.msCrypto; const rand = new Uint32Array(length); cryptoObj.getRandomValues(rand);
      for(let i=0;i<length;i++){ out += chars[rand[i] % chars.length]; }
      return out;
    }
    document.getElementById('pw-gen').addEventListener('click',()=>{
      const l = Math.min(64, Math.max(4, +document.getElementById('pw-length').value||16));
      const opts = {upper:document.getElementById('pw-upper').checked, lower:document.getElementById('pw-lower').checked, digits:document.getElementById('pw-digits').checked, symbols:document.getElementById('pw-symbols').checked};
      const pw = genPassword(l,opts); document.getElementById('pw-output').value = pw;
    });
    document.getElementById('pw-copy').addEventListener('click',()=>{ navigator.clipboard.writeText(document.getElementById('pw-output').value||'').then(()=>alert('Copiado al portapapeles')) });

    // --- Timer ---
    let timerId=null, remaining=0;
    function fmt(s){ const mm = String(Math.floor(s/60)).padStart(2,'0'); const ss = String(s%60).padStart(2,'0'); return `${mm}:${ss}` }
    document.getElementById('timer-start').addEventListener('click',()=>{
      const mins = Math.max(0, +document.getElementById('timer-mins').value||0); remaining = Math.floor(mins*60);
      if(remaining<=0) return alert('Pon minutos mayores que 0'); clearInterval(timerId); document.getElementById('timer-display').textContent = fmt(remaining);
      timerId = setInterval(()=>{ remaining--; document.getElementById('timer-display').textContent = fmt(remaining); if(remaining<=0){ clearInterval(timerId); alert('¡Tiempo!'); } },1000);
    });
    document.getElementById('timer-stop').addEventListener('click',()=>{ clearInterval(timerId); });
    document.getElementById('timer-reset').addEventListener('click',()=>{ clearInterval(timerId); remaining=0; document.getElementById('timer-display').textContent='00:00'; });

    // --- Notes (localStorage) ---
    const notesArea = document.getElementById('notes-area');
    notesArea.value = localStorage.getItem('notes-v1')||'';
    document.getElementById('notes-save').addEventListener('click',()=>{ localStorage.setItem('notes-v1', notesArea.value); alert('Notas guardadas localmente'); });
    document.getElementById('notes-clear').addEventListener('click',()=>{ if(confirm('Borrar notas?')){ notesArea.value=''; localStorage.removeItem('notes-v1'); }});

    // Accessibility: keyboard focus for nav
    document.querySelectorAll('nav button').forEach(b=>b.addEventListener('keydown',e=>{
      if(e.key==='ArrowDown'||e.key==='ArrowUp'){ e.preventDefault(); const all = Array.from(document.querySelectorAll('nav button')); const i = all.indexOf(b); const next = e.key==='ArrowDown' ? all[(i+1)%all.length] : all[(i-1+all.length)%all.length]; next.focus(); }
    }));

    // inicial status
    document.getElementById('status').textContent = 'Herramienta: Calculadora';
  </script>
</body>
</html>
