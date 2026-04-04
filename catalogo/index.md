---
layout: single
title: "Catálogo de Productos"
permalink: /catalogo/
---

<h2>Catálogo</h2>

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
      const nombre = col[0];
      const categoria = col[1];
      const foto = col[2] ? col[2].trim() : "";
      const descripcion = col[3];
      const precio = parseFloat(col[4]);

      if(nombre){
        html += `
          <div style="border:1px solid #ddd; padding:15px; margin:10px;">
            
            <img src="/assets/images/${foto}" 
                 width="150" 
                 onerror="this.style.display='none'"><br>

            <strong>${nombre}</strong><br>
            <small>${categoria}</small><br>
            <p>${descripcion}</p>
            <b>Q${precio.toFixed(2)}</b><br><br>

            <button onclick="agregar('${nombre}', ${precio})">
              Agregar al carrito
            </button>

          </div>
        `;
      }
    });

    document.getElementById("productos").innerHTML = html;
    actualizarCarrito();
  });

// 🛒 Agregar producto
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

// 🛒 Mostrar carrito (MEJORADO)
function actualizarCarrito(){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  let total = 0;
  let detalleHTML = "";

  carrito.forEach(p => {
    total += p.precio * p.cantidad;

    detalleHTML += `
      <div style="margin-bottom:10px;">
        ${p.nombre} x${p.cantidad} - Q${(p.precio * p.cantidad).toFixed(2)}
        <button onclick="sumar('${p.nombre}')">+</button>
        <button onclick="restar('${p.nombre}')">-</button>
      </div>
    `;
  });

  document.getElementById("carrito").innerHTML = `
    <h3>🛒 Carrito (${carrito.length})</h3>

    ${detalleHTML}

    <p><strong>Total: Q${total.toFixed(2)}</strong></p>

    <input id="clienteNombre" placeholder="Tu nombre" style="width:100%; margin-bottom:5px;"><br>
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

// 📲 ENVIAR WHATSAPP
function enviarWhatsApp(){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  let nombre = document.getElementById("clienteNombre").value;
  let direccion = document.getElementById("clienteDireccion").value;

  if(!nombre || !direccion){
    alert("Por favor completa tus datos");
    return;
  }

  let total = 0;
  let mensaje = `Hola, quiero hacer un pedido:%0A`;

  carrito.forEach(p => {
    total += p.precio * p.cantidad;
    mensaje += `• ${p.nombre} x${p.cantidad} - Q${(p.precio * p.cantidad).toFixed(2)}%0A`;
  });

  mensaje += `%0ATotal: Q${total.toFixed(2)}%0A`;
  mensaje += `%0ANombre: ${nombre}%0A`;
  mensaje += `Dirección: ${direccion}`;

  window.open(`https://wa.me/50240648733?text=${mensaje}`, "_blank");
}
</script>
