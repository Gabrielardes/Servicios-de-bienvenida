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

/* PRODUCTOS SCROLL */
#productos {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 20px;
  max-height: 80vh;
  overflow-y: auto;
  padding-right: 10px;
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

.producto-top {
  text-align: center;
}

.producto img {
  width: 100%;
  height: 160px;
  object-fit: cover;
  border-radius: 8px;
}

.nombre { font-weight: bold; margin: 10px 0 5px; }
.categoria { font-size: 12px; color: #777; }
.descripcion { font-size: 13px; min-height: 40px; }

.precio {
  color: #f59b83;
  font-size: 18px;
  margin: 10px 0;
}

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

/* CARRITO */
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

  #productos {
    max-height: none;
  }

  #carrito {
    position: relative;
  }
}

</style>

<script>

const URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vS34ggzEln16jeRxm1-L7a3p5TuaT4_oOe6VCI9nHlDr80RLcPk-ZptbPHFQ7ZCXxO7puhHUZoMfWq9/pub?output=csv";

// 🧹 LIMPIAR TODO AL CARGAR (SOLUCIÓN A TU PROBLEMA)
localStorage.removeItem("carrito");
localStorage.removeItem("clienteData");

// 🔹 CARGAR PRODUCTOS
fetch(URL)
  .then(res => res.text())
  .then(data => {
    const filas = data.split("\n").map(f => f.split(","));
    filas.shift();

    let html = "";

    filas.forEach(col => {
      if(col.length < 5) return;

      const nombre = col[0]?.replace(/"/g, "") || "";
      const categoria = col[1] || "";
      const foto = col[2]?.trim() || "";
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

<div class="producto-bottom" id="control-${nombre}">
  ${controlCantidad(nombre, precio)}
</div>
          </div>
        `;
      }
    });

    document.getElementById("productos").innerHTML = html;
    actualizarCarrito();
    actualizarControles();
  });

// 🛒 ACTUALIZAR CARRITO
function actualizarCarrito(){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  let total = 0;
  let detalleHTML = "";

  carrito.forEach(p => {
    total += p.precio * p.cantidad;
    detalleHTML += `<div>${p.nombre} x${p.cantidad}</div>`;
  });

  document.getElementById("carrito").innerHTML = `
    <h3>🛒 Pedido</h3>

    ${detalleHTML}

    <p><strong>Total: Q${total.toFixed(2)}</strong></p>

    <hr>

    <input id="clienteNombre" placeholder="Nombre">
    <input id="clienteTelefono" placeholder="Teléfono">
    <input id="clienteCorreo" placeholder="Correo">
    <input id="clienteDireccion" placeholder="Dirección">

    <button onclick="enviarWhatsApp()" class="boton">
      Enviar Pedido y Confirmar Existencias
    </button>
  `;
}
// 🧠 CONTROL DE CANTIDAD (NUEVO)
function controlCantidad(nombre, precio){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];
  let producto = carrito.find(p => p.nombre === nombre);

  if(producto){
    return `
      <div style="display:flex; align-items:center; justify-content:space-between; gap:10px;">
        <button onclick="cambiarCantidad('${nombre}', -1)" class="boton">-</button>
        <span>${producto.cantidad}</span>
        <button onclick="cambiarCantidad('${nombre}', 1)" class="boton">+</button>
      </div>
    `;
  } else {
    return `
      <button class="boton" onclick='agregar("${nombre}", ${precio})'>
        Agregar
      </button>
    `;
  }
}
function cambiarCantidad(nombre, cambio){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  let producto = carrito.find(p => p.nombre === nombre);

  if(producto){
    producto.cantidad += cambio;

    if(producto.cantidad <= 0){
      carrito = carrito.filter(p => p.nombre !== nombre);
    }
  }

  localStorage.setItem("carrito", JSON.stringify(carrito));

  actualizarCarrito();
  actualizarControles();
}
function actualizarControles(){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  carrito.forEach(p => {
    let contenedor = document.getElementById(`control-${p.nombre}`);
    if(contenedor){
      contenedor.innerHTML = controlCantidad(p.nombre, p.precio);
    }
  });

  // También actualizar los que ya no están
  document.querySelectorAll("[id^='control-']").forEach(div => {
    let nombre = div.id.replace("control-", "");
    let producto = carrito.find(p => p.nombre === nombre);

    if(!producto){
      // reconstruir botón agregar (necesitamos precio)
      let precioTexto = div.closest(".producto").querySelector(".precio").innerText;
      let precio = parseFloat(precioTexto.replace("Q",""));

      div.innerHTML = `
        <button class="boton" onclick='agregar("${nombre}", ${precio})'>
          Agregar
        </button>
      `;
    }
  });
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
   actualizarControles(); // 👈 PASO 5 AQUÍ
}

// 📲 WHATSAPP + GOOGLE SHEETS
function enviarWhatsApp(){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  const nombre = document.getElementById("clienteNombre").value;
  const telefono = document.getElementById("clienteTelefono").value;
  const correo = document.getElementById("clienteCorreo").value;
  const direccion = document.getElementById("clienteDireccion").value;

  if(!nombre || !telefono || !correo || !direccion){
    alert("Por favor completa todos los datos");
    return;
  }

  let total = 0;
  let pedidoTexto = "";
  let mensaje = "Hola, quiero hacer un pedido:%0A";

  carrito.forEach(p => {
    total += p.precio * p.cantidad;
    pedidoTexto += `${p.nombre} x${p.cantidad} | `;
    mensaje += `• ${p.nombre} x${p.cantidad}%0A`;
  });

  mensaje += `%0ATotal: Q${total.toFixed(2)}%0A`;
  mensaje += `%0ANombre: ${nombre}%0A`;
  mensaje += `Teléfono: ${telefono}%0A`;
  mensaje += `Correo: ${correo}%0A`;
  mensaje += `Dirección: ${direccion}%0A`;
  mensaje += `%0AGracias por tu pedido, te contactaremos en breve 🙌`;

  // 🔥 CRM
  fetch("https://script.google.com/macros/s/AKfycbw3xO7W-3dJ0YgGvOUkL46nCTB5OMyfO5wkMONmEPiZChq-r3bx9kVFywTx9yxrklh9/exec", {
    method: "POST",
    mode: "no-cors",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      nombre: nombre,
      telefono: telefono,
      correo: correo,
      direccion: direccion,
      pedido: pedidoTexto,
      total: total.toFixed(2)
    })
  });

  // 📲 WhatsApp (UNA SOLA VEZ)
  window.open(`https://wa.me/50240648733?text=${mensaje}`, "_blank");

  // 🧹 LIMPIAR TODO DESPUÉS DE ENVIAR
  localStorage.removeItem("carrito");
  localStorage.removeItem("clienteData");

  setTimeout(() => {
    actualizarCarrito();
  }, 500);

  alert("Pedido enviado correctamente ✅");
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
