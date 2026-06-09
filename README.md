<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Texas Hold'em - Tilt Billiard Club</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>

    <div class="contenedor-juego">
        <h1>TEXAS HOLD'EM POKER</h1>
        
        <div class="mesa-poker">
            
            <div class="info-bote">BOTE: <span id="monto-bote">15</span>€</div>

            <div class="seccion-tablero">
                <h3>TABLERO (BOARD)</h3>
                <div class="cartas-comunitarias" id="board">
                    <div class="carta-vacia">?</div>
                    <div class="carta-vacia">?</div>
                    <div class="carta-vacia">?</div>
                    <div class="carta-vacia">?</div>
                    <div class="carta-vacia">?</div>
                </div>
                <div class="estado-calle" id="estado-calle">Pre-flop</div>
            </div>

            <div class="seccion-jugador">
                <h3>Tus Cartas (Hole Cards)</h3>
                <div class="cartas-propias" id="cartas-propias">
                    <div class="carta-vacia">?</div>
                    <div class="carta-vacia">?</div>
                </div>
            </div>

        </div>

        <div class="controles">
            <button onclick="repartirMano()">Repartir Mano</button>
            <button onclick="avanzarCalle()">Siguiente Fase</button>
        </div>
    </div>

    <script src="app.js"></script>
    body {
    background-color: #1a1a1a;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    color: white;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    margin: 0;
}

.contenedor-juego {
    text-align: center;
    width: 90%;
    max-width: 800px;
}

h1 {
    color: #e67e22;
    letter-spacing: 2px;
    margin-bottom: 20px;
}

