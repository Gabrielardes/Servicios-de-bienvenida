---
layout: single
title: "Catálogo de Insumos Imprescindibles"
permalink: /catalogo/
---

<style>
.contenedor-tienda {
  display: grid;
  grid-template-columns: 1fr 320px;
  gap: 20px;
}

.grid-productos {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 20px;
}

.producto {
  border: 1px solid #eee;
  border-radius: 12px;
  padding: 15px;
  background: white;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.producto img {
  width: 100%;
  border-radius: 10px;
}

.boton {
  background-color: #f59b83;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  width: 100%;
}

#carrito {
  position: sticky;
  top: 20px;
  border: 1px solid #eee;
  padding: 15px;
  border-radius: 12px;
  background: #fafafa;
}

input {
  width: 100%;
  margin-bottom: 5px;
  padding: 8px;
  border-radius: 6px;
  border: 1px solid #ddd;
}

@media (max-width: 900px) {
  .contenedor-tienda {
    grid-template-columns: 1fr;
  }

  #carrito {
    position: relative;
  }
}
</style>

<h2>🛒 Catálogo</h2>

<div class="contenedor-tienda">
  <div id="productos" class="grid-productos"></div>
  <div id="carrito"></div>
</div>

<script>
const URL = "TU_URL_AQUI";

let carrito = JSON.parse(localStorage.getItem("carrito")) || [];
let clienteData = JSON.parse(localStorage.getItem("clienteData")) || {};

fetch(URL)
  .then(res => res.json())
  .then(data => renderProductos(data))
  .catch(err => {
    document.getElementById("productos").innerHTML = "Error cargando productos";
    console.error(err);
  });

function renderProductos(productos) {
  const contenedor = document.getElementById("productos");
  contenedor.innerHTML = "";

  productos.forEach(p => {
    contenedor.innerHTML += `
      <div class="producto">
        <div>
          <img src="${p.imagen}">
          <h3>${p.nombre}</h3>
          <p>${p.descripcion || ""}</p>
          <strong>Q${p.precio}</strong>
        </div>

        <div style="margin-top:auto;">
          <button class="boton" onclick="agregar('${p.nombre}', ${p.precio})">
            Agregar
          </button>
        </div>
      </div>
    `;
  });
}

function agregar(nombre, precio) {
  carrito.push({ nombre, precio });
  localStorage.setItem("carrito", JSON.stringify(carrito));
  actualizarCarrito();
}

function guardarCliente(){
  clienteData = {
    nombre: document.getElementById("clienteNombre").value,
    telefono: document.getElementById("clienteTelefono").value,
    correo: document.getElementById("clienteCorreo").value,
    direccion: document.getElementById("clienteDireccion").value
  };

  localStorage.setItem("clienteData", JSON.stringify(clienteData));
}

function actualizarCarrito() {
  const div = document.getElementById("carrito");

  let total = carrito.reduce((sum, item) => sum + item.precio, 0);

  div.innerHTML = `
    <h3>🧾 Pedido</h3>

    <ul>
      ${carrito.map(p => `<li>${p.nombre} - Q${p.precio}</li>`).join("")}
    </ul>

    <p><strong>Total: Q${total.toFixed(2)}</strong></p>

    <hr>

    <h4>📋 Datos del cliente</h4>

    <input id="clienteNombre" placeholder="Nombre" oninput="guardarCliente()">
    <input id="clienteTelefono" placeholder="Teléfono" oninput="guardarCliente()">
    <input id="clienteCorreo" placeholder="Correo" oninput="guardarCliente()">
    <input id="clienteDireccion" placeholder="Dirección" oninput="guardarCliente()">

    <button class="boton" onclick="enviarPedido()">Enviar por WhatsApp</button>
  `;

  let data = JSON.parse(localStorage.getItem("clienteData")) || {};

  document.getElementById("clienteNombre").value = data.nombre || "";
  document.getElementById("clienteTelefono").value = data.telefono || "";
  document.getElementById("clienteCorreo").value = data.correo || "";
  document.getElementById("clienteDireccion").value = data.direccion || "";
}

function enviarPedido() {
  let mensaje = "Hola, quiero hacer un pedido:%0A";

  carrito.forEach(p => {
    mensaje += `- ${p.nombre} Q${p.precio}%0A`;
  });

  let total = carrito.reduce((sum, p) => sum + p.precio, 0);
  mensaje += `%0ATotal: Q${total}%0A`;

  let data = JSON.parse(localStorage.getItem("clienteData")) || {};

  mensaje += `%0ACliente:%0A`;
  mensaje += `Nombre: ${data.nombre}%0A`;
  mensaje += `Teléfono: ${data.telefono}%0A`;
  mensaje += `Correo: ${data.correo}%0A`;
  mensaje += `Dirección: ${data.direccion}`;

  window.open(`https://wa.me/50240339081?text=${mensaje}`, "_blank");
}

actualizarCarrito();
</script>
