---
layout: default
title: Catálogo
permalink: /catalogo/
---

<h1>Catálogo de Productos</h1>

<div id="productos"></div>

<script>
const URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vS34ggzEln16jeRxm1-L7a3p5TuaT4_oOe6VCI9nHlDr80RLcPk-ZptbPHFQ7ZCXxO7puhHUZoMfWq9/pub?output=csv";

fetch(URL)
  .then(res => res.text())
  .then(data => {
    const filas = data.split("\n").slice(1);

    let html = "";

    filas.forEach(fila => {
      const cols = fila.split(",");

      const nombre = cols[0];
      const categoria = cols[1];
      const foto = cols[2];
      const descripcion = cols[3];
      const precio = cols[4];
      const slug = cols[5];

      if(nombre){
        html += `
          <div style="border:1px solid #ddd; padding:15px; margin:10px;">
            <img src="${foto}" width="150"><br>
            <strong>${nombre}</strong><br>
            <small>${categoria}</small><br>
            <p>${descripcion}</p>
            <b>${precio}</b><br><br>
            <button onclick="agregar('${nombre}','${precio}')">
              Agregar al carrito
            </button>
          </div>
        `;
      }
    });

    document.getElementById("productos").innerHTML = html;
  });

function agregar(nombre, precio){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];
  carrito.push({nombre, precio});
  localStorage.setItem("carrito", JSON.stringify(carrito));

  alert(nombre + " agregado al carrito");
}
</script>