/* El Tapete Verde de la Mesa */
.mesa-poker {
    background: radial-gradient(circle, #11998e, #38ef7d);
    border: 15px solid #3e2723; /* Borde de madera */
    border-radius: 150px;
    padding: 40px;
    box-shadow: 0px 10px 30px rgba(0,0,0,0.7);
    position: relative;
}

.info-bote {
    background-color: rgba(0, 0, 0, 0.6);
    padding: 8px 20px;
    border-radius: 20px;
    display: inline-block;
    font-weight: bold;
    font-size: 1.2rem;
    margin-bottom: 20px;
    border: 1px solid #gold;
}

.seccion-tablero, .seccion-jugador {
    margin: 20px 0;
}

h3 {
    margin: 5px 0;
    font-size: 0.9rem;
    text-transform: uppercase;
    color: #f1f1f1;
    opacity: 0.8;
}

/* Contenedores de cartas */
.cartas-comunitarias, .cartas-propias {
    display: flex;
    justify-content: center;
    gap: 15px;
    min-height: 120px;
    align-items: center;
}

/* Diseño de una Carta de Póker Física */
.carta {
    background-color: white;
    color: black;
    width: 80px;
    height: 115px;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding: 8px;
    box-sizing: border-box;
    font-weight: bold;
    font-size: 1.2rem;
    box-shadow: 3px 3px 10px rgba(0,0,0,0.4);
    transition: transform 0.2s;
}

.carta:hover {
    transform: translateY(-5px);
}

.carta.rojo {
    color: #e74c3c;
}

.carta.negro {
    color: #2c3e50;
}

.carta .icono-palo {
    font-size: 2rem;
    align-self: center;
}

.carta .palo-inferior {
    align-self: flex-end;
    transform: rotate(180deg);
}

/* Espacio vacío antes de repartir */
.carta-vacia {
    border: 2px dashed rgba(255,255,255,0.4);
    width: 80px;
    height: 115px;
    border-radius: 8px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 1.5rem;
    color: rgba(255,255,255,0.4);
}

.estado-calle {
    margin-top: 10px;
    font-style: italic;
    color: #ffeb3b;
}

/* Botones inferiores */
.controles {
    margin-top: 25px;
}

button {
    background-color: #e67e22;
    color: white;
    border: none;
    padding: 12px 25px;
    font-size: 1rem;
    border-radius: 5px;
    cursor: pointer;
    margin: 0 10px;
    font-weight: bold;
    box-shadow: 0 4px 6px rgba(0,0,0,0.3);
}

button:hover {
    background-color: #d35400;
}
// Configuración de los Palos y Símbolos según el reglamento de Tilt Club
const PALOS = [
    { nombre: 'Corazones', simbolo: '♥', color: 'rojo' },
    { nombre: 'Diamantes', simbolo: '♦', color: 'rojo' },
    { nombre: 'Tréboles', simbolo: '♣', color: 'negro' },
    { nombre: 'Picas', simbolo: '♠', color: 'negro' }
];
const VALORES = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A'];

let baraja = [];
let cartasMesa = [];
let cartasMio = [];
let faseActual = 0; // 0: Preflop, 1: Flop, 2: Turn, 3: River

// Inicializar la baraja mezclada
function crearBaraja() {
    baraja = [];
    for (let p of PALOS) {
        for (let v of VALORES) {
            baraja.push({ valor: v, palo: p });
        }
    }
    baraja.sort(() => Math.random() - 0.5);
}

// Genera el bloque HTML de la carta para pintarla en el tapete
function generarCartaHTML(carta) {
    return `
        <div class="carta ${carta.palo.color}">
            <div>${carta.valor}</div>
            <div class="icono-palo">${carta.palo.simbolo}</div>
            <div class="palo-inferior">${carta.valor}</div>
        </div>
    `;
}

function generarCartaVaciaHTML() {
    return `<div class="carta-vacia">?</div>`;
}

// Fase Inicial: Reparte tus 2 cartas privadas individuales (Hole Cards)
function repartirMano() {
    crearBaraja();
    cartasMesa = [];
    faseActual = 0;
    
    // Tomar 2 cartas para el jugador y 5 destinadas a la mesa
    cartasMio = [baraja.pop(), baraja.pop()];
    cartasMesa = [baraja.pop(), baraja.pop(), baraja.pop(), baraja.pop(), baraja.pop()];
    
    // Renderizar cartas propias
    document.getElementById('cartas-propias').innerHTML = cartasMio.map(generarCartaHTML).join('');
    
    // Resetear el tablero visual (vacío)
    document.getElementById('board').innerHTML = Array(5).fill(generarCartaVaciaHTML()).join('');
    document.getElementById('estado-calle').innerText = "Pre-flop: Las apuestas obligatorias están en juego.";
}

// Maneja los clicks para revelar las cartas comunitarias progresivamente
function avanzarCalle() {
    if (cartasMesa.length === 0) return alert("¡Primero haz click en 'Repartir Mano'!");

    let htmlBoard = "";
    faseActual++;

    if (faseActual === 1) {
        // EL FLOP: Se muestran las 3 primeras cartas comunitarias
        htmlBoard += generarCartaHTML(cartasMesa[0]);
        htmlBoard += generarCartaHTML(cartasMesa[1]);
        htmlBoard += generarCartaHTML(cartasMesa[2]);
        htmlBoard += generarCartaVaciaHTML() + generarCartaVaciaHTML();
        document.getElementById('estado-calle').innerText = "Flop: 3 cartas comunitarias en mesa.";
    } 
    else if (faseActual === 2) {
        // EL TURN: Se agrega la 4ta carta
        htmlBoard += generarCartaHTML(cartasMesa[0]);
        htmlBoard += generarCartaHTML(cartasMesa[1]);
        htmlBoard += generarCartaHTML(cartasMesa[2]);
        htmlBoard += generarCartaHTML(cartasMesa[3]);
        htmlBoard += generarCartaVaciaHTML();
        document.getElementById('estado-calle').innerText = "Turn (Cuarta calle): Se añade una carta.";
    } 
    else if (faseActual === 3) {
        // EL RIVER: Se revela la 5ta y última carta
        htmlBoard += cartasMesa.map(generarCartaHTML).join('');
        document.getElementById('estado-calle').innerText = "River (Quinta calle): Tablero completo. ¡Showdown!";
    } 
    else {
        alert("La mano ha terminado. Reparte una nueva mano.");
        return;
    }

    document.getElementById('board').innerHTML = htmlBoard;
}
</body>
</html>
