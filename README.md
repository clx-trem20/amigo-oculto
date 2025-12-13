<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>🎄 Amigo Oculto Natalino</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- Biblioteca para gerar Excel -->
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

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
  box-shadow: 0 20px 45px rgba(0,0,0,0.3);
}

h2 {
  text-align: center;
  color: #b30000;
}

p { text-align: center; }

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
  color: #0f7a3a;
  font-weight: bold;
  text-decoration: none;
}
</style>
</head>

<body>

<div class="card" id="card">
  <h2>🎄 Amigo Oculto</h2>

  <!-- CRIAÇÃO -->
  <div id="setup">
    <p>Digite um nome por linha:</p>
    <textarea id="nomes" placeholder="Ex:\nAna\nCarlos\nJoão"></textarea>
    <button onclick="criarSorteio()">🎁 Criar Sorteio</button>
  </div>

  <!-- LINKS -->
  <div id="links" class="hidden"></div>
</div>

<script>
let sorteioId = null;

// ============================
// CRIAR SORTEIO
// ============================
function criarSorteio() {
  const nomes = document.getElementById("nomes").value
    .split("\n").map(n => n.trim()).filter(n => n);

  if (nomes.length < 2) {
    alert("Digite pelo menos 2 nomes");
    return;
  }

  let disponiveis = [...nomes];
  let participantes = {};

  for (let nome of nomes) {
    let possiveis = disponiveis.filter(n => n !== nome);
    if (!possiveis.length) return criarSorteio();

    let escolhido = possiveis[Math.floor(Math.random() * possiveis.length)];
    let senha = Math.random().toString(36).substring(2,8).toUpperCase();
    let pid = crypto.randomUUID();

    participantes[pid] = {
      nome,
      senha,
      resultado: escolhido,
      visto: false
    };

    disponiveis.splice(disponiveis.indexOf(escolhido), 1);
  }

  sorteioId = crypto.randomUUID();
  localStorage.setItem("sorteio_" + sorteioId, JSON.stringify(participantes));

  document.getElementById("setup").classList.add("hidden");
  const div = document.getElementById("links");
  div.classList.remove("hidden");

  div.innerHTML = `
    <h3>🔐 Links Individuais</h3>
    <button onclick="baixarExcel()">📥 Baixar lista (Excel)</button>
  `;

  for (let pid in participantes) {
    const p = participantes[pid];
    const link = location.href.split("?")[0] + `?s=${sorteioId}&p=${pid}`;

    const msg =
`🎄 Amigo Oculto 🎄

Olá ${p.nome}!
🔑 Sua senha: ${p.senha}

Acesse o link abaixo para descobrir quem você tirou 🤫👇
${link}`;

    const wpp = "https://wa.me/?text=" + encodeURIComponent(msg);

    div.innerHTML += `
      <div class="link">
        <strong>${p.nome}</strong><br>
        <a href="${wpp}" target="_blank">📲 Enviar WhatsApp</a>
      </div>`;
  }
}

// ============================
// BAIXAR EXCEL
// ============================
function baixarExcel() {
  const dados = JSON.parse(localStorage.getItem("sorteio_" + sorteioId));
  if (!dados) return alert("Sorteio não encontrado");

  const linhas = [["Nome", "Senha", "Amigo Oculto", "Visualizado"]];

  for (let id in dados) {
    const p = dados[id];
    linhas.push([
      p.nome,
      p.senha,
      p.resultado,
      p.visto ? "Sim" : "Não"
    ]);
  }

  const wb = XLSX.utils.book_new();
  const ws = XLSX.utils.aoa_to_sheet(linhas);
  XLSX.utils.book_append_sheet(wb, ws, "Sorteio");
  XLSX.writeFile(wb, "amigo-oculto.xlsx");
}

// ============================
// TELA DO PARTICIPANTE
// ============================
const params = new URLSearchParams(location.search);

if (params.get("s") && params.get("p")) {
  const dados = JSON.parse(localStorage.getItem("sorteio_" + params.get("s")));
  const p = dados?.[params.get("p")];

  if (!p) {
    card.innerHTML = "<h2>❌ Link inválido</h2>";
  } else if (p.visto) {
    card.innerHTML = "<h2>⛔ Você já visualizou</h2>";
  } else {
    card.innerHTML = `
      <h2>🔒 Área Segura</h2>
      <p>${p.nome}, digite sua senha:</p>
      <input type="password" id="senha">
      <button onclick="verResultado()">Ver Resultado</button>
      <div id="resposta"></div>
    `;

    window.verResultado = function () {
      if (document.getElementById("senha").value !== p.senha) {
        alert("Senha incorreta");
        return;
      }
      p.visto = true;
      localStorage.setItem("sorteio_" + params.get("s"), JSON.stringify(dados));
      document.getElementById("resposta").innerHTML =
        `<h3>🎉 Você tirou:</h3><h2>${p.resultado}</h2><p>🤫 Guarde segredo!</p>`;
    };
  }
}
</script>

</body>
</html>
