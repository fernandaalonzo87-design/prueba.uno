// Frases motivacionales para mostrar aleatoriamente
const frases = [
  "Hoy es un gran día para crecer 💛",
  "Recuerda: lo estás haciendo muy bien ",
  "Respira profundo, todo estará bien ",
  "¡Una tarea a la vez, tú puedes! 💪",
  "Tu esfuerzo vale oro 🧡"
];

// Esta función sirve para activar modo noche
function toggleModoNoche() {
  document.body.classList.toggle("noche");
  document.getElementById("btn-noche").classList.toggle("activo");
}

// Esta función sirve para agregar tareas al día seleccionado
function agregarTarea() {
  const texto = document.getElementById("nueva-tarea").value.trim();
  const categoria = document.getElementById("categoria").value;
  const prioridad = document.getElementById("prioridad").value;
  const fecha = document.getElementById("fecha-tarea").value;
  const hora = document.getElementById("hora-tarea").value;
  const dia = document.getElementById("dia-semana").value;

  if (!texto || !dia) return;

  const tarea = document.createElement("li");
  tarea.classList.add(`prioridad-${prioridad}`);
  tarea.innerHTML = `
    <span onclick="toggleCompletado(this)">
      <strong>${texto}</strong><br/>
      <small>📅 ${fecha}  ${hora}</small><br/>
      <small>📌 ${categoria} | ${prioridad}</small>
    </span>
    <button class="estrella" onclick="toggleFavorito(this)">⭐</button>
    <button class="eliminar-tarea" onclick="this.parentElement.remove(); guardarTareas()">❌</button>
  `;

  document.querySelector(`#${dia} ul`).appendChild(tarea);
  document.getElementById("nueva-tarea").value = "";
  guardarTareas();
}

// Esta función alterna si una tarea está completada o no
function toggleCompletado(elemento) {
  const li = elemento.parentElement;
  li.classList.toggle("completada");
  guardarTareas();
}

// Esta función sirve para marcar como favorito
function toggleFavorito(boton) {
  boton.classList.toggle("favorito");
  guardarTareas();
}

// Esta función muestra una frase aleatoria en el post-it
function mostrarPostitDia() {
  const random = Math.floor(Math.random() * frases.length);
  document.getElementById("frase-motivacional").innerText = frases[random];
}

// Esta función guarda todas las tareas en localStorage
function guardarTareas() {
  const dias = [...document.querySelectorAll(".dia")];
  const data = dias.map(dia => {
    const tareas = [...dia.querySelectorAll("li")].map(li => ({
      texto: li.querySelector("strong")?.innerText || "",
      fecha: li.innerText.match(/\d{4}-\d{2}-\d{2}/)?.[0] || "",
      hora: li.innerText.match(/\d{2}:\d{2}/)?.[0] || "",
      categoria: li.innerText.includes("Personal") ? "personal" : li.innerText.includes("Estudio") ? "estudio" : "salud",
      prioridad: li.classList.contains("prioridad-alta") ? "alta" : li.classList.contains("prioridad-media") ? "media" : "baja",
      completada: li.classList.contains("completada"),
      favorito: li.querySelector(".estrella")?.classList.contains("favorito") || false
    }));
    return { id: dia.id, tareas };
  });
  localStorage.setItem("tareas", JSON.stringify(data));
}

// Esta función carga tareas guardadas desde localStorage
function cargarTareas() {
  const data = JSON.parse(localStorage.getItem("tareas")) || [];
  data.forEach(dia => {
    const ul = document.querySelector(`#${dia.id} ul`);
    dia.tareas.forEach(t => {
      const tarea = document.createElement("li");
      if (t.completada) tarea.classList.add("completada");
      tarea.classList.add(`prioridad-${t.prioridad}`);
      tarea.innerHTML = `
        <span onclick="toggleCompletado(this)">
          <strong>${t.texto}</strong><br/>
          <small>📅 ${t.fecha} 🕒 ${t.hora}</small><br/>
          <small>📌 ${t.categoria} | ${t.prioridad}</small>
        </span>
        <button class="estrella ${t.favorito ? 'favorito' : ''}" onclick="toggleFavorito(this)">⭐</button>
        <button class="eliminar-tarea" onclick="this.parentElement.remove(); guardarTareas()">❌</button>
      `;
      ul.appendChild(tarea);
    });
  });
}

// Esta función filtra las tareas por tipo: todas, completadas o pendientes
function filtrarTareas(tipo) {
  const tareas = document.querySelectorAll(".dia li");
  tareas.forEach(tarea => {
    switch (tipo) {
      case "todas": tarea.style.display = ""; break;
      case "completadas": tarea.style.display = tarea.classList.contains("completada") ? "" : "none"; break;
      case "pendientes": tarea.style.display = tarea.classList.contains("completada") ? "none" : ""; break;
    }
  });
}

// Esta función ordena las tareas poniendo favoritos arriba
function ordenarPorFavoritos() {
  const dias = document.querySelectorAll(".dia ul");
  dias.forEach(ul => {
    const tareas = [...ul.querySelectorAll("li")];
    tareas.sort((a, b) => {
      const aFav = a.querySelector(".estrella")?.classList.contains("favorito") ? -1 : 1;
      const bFav = b.querySelector(".estrella")?.classList.contains("favorito") ? -1 : 1;
      return aFav - bFav;
    });
    tareas.forEach(t => ul.appendChild(t));
  });
}

// Esta función se ejecuta al cargar la página
// Inicia eventos y carga tareas

document.addEventListener("DOMContentLoaded", () => {
  mostrarPostitDia();
  cargarTareas();
  document.getElementById("otro-postit").addEventListener("click", mostrarPostitDia);
  document.getElementById("btn-noche").addEventListener("click", toggleModoNoche);
});
