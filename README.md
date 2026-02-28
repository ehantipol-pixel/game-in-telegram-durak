# game-in-telegram-durak
mini game for telegram
<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Дурень — Telegram Mini Game</title>
<script src="https://telegram.org/js/telegram-web-app.js"></script>
<style>
body {
    margin:0;
    background:#0b5d1e;
    font-family:Arial;
    color:white;
    text-align:center;
}
h2 { margin:10px 0; }
.area { margin:10px; }
.cards { display:flex; justify-content:center; flex-wrap:wrap; gap:6px; }
.card {
    width:60px;
    height:90px;
    background:white;
    color:black;
    border-radius:8px;
    display:flex;
    align-items:center;
    justify-content:center;
    font-weight:bold;
    cursor:pointer;
}
.enemy .card {
    background:#333;
}
.trump { color:yellow; font-weight:bold; }
button {
    padding:10px 20px;
    margin:10px;
    border-radius:10px;
    border:none;
    font-size:16px;
}
</style>
</head>
<body>

<h2>🃏 Дурень (проти бота)</h2>
<div id="trump" class="trump"></div>

<div class="area">
    <h3>Стіл</h3>
    <div id="table" class="cards"></div>
</div>

<div class="area enemy">
    <h3>Бот</h3>
    <div id="enemy" class="cards"></div>
</div>

<div class="area">
    <h3>Твої карти</h3>
    <div id="player" class="cards"></div>
</div>

<button onclick="endTurn()">Завершити хід</button>

<script>
const tg = window.Telegram?.WebApp;
tg?.expand();

const suits = ["♠","♥","♦","♣"];
const ranks = ["6","7","8","9","10","J","Q","K","A"];

let deck = [];
let player = [];
let enemy = [];
let table = [];
let trump = "";

function createDeck() {
    deck = [];
    suits.forEach(s => {
        ranks.forEach((r,i) => {
            deck.push({suit:s, rank:r, value:i});
        });
    });
    deck.sort(() => Math.random()-0.5);
}

function drawCards(hand) {
    while(hand.length < 6 && deck.length > 0) {
        hand.push(deck.pop());
    }
}

function render() {
    document.getElementById("player").innerHTML="";
    document.getElementById("enemy").innerHTML="";
    document.getElementById("table").innerHTML="";

    player.forEach((c,i)=>{
        const el = document.createElement("div");
        el.className="card";
        el.innerText = c.rank + c.suit;
        el.onclick = ()=> playCard(i);
        document.getElementById("player").appendChild(el);
    });

    enemy.forEach(()=>{
        const el = document.createElement("div");
        el.className="card";
        el.innerText="🂠";
        document.getElementById("enemy").appendChild(el);
    });

    table.forEach(c=>{
        const el = document.createElement("div");
        el.className="card";
        el.innerText = c.rank + c.suit;
        document.getElementById("table").appendChild(el);
    });
}

function playCard(index){
    table.push(player[index]);
    player.splice(index,1);
    render();
    botMove();
}

function botMove(){
    if(table.length===0) return;
    const attack = table[table.length-1];

    let defendIndex = enemy.findIndex(c =>
        (c.suit===attack.suit && c.value>attack.value) ||
        (c.suit===trump && attack.suit!==trump)
    );

    if(defendIndex!==-1){
        table.push(enemy[defendIndex]);
        enemy.splice(defendIndex,1);
    } else {
        enemy = enemy.concat(table);
        table=[];
    }

    render();
}

function endTurn(){
    table=[];
    drawCards(player);
    drawCards(enemy);
    checkWin();
    render();
}

function checkWin(){
    if(player.length===0 && deck.length===0){
        alert("🎉 Ти виграв!");
        startGame();
    }
    if(enemy.length===0 && deck.length===0){
        alert("🤖 Бот виграв!");
        startGame();
    }
}

function startGame(){
    createDeck();
    trump = deck[0].suit;
    document.getElementById("trump").innerText = "Козир: " + trump;
    player=[];
    enemy=[];
    table=[];
    drawCards(player);
    drawCards(enemy);
    render();
}

startGame();
</script>
</body>
</html>