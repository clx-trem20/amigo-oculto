<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>🎄 Amigo Oculto Natalino</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

<style>
body{
  margin:0;
  font-family:Arial, sans-serif;
  background:linear-gradient(135deg,#b30000,#0f7a3a);
  min-height:100vh;
  display:flex;
  justify-content:center;
  align-items:center;
}
.card{
  background:#fff;
  max-width:440px;
  width:100%;
  padding:20px;
  border-radius:18px;
  box-shadow:0 15px 40px rgba(0,0,0,.3);
}
h2{text-align:center;color:#b30000}
textarea,input{
  width:100%;
  padding:12px;
  margin-top:10px;
  border-radius:10px;
  border:1px solid #ccc;
}
button{
  width:100%;
  padding:14px;
  margin-top:12px;
  border:none;
  border-radius:12px;
  background:#c62828;
  color:#fff;
  font-size:16px;
  cursor:pointer;
}
.link{
  background:#f5f5f5;
  padding:10px;
  margin-top:10px;
  border-radius:10px;
}
a{color:#0f7a3a;font-weight:bold;text-decoration:none}
.small{font-size:13px;color:#555}
</style>
</head>

<body>
<div class="card" id="card">

<h2>🎄 Amigo Oculto</h2>

<div id="setup">
<textarea id="dados" placeholder="Nome,email@email.com (um por linha)"></textarea>

<button onclick="criarSorteio()">🎁 Criar Sorteio</button>
<button onclick="mostrarHistorico()">🗂️ Histórico</button>
<button onclick="exportarExcel()">📥 Baixar Excel</button>

<p class="small">* Exportação protegida por senha</p>
</div>

<div id="links"></div>

</div>

<script>
const ADMIN = "clx";
let sorteioAtual = null;

/* ===== URL SEGURA (iPhone FIX) ===== */
function baseURL(){
  return window.location.href.split("?")[0];
}

/* ===== SHUFFLE REAL ===== */
function shuffle(arr){
  let a = [...arr];
  for(let i=a.length-1;i>0;i--){
    const j = Math.floor(Math.random()*(i+1));
    [a[i],a[j]]=[a[j],a[i]];
  }
  return a;
}

/* ===== CRIAR SORTEIO ===== */
function criarSorteio(){
  const linhas = document.getElementById("dados").value
    .split("\n")
    .map(l=>l.trim())
    .filter(l=>l);

  if(linhas.length < 2){
    alert("Mínimo 2 participantes");
    return;
  }

  const base = linhas.map(l=>{
    const [nome,email] = l.split(",");
    if(!nome || !email) return null;
    return {nome:nome.trim(), email:email.trim()};
  }).filter(Boolean);

  const nomes = base.map(p=>p.nome);
  let sorteados;

  do{
    sorteados = shuffle(nomes);
  }while(!nomes.every((n,i)=>n !== sorteados[i]));

  const participantes = {};
  base.forEach((p,i)=>{
    const id = crypto.randomUUID();
    participantes[id] = {
      nome:p.nome,
      email:p.email,
      senha:Math.random().toString(36).substring(2,8).toUpperCase(),
      resultado:sorteados[i],
      visto:false
    };
  });

  sorteioAtual = crypto.randomUUID();
  localStorage.setItem("sorteio_"+sorteioAtual, JSON.stringify(participantes));

  const hist = JSON.parse(localStorage.getItem("historico")) || [];
  hist.push({id:sorteioAtual, data:new Date().toLocaleString(), participantes});
  localStorage.setItem("historico", JSON.stringify(hist));

  renderizarLinks(sorteioAtual, participantes);
}

/* ===== LINKS / EMAIL ===== */
function renderizarLinks(id, participantes){
  document.getElementById("setup").style.display="none";
  const links = document.getElementById("links");
  links.innerHTML="";

  for(let pid in participantes){
    const p = participantes[pid];
    const link = baseURL() + `?s=${id}&p=${pid}`;

    const assunto = "🎄 Seu Amigo Oculto chegou! 🎁✨";
    const corpo = `Olá ${p.nome}! 🎅✨

Que alegria ter você participando do nosso Amigo Oculto! 🎁❤️

🔐 Sua senha secreta: ${p.senha}

👉 Clique no link abaixo para descobrir quem você tirou:
${link}

🤫 Guarde segredo!
Feliz Natal 🎄💝`;

    const mailto = `mailto:${p.email}?subject=${encodeURIComponent(assunto)}&body=${encodeURIComponent(corpo)}`;

    links.innerHTML += `
      <div class="link">
        <strong>${p.nome}</strong><br>
        📧 ${p.email}<br><br>
        <a href="${mailto}">📨 Enviar por e-mail</a>
      </div>
    `;
  }
}

/* ===== HISTÓRICO ===== */
function mostrarHistorico(){
  if(prompt("Senha do administrador:") !== ADMIN){
    alert("Senha incorreta");
    return;
  }

  const hist = JSON.parse(localStorage.getItem("historico")) || [];
  if(!hist.length){
    alert("Nenhum histórico");
    return;
  }

  let txt = hist.map((h,i)=>`${i+1} - ${h.data}`).join("\n");
  const idx = parseInt(prompt(txt+"\nEscolha o número:")) - 1;
  if(!hist[idx]) return;

  sorteioAtual = hist[idx].id;
  renderizarLinks(sorteioAtual, hist[idx].participantes);
}

/* ===== EXCEL ===== */
function exportarExcel(){
  if(prompt("Senha do administrador:") !== ADMIN){
    alert("Senha incorreta");
    return;
  }

  if(!sorteioAtual){
    alert("Nenhum sorteio selecionado");
    return;
  }

  const dados = JSON.parse(localStorage.getItem("sorteio_"+sorteioAtual));
  const linhas = [["Nome","Email","Amigo Oculto","Senha","Visualizado","Link"]];

  for(let pid in dados){
    const p = dados[pid];
    const link = baseURL() + `?s=${sorteioAtual}&p=${pid}`;
    linhas.push([p.nome,p.email,p.resultado,p.senha,p.visto?"Sim":"Não",link]);
  }

  const wb = XLSX.utils.book_new();
  const ws = XLSX.utils.aoa_to_sheet(linhas);
  XLSX.utils.book_append_sheet(wb, ws, "Sorteio");
  XLSX.writeFile(wb, "amigo-oculto.xlsx");
}

/* ===== PARTICIPANTE ===== */
const params = new URLSearchParams(window.location.search);
if(params.get("s") && params.get("p")){
  const dados = JSON.parse(localStorage.getItem("sorteio_"+params.get("s")));
  const p = dados?.[params.get("p")];

  if(!p){
    card.innerHTML="<h2>Link inválido</h2>";
  }else if(p.visto){
    card.innerHTML="<h2>⛔ Já visualizado</h2>";
  }else{
    card.innerHTML=`
      <h2>🔒 Área Segura</h2>
      <p>${p.nome}</p>
      <input id="senha" type="password" placeholder="Senha">
      <button onclick="ver()">Ver Resultado</button>
      <div id="res"></div>
    `;
    window.ver = ()=>{
      if(senha.value !== p.senha){
        alert("Senha incorreta");
        return;
      }
      p.visto = true;
      localStorage.setItem("sorteio_"+params.get("s"), JSON.stringify(dados));
      res.innerHTML = `<h3>🎉 Você tirou:</h3><h2>${p.resultado}</h2>`;
    }
  }
}
</script>

</body>
</html>
