---
layout: single
title: "Catálogo de Productos"
permalink: /catalogo/
---

<h2>Insumos Imprescindibles</h2>

<!-- FILTROS -->
<div style="text-align:center; margin:20px 0;">
  <button class="filtro" onclick="filtrar('todos')">Todos</button>
  <button class="filtro" onclick="filtrar('Baño')">Baño</button>
  <button class="filtro" onclick="filtrar('Dormitorio')">Dormitorio</button>
  <button class="filtro" onclick="filtrar('Limpieza')">Limpieza</button>
  <button class="filtro" onclick="filtrar('Utiles')">Utiles</button>
  <button class="filtro" onclick="filtrar('Consumibles')">Consumibles</button>
</div>

<div class="contenedor-tienda">
  <div id="productos"></div>
  <div id="carrito"></div>
</div>

<style>

.contenedor-tienda {
  display: grid;
  grid-template-columns: 1fr 320px;
  gap: 20px;
}

/* GRID PRODUCTOS */
#productos {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 20px;
}

/* TARJETA */
.producto {
  border: 1px solid #eee;
  border-radius: 12px;
  padding: 15px;
  background: white;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

/* CONTENIDO SUPERIOR */
.producto-top {
  text-align: center;
}

.producto img {
  width: 100%;
  height: 160px;
  object-fit: cover;
  border-radius: 8px;
}

.nombre {
  font-weight: bold;
  margin: 10px 0 5px;
}

.categoria {
  font-size: 12px;
  color: #777;
}

.descripcion {
  font-size: 13px;
  min-height: 40px;
}

.precio {
  color: #f59b83;
  font-size: 18px;
  margin: 10px 0;
}

/* BOTON ABAJO */
.producto-bottom {
  margin-top: auto;
}

.boton {
  background-color: #f59b83;
  color: white;
  border: none;
  padding: 10px;
  border-radius: 8px;
  cursor: pointer;
  width: 100%;
}

/* CARRITO FIJO */
#carrito {
  position: sticky;
  top: 20px;
  height: fit-content;
  border: 1px solid #eee;
  padding: 15px;
  border-radius: 12px;
  background: #fafafa;
}

/* FILTROS */
.filtro {
  background-color: #eee;
  border: none;
  padding: 8px 14px;
  margin: 5px;
  border-radius: 20px;
  cursor: pointer;
}

.filtro:hover {
  background-color: #f59b83;
  color: white;
}

/* MOBILE */
@media (max-width: 900px) {
  .contenedor-tienda {
    grid-template-columns: 1fr;
  }

  #carrito {
    position: relative;
  }
}

</style>

<script>

// 🔥 YA NO BORRAMOS CARRITO AUTOMATICAMENTE
// localStorage.removeItem("carrito");

const URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vS34ggzEln16jeRxm1-L7a3p5TuaT4_oOe6VCI9nHlDr80RLcPk-ZptbPHFQ7ZCXxO7puhHUZoMfWq9/pub?output=csv";

let clienteData = JSON.parse(localStorage.getItem("clienteData")) || {};

// 🔹 CSV PARSER (igual que el tuyo)
function parseCSV(text) {
  const rows = [];
  let current = '';
  let insideQuotes = false;
  let row = [];

  for (let char of text) {
    if (char === '"') {
      insideQuotes = !insideQuotes;
    } else if (char === ',' && !insideQuotes) {
      row.push(current);
      current = '';
    } else if (char === '\n' && !insideQuotes) {
      row.push(current);
      rows.push(row);
      row = [];
      current = '';
    } else {
      current += char;
    }
  }

  if (current) {
    row.push(current);
    rows.push(row);
  }

  return rows;
}

// 🔹 CARGA PRODUCTOS
fetch(URL)
  .then(res => res.text())
  .then(data => {
    const filas = parseCSV(data);
    filas.shift();

    let html = "";

    filas.forEach(col => {
      const nombre = (col[0] || "").replace(/"/g, '&quot;');
      const categoria = col[1] || "";
      const foto = col[2] ? col[2].trim() : "";
      const descripcion = col[3] || "";
      const precio = parseFloat(col[4]) || 0;

      if(nombre){
        html += `
          <div class="producto" data-categoria="${categoria}">
            
            <div class="producto-top">
              <img src="/assets/images/${foto}" onerror="this.style.display='none'">
              <div class="nombre">${nombre}</div>
              <div class="categoria">${categoria}</div>
              <div class="descripcion">${descripcion}</div>
              <div class="precio">Q${precio.toFixed(2)}</div>
            </div>

            <div class="producto-bottom">
              <button class="boton" onclick='agregar("${nombre}", ${precio})'>
                Agregar
              </button>
            </div>

          </div>
        `;
      }
    });

    document.getElementById("productos").innerHTML = html;
    actualizarCarrito();
  });

// 🛒 GUARDAR CLIENTE
function guardarCliente(){
  const data = {
    nombre: document.getElementById("clienteNombre").value,
    telefono: document.getElementById("clienteTelefono").value,
    correo: document.getElementById("clienteCorreo").value,
    direccion: document.getElementById("clienteDireccion").value
  };
  localStorage.setItem("clienteData", JSON.stringify(data));
}

// 🛒 CARRITO
function actualizarCarrito(){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  let total = 0;
  let detalleHTML = "";

  carrito.forEach(p => {
    total += p.precio * p.cantidad;

    detalleHTML += `
      <div>
        ${p.nombre} x${p.cantidad}
      </div>
    `;
  });

  document.getElementById("carrito").innerHTML = `
    <h3>🛒 Pedido</h3>

    ${detalleHTML}

    <p><strong>Total: Q${total.toFixed(2)}</strong></p>

    <hr>

    <input id="clienteNombre" placeholder="Nombre" oninput="guardarCliente()">
    <input id="clienteTelefono" placeholder="Teléfono" oninput="guardarCliente()">
    <input id="clienteCorreo" placeholder="Correo" oninput="guardarCliente()">
    <input id="clienteDireccion" placeholder="Dirección" oninput="guardarCliente()">

    <button onclick="enviarWhatsApp()" class="boton">
      Enviar por WhatsApp
    </button>
  `;

  // 🔁 RECUPERAR DATOS
  let data = JSON.parse(localStorage.getItem("clienteData")) || {};

  document.getElementById("clienteNombre").value = data.nombre || "";
  document.getElementById("clienteTelefono").value = data.telefono || "";
  document.getElementById("clienteCorreo").value = data.correo || "";
  document.getElementById("clienteDireccion").value = data.direccion || "";
}

// ➕ AGREGAR
function agregar(nombre, precio){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  let existe = carrito.find(p => p.nombre === nombre);

  if(existe){
    existe.cantidad += 1;
  } else {
    carrito.push({nombre, precio, cantidad: 1});
  }

  localStorage.setItem("carrito", JSON.stringify(carrito));
  actualizarCarrito();
}

// 📲 WHATSAPP
function enviarWhatsApp(){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];
  let data = JSON.parse(localStorage.getItem("clienteData")) || {};

  let mensaje = "Hola, quiero hacer un pedido:%0A";

  carrito.forEach(p => {
    mensaje += `• ${p.nombre} x${p.cantidad}%0A`;
  });

  window.open(`https://wa.me/50240648733?text=${mensaje}`, "_blank");
}

// 🔎 FILTRO
function filtrar(cat) {
  let productos = document.querySelectorAll('.producto');

  productos.forEach(p => {
    if (cat === 'todos' || p.dataset.categoria === cat) {
      p.style.display = 'block';
    } else {
      p.style.display = 'none';
    }
  });
}

</script>
