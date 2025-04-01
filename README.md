# HoyFui😝
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Contador de Días de Gimnasio</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: #A2D5F2;
      color: #003049;
      padding: 20px;
      max-width: 1000px;
      margin: auto;
    }
    h1 {
      text-align: center;
      color: #003049;
    }
    .contador {
      font-size: 22px;
      margin-bottom: 20px;
      text-align: center;
    }
    .calendarios {
      display: flex;
      gap: 40px;
      justify-content: center;
      flex-wrap: wrap;
    }
    .calendario {
      flex: 1;
      min-width: 300px;
    }
    .mes-selector {
      display: block;
      margin: 10px auto;
      font-size: 16px;
      padding: 5px 10px;
      background-color: #36A0CE;
      color: white;
      border: none;
      border-radius: 5px;
    }
    .semana-header, .dias-grid {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 8px;
      margin-top: 10px;
    }
    .semana-header div {
      font-weight: bold;
      text-align: center;
      color: #003049;
    }
    .dia {
      cursor: pointer;
      padding: 10px 0;
      background: #FDCB6E;
      text-align: center;
      border-radius: 8px;
      font-weight: bold;
      position: relative;
      transition: background 0.3s;
      color: #003049;
    }
    .dia:hover {
      background: #F0932B;
      color: white;
    }
    .dia.activo {
      background: #4cd137;
      color: white;
    }
    .foto-dia-img {
      width: 100%;
      border-radius: 6px;
      cursor: pointer;
      transition: transform 0.2s;
    }
    .foto-dia-img:hover {
      transform: scale(1.05);
    }
    .icono-foto {
      position: absolute;
      bottom: 2px;
      right: 4px;
      font-size: 12px;
    }
    #foto-preview {
      margin-top: 20px;
      max-width: 100%;
      max-height: 250px;
      display: none;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.2);
    }
    #cargar-foto-btn {
      margin-top: 15px;
      display: none;
      background: #36A0CE;
      color: white;
      border: none;
      padding: 10px 20px;
      border-radius: 5px;
      cursor: pointer;
    }
    #cargar-foto-btn:hover {
      background: #003049;
    }
  </style>
