<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>🎄 Amigo Oculto</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  font-family: "Segoe UI", Arial, sans-serif;
  background: linear-gradient(135deg, #b30000, #0f7a3a);
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.card {
  background: #fff;
  width: 100%;
  max-width: 420px;
  border-radius: 20px;
  padding: 22px;
  box-shadow: 0 20px 45px rgba(0,0,0,.3);
}

h2 { text-align: center; color: #b30000; }
p { text-align: center; }

input, textarea {
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
  background: #c62828;
  color: white;
  cursor: pointer;
}

button:hover { opacity: 0.9; }
.hidden { display: none; }

.link {
  margin-top: 12px;
  padding: 10px;
  background: #f7f7f7;
  border-radius: 10px;
  font-size: 14px;
}

.link a {
  display: inline-block;
  margin-top: 5px;
  color: #0f7a3a;
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

<div class="card" id="card">

  <h2>🎄 Amigo Oculto</h2>

  <!-- SETUP -->
  <div id="setup">
    <input id="grupo" placeholder="Nome do grupo">
    <input id="valor" placeholder="Valor do presente (ex: R$50)">
    <input id="data" type="date">
    <textarea id="nomes" placeholder="Um nome por linha"></textarea>
    <button onclick="criarSorteio()">🎁 Criar Sorteio</button>
  </div>

  <!-- LINKS -->
  <div id="links" class="hidden"></div>

</div>

<script>
// ===============================
// CRIAR SORTEIO
// ===============================
function criarSorteio() {
  const grupo = document.getElementById("grupo").value;
  const valor = document.getElementById("valor").value;
  const data = document.getElementById("data").value;

  const nomes = document.getElementById("nomes").value
    .split("\n").map(n => n.trim()).filter(n => n);

  if (nomes.length < 2) {
    alert("Digite pelo menos 2 nomes");
    return;
  }

  const sorteioId = crypto.randomUUID();
  let disponiveis = [...nomes];
  let participantes = {};

  for (let nome of nomes) {
    let possiveis = disponiveis.filter(n => n !== nome);
    if (!possiveis.length) return criarSorteio();

    const escolhido = possiveis[Math.floor(Math.random() * possiveis.length)];
    const senha = Math.random().toString(36).substring(2,8).toUpperCase();
    const pid = crypto.randomUUID();

    participantes[pid] = {
      nome,
      senha,
      resultado: escolhido,
      visto: false
    };

    disponiveis.splice(disponiveis.indexOf(escolhido), 1);
  }

  const dados = { grupo, valor, data, participantes };
  localStorage.setItem("sorteio_" + sorteioId, JSON.stringify(dados));

  document.getElementById("setup").classList.add("hidden");
  const div = document.getElementById("links");
  div.classList.remove("hidden");

  div.innerHTML = `<h3>🔐 Links Individuais</h3>
                   <p><strong>Grupo:</strong> ${grupo}</p>
                   <p><strong>Valor:</strong> ${valor}</p>
                   <p><strong>Data:</strong> ${data}</p>`;

  for (let pid in participantes) {
    const p = participantes[pid];

    const link =
      location.href.split("?")[0] +
      `?sorteio=${sorteioId}&pid=${pid}`;

    const msg =
`🎄 Amigo Oculto 🎄

Grupo: ${grupo}
🎁 Valor: ${valor}
📅 Data: ${data}

Olá ${p.nome}!

🔐 Sua senha: ${p.senha}

Acesse o link abaixo para descobrir quem você tirou 🤫👇
${link}`;

    const wpp = "https://wa.me/?text=" + encodeURIComponent(msg);

    div.innerHTML += `
      <div class="link">
        <strong>${p.nome}</strong><br>
        <a href="${wpp}" target="_blank">📲 Enviar no WhatsApp</a>
      </div>`;
  }
}

// ===============================
// TELA DO PARTICIPANTE
// ===============================
const params = new URLSearchParams(location.search);
if (params.get("sorteio") && params.get("pid")) {
  const sorteio = JSON.parse(localStorage.getItem("sorteio_" + params.get("sorteio")));
  const p = sorteio?.participantes[params.get("pid")];

  if (!p) {
    document.getElementById("card").innerHTML = "<h2>❌ Link inválido</h2>";
  } else if (p.visto) {
    document.getElementById("card").innerHTML = "<h2>⛔ Você já visualizou</h2>";
  } else {
    document.getElementById("card").innerHTML = `
      <h2>🔒 Área Segura</h2>
      <p><strong>${p.nome}</strong>, digite sua senha:</p>
      <input id="senha" type="password">
      <button onclick="verResultado()">Ver Resultado</button>
      <div id="resposta" class="result"></div>
    `;

    window.verResultado = function () {
      if (document.getElementById("senha").value !== p.senha) {
        alert("Senha incorreta");
        return;
      }

      p.visto = true;
      localStorage.setItem("sorteio_" + params.get("sorteio"), JSON.stringify(sorteio));

      document.getElementById("resposta").innerHTML = `
        <h3>🎉 Você tirou:</h3>
        <h2>${p.resultado}</h2>
        <p>🤫 Guarde segredo!</p>
      `;
    };
  }
}
</script>

</body>
</html>
