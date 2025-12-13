<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Amigo Oculto Seguro</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  font-family: "Segoe UI", Arial, sans-serif;
  background: linear-gradient(135deg, #1d976c, #93f9b9);
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.card {
  background: #fff;
  width: 100%;
  max-width: 420px;
  border-radius: 18px;
  padding: 22px;
  box-shadow: 0 20px 45px rgba(0,0,0,0.25);
}

h2 {
  text-align: center;
  margin-bottom: 10px;
}

p {
  text-align: center;
}

textarea, input {
  width: 100%;
  padding: 12px;
  margin-top: 10px;
  font-size: 16px;
  border-radius: 10px;
  border: 1px solid #ccc;
}

button {
  width: 100%;
  padding: 14px;
  margin-top: 15px;
  font-size: 18px;
  border: none;
  border-radius: 12px;
  background: #25d366;
  color: white;
  cursor: pointer;
}

button:hover { opacity: 0.9; }

.hidden { display: none; }

.link {
  margin-top: 12px;
  padding: 10px;
  background: #f4f4f4;
  border-radius: 10px;
  font-size: 14px;
}

.link a {
  display: inline-block;
  margin-top: 5px;
  color: #128c7e;
  font-weight: bold;
  text-decoration: none;
}

.result {
  text-align: center;
  margin-top: 15px;
}
</style>
</head>

<body>

<div class="card">
  <h2>🎁 Amigo Oculto</h2>

  <div id="setup">
    <p>Digite um nome por linha:</p>
    <textarea id="nomes" placeholder="João&#10;Maria&#10;Carlos&#10;Ana"></textarea>
    <button onclick="criarSorteio()">Criar Sorteio Seguro</button>
  </div>

  <div id="links" class="hidden"></div>
</div>

<script>
let sorteio = {};

function criarSorteio() {
  const nomes = document.getElementById("nomes").value
    .split("\n")
    .map(n => n.trim())
    .filter(n => n);

  if (nomes.length < 2) {
    alert("Digite pelo menos 2 nomes!");
    return;
  }

  let disponiveis = [...nomes];
  sorteio = {};

  for (let nome of nomes) {
    let possiveis = disponiveis.filter(n => n !== nome);
    if (possiveis.length === 0) return criarSorteio();

    let escolhido = possiveis[Math.floor(Math.random() * possiveis.length)];
    sorteio[nome] = escolhido;
    disponiveis.splice(disponiveis.indexOf(escolhido), 1);
  }

  document.getElementById("setup").classList.add("hidden");
  const div = document.getElementById("links");
  div.classList.remove("hidden");

  div.innerHTML = "<h3>🔒 Links Individuais</h3>";

  for (let pessoa in sorteio) {
    const senha = Math.random().toString(36).substring(2, 8).toUpperCase();

    const linkSeguro =
      location.href.split("?")[0] +
      `?nome=${encodeURIComponent(pessoa)}` +
      `&senha=${senha}` +
      `&alvo=${encodeURIComponent(sorteio[pessoa])}`;

    const mensagem =
`🎁 *Amigo Oculto* 🎁

Olá ${pessoa}!

Sua senha é: *${senha}*

Clique no link abaixo para descobrir quem você tirou 🤫👇
${linkSeguro}`;

    const linkWpp = "https://wa.me/?text=" + encodeURIComponent(mensagem);

    div.innerHTML += `
      <div class="link">
        <strong>${pessoa}</strong><br>
        <a href="${linkWpp}" target="_blank">📲 Enviar no WhatsApp</a>
      </div>
    `;
  }
}

/* Tela individual */
const params = new URLSearchParams(window.location.search);
if (params.get("nome")) {
  document.querySelector(".card").innerHTML = `
    <h2>🔐 Área Segura</h2>
    <p><strong>${params.get("nome")}</strong>, digite sua senha:</p>
    <input type="password" id="senha" placeholder="Senha">
    <button onclick="verResultado()">Ver Resultado</button>
    <div id="resposta" class="result"></div>
  `;
}

function verResultado() {
  const senhaDigitada = document.getElementById("senha").value;
  if (senhaDigitada === params.get("senha")) {
    document.getElementById("resposta").innerHTML = `
      🎉 Você tirou:<br><br>
      <h3>${params.get("alvo")}</h3>
      🤫 Não conte para ninguém!
    `;
  } else {
    alert("❌ Senha incorreta");
  }
}
</script>

</body>
</html>
