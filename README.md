<!DOCTYPE html>
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
    }
    button {
      margin-top: 15px;
      background: #36A0CE;
      color: white;
      border: none;
      padding: 10px 20px;
      border-radius: 6px;
      cursor: pointer;
      font-size: 15px;
    }
    #descargar-fotos-btn {
      background: #00b894;
    }
  </style>
</head>

<body>

  <h1>Contador de Días de Gimnasio</h1>
  <div class="contador">Has ido <span id="contador">0</span> días al gimnasio este año.</div>

  <label style="text-align:center;">Selecciona el mes:</label>
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

  <button id="cargar-foto-btn">Subir foto al día seleccionado</button>
  <button id="descargar-fotos-btn">📥 Descargar todas las fotos</button>

  <input type="file" accept="image/*" id="foto-input" style="display:none">
  <img id="foto-preview">

<script>
const contadorSpan = document.getElementById('contador');
const diasDiv = document.getElementById('dias');
const fotoDiasDiv = document.getElementById('foto-dias');
const mesSelect = document.getElementById('mes');
const fotoInput = document.getElementById('foto-input');
const fotoPreview = document.getElementById('foto-preview');
const cargarFotoBtn = document.getElementById('cargar-foto-btn');
const descargarFotosBtn = document.getElementById('descargar-fotos-btn');

const meses = ["Enero","Febrero","Marzo","Abril","Mayo","Junio","Julio","Agosto","Septiembre","Octubre","Noviembre","Diciembre"];
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

function cargarDias(mes) {
  diasDiv.innerHTML = '';
  const primerDia = new Date(anioActual, mes, 1).getDay();
  const diasMes = new Date(anioActual, mes + 1, 0).getDate();

  for (let i = 0; i < primerDia; i++) diasDiv.appendChild(document.createElement('div'));

  for (let d = 1; d <= diasMes; d++) {
    const fecha = `${anioActual}-${String(mes+1).padStart(2,'0')}-${String(d).padStart(2,'0')}`;
    const el = document.createElement('div');
    el.className = 'dia';
    el.textContent = d;
    if (historial.includes(fecha)) el.classList.add('activo');
    if (fotos[fecha]) el.innerHTML += ' <span class="icono-foto">📷</span>';
    el.onclick = () => {
      fechaSeleccionada = fecha;
      if (!historial.includes(fecha)) {
        historial.push(fecha);
        guardarHistorial();
        actualizarContador();
      }
      fotoPreview.src = fotos[fecha] || '';
      fotoPreview.style.display = fotos[fecha] ? 'block' : 'none';
      cargarDias(mes);
      cargarFotoCalendario();
    };
    diasDiv.appendChild(el);
  }
}

function cargarFotoCalendario() {
  fotoDiasDiv.innerHTML = '';
  Object.keys(fotos).forEach(fecha => {
    const img = document.createElement('img');
    img.src = fotos[fecha];
    img.className = 'foto-dia-img';
    img.onclick = () => {
      fotoPreview.src = fotos[fecha];
      fotoPreview.style.display = 'block';
    };
    fotoDiasDiv.appendChild(img);
  });
}

cargarFotoBtn.onclick = () => {
  if (fechaSeleccionada && !fotos[fechaSeleccionada]) fotoInput.click();
};

fotoInput.onchange = e => {
  const file = e.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = ev => {
    fotos[fechaSeleccionada] = ev.target.result;
    guardarFotos();
    cargarDias(parseInt(mesSelect.value));
    cargarFotoCalendario();
    fotoPreview.src = ev.target.result;
    fotoPreview.style.display = 'block';
  };
  reader.readAsDataURL(file);
};

descargarFotosBtn.onclick = () => {
  const keys = Object.keys(fotos);
  if (!keys.length) {
    alert("No hay fotos para descargar");
    return;
  }
  keys.forEach(fecha => {
    const a = document.createElement('a');
    a.href = fotos[fecha];
    a.download = `Gym_${fecha}.jpg`;
    document.body.appendChild(a);
    a.click();
    a.remove();
  });
  alert(`Se descargaron ${keys.length} fotos`);
};

meses.forEach((m,i)=>{
  const o = document.createElement('option');
  o.value=i; o.textContent=m;
  if(i===hoy.getMonth()) o.selected=true;
  mesSelect.appendChild(o);
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
