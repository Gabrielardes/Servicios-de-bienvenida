---
layout: single
title: "Catálogo de Productos"
permalink: /catalogo/
---

<h2>Insumos Imprescindibles</h2>

<div id="carrito" style="margin-bottom:20px;"></div>
<div id="productos"></div>

<script>

// 🔥 LIMPIAR CARRITO AL CARGAR
localStorage.removeItem("carrito");

const URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vS34ggzEln16jeRxm1-L7a3p5TuaT4_oOe6VCI9nHlDr80RLcPk-ZptbPHFQ7ZCXxO7puhHUZoMfWq9/pub?output=csv";

// 🔹 PARSER CSV
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

// 🔹 Cargar productos
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
          <div style="border:1px solid #ddd; padding:15px; margin:10px;">
            
            <img src="/assets/images/${foto}" width="150" onerror="this.style.display='none'"><br>

            <strong>${nombre}</strong><br>
            <small>${categoria}</small><br>
            <p>${descripcion}</p>
            <b>Q${precio.toFixed(2)}</b><br><br>

            <button onclick='agregar("${nombre}", ${precio})'>
              Agregar al carrito
            </button>

          </div>
        `;
      }
    });

    document.getElementById("productos").innerHTML = html;
    actualizarCarrito();
  })
  .catch(err => {
    console.error("Error cargando productos:", err);
    document.getElementById("productos").innerHTML = "Error cargando catálogo";
  });

// 🛒 Agregar producto
function agregar(nombre, precio){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  let existe = carrito.find(p => p.nombre === nombre);

  if(existe){
    existe.cantidad += 1;
  } else {
    carrito.push({nombre: nombre, precio: precio, cantidad: 1});
  }

  localStorage.setItem("carrito", JSON.stringify(carrito));
  actualizarCarrito();
}

// 🛒 Mostrar carrito
function actualizarCarrito(){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  let total = 0;
  let detalleHTML = "";

  carrito.forEach(p => {
    total += p.precio * p.cantidad;

    detalleHTML += `
      <div style="margin-bottom:10px;">
        ${p.nombre} x${p.cantidad} - Q${(p.precio * p.cantidad).toFixed(2)}
        <button onclick='sumar("${p.nombre}")'>+</button>
        <button onclick='restar("${p.nombre}")'>-</button>
      </div>
    `;
  });

  document.getElementById("carrito").innerHTML = `
    <h3>🛒 Carrito (${carrito.length})</h3>

    ${detalleHTML}

    <p><strong>Total: Q${total.toFixed(2)}</strong></p>

    <input id="clienteNombre" placeholder="Tu nombre" style="width:100%; margin-bottom:5px;"><br>
    <input id="clienteTelefono" placeholder="Teléfono" style="width:100%; margin-bottom:5px;"><br>
    <input id="clienteCorreo" placeholder="Correo electrónico" style="width:100%; margin-bottom:5px;"><br>
    <input id="clienteDireccion" placeholder="Dirección de entrega" style="width:100%; margin-bottom:10px;"><br>

    <button onclick="enviarWhatsApp()" style="background:green;color:white;padding:10px;">
      Pedir por WhatsApp
    </button>
  `;
}

// ➕ SUMAR
function sumar(nombre){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  carrito.forEach(p => {
    if(p.nombre === nombre){
      p.cantidad += 1;
    }
  });

  localStorage.setItem("carrito", JSON.stringify(carrito));
  actualizarCarrito();
}

// ➖ RESTAR
function restar(nombre){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  carrito = carrito.map(p => {
    if(p.nombre === nombre){
      p.cantidad -= 1;
    }
    return p;
  }).filter(p => p.cantidad > 0);

  localStorage.setItem("carrito", JSON.stringify(carrito));
  actualizarCarrito();
}

// 📲 WHATSAPP + CRM
function enviarWhatsApp(){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  let nombre = document.getElementById("clienteNombre").value;
  let telefono = document.getElementById("clienteTelefono").value;
  let correo = document.getElementById("clienteCorreo").value;
  let direccion = document.getElementById("clienteDireccion").value;

  if(!nombre || !telefono || !correo || !direccion){
    alert("Por favor completa todos los datos");
    return;
  }

  let total = 0;
  let pedidoTexto = "";
  let mensaje = `Hola, quiero hacer un pedido:%0A`;

  carrito.forEach(p => {
    total += p.precio * p.cantidad;
    pedidoTexto += `${p.nombre} x${p.cantidad} | `;
    mensaje += `• ${p.nombre} x${p.cantidad}%0A`;
  });

  mensaje += `%0ATotal: Q${total.toFixed(2)}%0A`;
  mensaje += `%0ANombre: ${nombre}%0A`;
  mensaje += `Teléfono: ${telefono}%0A`;
  mensaje += `Correo: ${correo}%0A`;
  mensaje += `Dirección: ${direccion}`;

  // 🔥 ENVÍO A GOOGLE SHEETS (SIN CORS)
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

  // 📲 WhatsApp
  window.open(`https://wa.me/50240648733?text=${mensaje}`, "_blank");

  // 🧹 limpiar carrito
  localStorage.removeItem("carrito");
  actualizarCarrito();
}

</scrip
