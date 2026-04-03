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
      const precio = parseFloat(cols[4]);

      if(nombre){
        html += `
          <div style="border:1px solid #ddd; padding:15px; margin:10px;">
            <img src="${foto}" width="150"><br>
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

function actualizarCarrito(){
  let carrito = JSON.parse(localStorage.getItem("carrito")) || [];

  let total = 0;
  let items = "";

  carrito.forEach(p => {
    total += p.precio * p.cantidad;
    items += `• ${p.nombre} x${p.cantidad} = Q${(p.precio * p.cantidad).toFixed(2)}%0A`;
  });

  document.getElementById("carrito").innerHTML = `
    <h3>🛒 Carrito (${carrito.length})</h3>
    <p>Total: <b>Q${total.toFixed(2)}</b></p>
    <a href="https://wa.me/TUNUMERO?text=Hola,%20quiero%20pedir:%0A${items}%0ATotal:%20Q${total.toFixed(2)}"
       target="_blank"
       style="background:green;color:white;padding:10px;display:inline-block;">
       Pedir por WhatsApp
    </a>
  `;
}
</script>