</head>
<body>
  <h1>Contador de Días de Gimnasio</h1>
  <div class="contador">Has ido <span id="contador">0</span> días al gimnasio este año.</div>

  <label for="mes" style="text-align:center; display:block;">Selecciona el mes:</label>
  <select id="mes" class="mes-selector"></select>

  <div class="calendarios">
    <div class="calendario">
      <h3>Calendario Principal</h3>
      <div class="semana-header">
        <div>S</div><div>D</div><div>L</div><div>M</div><div>M</div><div>J</div><div>V</div>
      </div>
      <div class="dias-grid" id="dias"></div>
    </div>

    <div class="calendario">
      <h3>Historial de Fotos</h3>
      <div class="dias-grid" id="foto-dias"></div>
    </div>
  </div>

  <button id="cargar-foto-btn">Subir Foto</button>
  <input type="file" accept="image/*" capture="environment" id="foto-input" style="display: none;">
  <img id="foto-preview" src="" alt="Foto del día">

  <script>
    const contadorSpan = document.getElementById('contador');
    const diasDiv = document.getElementById('dias');
    const fotoDiasDiv = document.getElementById('foto-dias');
    const mesSelect = document.getElementById('mes');
    const fotoInput = document.getElementById('foto-input');
    const fotoPreview = document.getElementById('foto-preview');
    const cargarFotoBtn = document.getElementById('cargar-foto-btn');

    const meses = ["Enero", "Febrero", "Marzo", "Abril", "Mayo", "Junio", "Julio", "Agosto", "Septiembre", "Octubre", "Noviembre", "Diciembre"];
    const hoy = new Date();
    const anioActual = hoy.getFullYear();

    let historial = JSON.parse(localStorage.getItem('diasGimnasio')) || [];
    let fotos = JSON.parse(localStorage.getItem('fotosGimnasio')) || {};
    let fechaSeleccionada = null;

    function actualizarContador() {
      contadorSpan.textContent = historial.length;
    }

    function guardarHistorial() {
      localStorage.setItem('diasGimnasio', JSON.stringify(historial));
    }

    function guardarFotos() {
      localStorage.setItem('fotosGimnasio', JSON.stringify(fotos));
    }

    function esDiaMarcado(fechaStr) {
      return historial.includes(fechaStr);
    }

    function alternarDia(fechaStr, elementoDia) {
      if (esDiaMarcado(fechaStr)) {
        historial = historial.filter(f => f !== fechaStr);
        guardarHistorial();
        guardarFotos();
        actualizarContador();
        cargarDias(parseInt(mesSelect.value));
        cargarFotoCalendario();
        fotoPreview.style.display = 'none';
        cargarFotoBtn.style.display = 'none';
        fechaSeleccionada = null;
        return;
      }

      fechaSeleccionada = fechaStr;
      historial.push(fechaStr);
      guardarHistorial();
      actualizarContador();
      cargarDias(parseInt(mesSelect.value));
      cargarFotoCalendario();
      fotoPreview.style.display = fotos[fechaStr] ? 'block' : 'none';
      fotoPreview.src = fotos[fechaStr] || '';
      cargarFotoBtn.style.display = fotos[fechaStr] ? 'none' : 'inline-block';
    }

    function cargarDias(mes) {
      diasDiv.innerHTML = '';
      const primerDia = new Date(anioActual, mes, 1).getDay();
      const diasEnMes = new Date(anioActual, mes + 1, 0).getDate();

      for (let i = 0; i < primerDia; i++) {
        diasDiv.appendChild(document.createElement('div'));
      }

      for (let d = 1; d <= diasEnMes; d++) {
        const fecha = `${anioActual}-${String(mes + 1).padStart(2, '0')}-${String(d).padStart(2, '0')}`;
        const diaEl = document.createElement('div');
        diaEl.textContent = d;
        diaEl.className = 'dia';
        if (esDiaMarcado(fecha)) diaEl.classList.add('activo');
        if (fotos[fecha]) {
          const icon = document.createElement('span');
          icon.textContent = '📷';
          icon.className = 'icono-foto';
          diaEl.appendChild(icon);
        }
        diaEl.onclick = () => alternarDia(fecha, diaEl);
        diasDiv.appendChild(diaEl);
      }
    }

    function cargarFotoCalendario() {
      fotoDiasDiv.innerHTML = '';
      const clickCounters = {};
      for (const fecha in fotos) {
        const contenedor = document.createElement('div');
        const img = document.createElement('img');
        img.src = fotos[fecha];
        img.alt = fecha;
        img.className = 'foto-dia-img';
        clickCounters[fecha] = 0;
        img.onclick = () => {
          clickCounters[fecha]++;
          if (clickCounters[fecha] === 3) {
            delete fotos[fecha];
            guardarFotos();
            cargarFotoCalendario();
            fotoPreview.style.display = 'none';
            clickCounters[fecha] = 0;
          } else {
            fotoPreview.src = fotos[fecha];
            fotoPreview.style.display = 'block';
            setTimeout(() => {
              clickCounters[fecha] = 0;
            }, 1000);
          }
        };
        contenedor.appendChild(img);
        fotoDiasDiv.appendChild(contenedor);
      }
    }

    cargarFotoBtn.onclick = () => {
      if (fechaSeleccionada && !fotos[fechaSeleccionada]) fotoInput.click();
    };

    fotoInput.addEventListener('change', e => {
      const file = e.target.files[0];
      if (!file || fotos[fechaSeleccionada]) return;
      const reader = new FileReader();
      reader.onload = e2 => {
        const img = new Image();
        img.onload = () => {
          const canvas = document.createElement('canvas');
          const MAX_WIDTH = 300;
          const scale = MAX_WIDTH / img.width;
          canvas.width = MAX_WIDTH;
          canvas.height = img.height * scale;
          const ctx = canvas.getContext('2d');
          ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
          const compressed = canvas.toDataURL('image/jpeg', 0.6);
          fotos[fechaSeleccionada] = compressed;
          guardarFotos();
          cargarDias(parseInt(mesSelect.value));
          cargarFotoCalendario();
          fotoPreview.src = compressed;
          fotoPreview.style.display = 'block';
          cargarFotoBtn.style.display = 'none';
        };
        img.src = e2.target.result;
      };
      reader.readAsDataURL(file);
    });

    meses.forEach((nombre, i) => {
      const opt = document.createElement('option');
      opt.value = i;
      opt.textContent = nombre;
      if (i === hoy.getMonth()) opt.selected = true;
      mesSelect.appendChild(opt);
    });

    mesSelect.onchange = () => {
      cargarDias(parseInt(mesSelect.value));
      cargarFotoCalendario();
    };

    actualizarContador();
    cargarDias(hoy.getMonth());
    cargarFotoCalendario();
  </script>
</body>
</html>
