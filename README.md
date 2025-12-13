<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>🎄 Amigo Oculto Natalino</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- Excel -->
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

h2 { text-align: center; color: #b30000; }
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

.hidden { display: none; }

.link {
  margin-top: 12px;
  padding: 10px;
  background: #f7f7f7;
  border-radius: 10px;
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

  <!-- CRIAR -->
  <div id="setup">
    <p>Digite um nome por linha:</p>
    <textarea id="nomes" placeholder="Ana\nCarlos\nJoão"></textarea>
    <button onclick="criarSorteio()">🎁 Criar Sorteio</button>

    <button onclick="mostrarHistorico()">🗂️ Ver Histórico</button>
  </div>

  <!-- LINKS -->
  <div id="links" class="hidden"></div>
</div>

<script>
const ADMIN_SENHA = "clx";
let sorteioIdAtual = null;

// =======================
// CRIAR SORTEIO
// =======================
function criarSorteio() {
  const nomes = document.getElementById("nomes").value
    .split("\n").map(n => n.trim()).filter(n => n);

  if (nomes.length < 2) return alert("Digite pelo menos 2 nomes");

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

  sorteioIdAtual = crypto.randomUUID();
  localStorage.setItem("sorteio_" + sorteioIdAtual, JSON.stringify(participantes));

  salvarHistorico(sorteioIdAtual);

  document.getElementById("setup").classList.add("hidden");
  const div = document.getElementById("links");
  div.classList.remove("hidden");

  div.innerHTML = `
    <h3>🔐 Links Individuais</h3>
    <button onclick="baixarExcelProtegido()">📥 Baixar Excel (Admin)</button>
  `;

  for (let pid in participantes) {
    const p = participantes[pid];
    const link = location.href.split("?")[0] + `?s=${sorteioIdAtual}&p=${pid}`;

    const mensagem = 
`🎄 Amigo Oculto 🎄

Olá ${p.nome}! ✨

Chegou o momento de espalhar alegria, carinho e boas surpresas 🎁❤️  
Preparamos este amigo oculto com muito cuidado especialmente para você!

🔐 Sua senha: ${p.senha}

Clique no link abaixo para descobrir quem você tirou 🤫👇
${link}

🎅 Que esse Natal seja cheio de amor, risadas e bons presentes!`;

    const wpp = "https://wa.me/?text=" + encodeURIComponent(mensagem);

    div.innerHTML += `
      <div class="link">
        <strong>${p.nome}</strong><br>
        <a href="${wpp}" target="_blank">📲 Enviar WhatsApp</a>
      </div>`;
  }
}

// =======================
// ADMIN + EXCEL
// =======================
function baixarExcelProtegido() {
  const senha = prompt("Digite a senha de administrador:");
  if (senha !== ADMIN_SENHA) return alert("Senha incorreta");

  const dados = JSON.parse(localStorage.getItem("sorteio_" + sorteioIdAtual));
  if (!dados) return alert("Sorteio não encontrado");

  const linhas = [["Nome", "Senha", "Amigo Oculto", "Visualizado"]];

  for (let id in dados) {
    const p = dados[id];
    linhas.push([p.nome, p.senha, p.resultado, p.visto ? "Sim" : "Não"]);
  }

  const wb = XLSX.utils.book_new();
  const ws = XLSX.utils.aoa_to_sheet(linhas);
  XLSX.utils.book_append_sheet(wb, ws, "Sorteio");
  XLSX.writeFile(wb, "amigo-oculto.xlsx");
}

// =======================
// HISTÓRICO
// =======================
function salvarHistorico(id) {
  const hist = JSON.parse(localStorage.getItem("historico")) || [];
  hist.push({ id, data: new Date().toLocaleString() });
  localStorage.setItem("historico", JSON.stringify(hist));
}

function mostrarHistorico() {
  const hist = JSON.parse(localStorage.getItem("historico")) || [];
  if (!hist.length) return alert("Nenhum sorteio salvo");

  let texto = "📦 HISTÓRICO DE SORTEIOS:\n\n";
  hist.forEach((h, i) => {
    texto += `${i+1}) ${h.data}\nID: ${h.id}\n\n`;
  });

  alert(texto);
}

// =======================
// PARTICIPANTE
// =======================
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
