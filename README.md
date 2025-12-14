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
}
textarea,input,button{
  width:100%;
  padding:12px;
  margin-top:10px;
}
button{
  background:#c62828;
  color:#fff;
  border:none;
  border-radius:10px;
  cursor:pointer;
}
.link{
  background:#f5f5f5;
  padding:10px;
  margin-top:10px;
  border-radius:10px;
}
a{color:#0f7a3a;font-weight:bold;text-decoration:none}
</style>
</head>

<body>
<div class="card" id="card">
<h2 style="text-align:center">🎄 Amigo Oculto</h2>

<div id="setup">
<textarea id="dados" placeholder="Nome,email@email.com,Categoria (um por linha)"></textarea>
<button onclick="criarSorteio()">🎁 Criar Sorteio</button>
<button onclick="mostrarHistorico()">🗂️ Histórico</button>
<button onclick="exportarExcel()">📥 Baixar Resultados (Excel)</button>
</div>

<div id="links"></div>
</div>

<script>
const ADMIN = "clx";
let sorteioAtual = null;

const setup = document.getElementById("setup");
const links = document.getElementById("links");

function shuffle(arr){
  let a=[...arr];
  for(let i=a.length-1;i>0;i--){
    const j=Math.floor(Math.random()*(i+1));
    [a[i],a[j]]=[a[j],a[i]];
  }
  return a;
}

function criarSorteio(){
  const linhas = document.getElementById("dados").value
    .split("\n").map(l=>l.trim()).filter(Boolean);

  if(linhas.length < 2){
    alert("Mínimo 2 participantes");
    return;
  }

  const base = linhas.map(l=>{
    const [nome,email,categoria] = l.split(",");
    return {nome,email,categoria};
  });

  let nomes = base.map(p=>p.nome);
  let sorteados;

  do{
    sorteados = shuffle(nomes);
  }while(!nomes.every((n,i)=>n!==sorteados[i]));

  let participantes = {};
  base.forEach((p,i)=>{
    const id = crypto.randomUUID();
    participantes[id] = {
      ...p,
      senha: Math.random().toString(36).substring(2,8).toUpperCase(),
      resultado: sorteados[i],
      visto:false
    };
  });

  sorteioAtual = crypto.randomUUID();
  localStorage.setItem("sorteio_"+sorteioAtual, JSON.stringify(participantes));

  const hist = JSON.parse(localStorage.getItem("historico")) || [];
  hist.push({id:sorteioAtual,data:new Date().toLocaleString(),participantes});
  localStorage.setItem("historico", JSON.stringify(hist));

  renderizarLinks(sorteioAtual, participantes);
}

function renderizarLinks(id, participantes){
  setup.style.display="none";
  links.innerHTML="";

  for(let pid in participantes){
    const p = participantes[pid];
    const link = location.pathname + `?s=${id}&p=${pid}`;

    const assunto = "🎄 Seu Amigo Oculto chegou!";
    const corpo =
`🎄 Amigo Oculto 🎄

Olá ${p.nome}! ✨

Categoria: ${p.categoria}

Chegou o momento de espalhar alegria e boas surpresas 🎁❤️

🔐 Sua senha: ${p.senha}

Clique no link abaixo para descobrir quem você tirou:
${location.origin}${link}

🤫 Guarde segredo!
🎅 Que esse Natal seja cheio de amor e felicidade!`;

    const mailto = `mailto:${p.email}?subject=${encodeURIComponent(assunto)}&body=${encodeURIComponent(corpo)}`;

    links.innerHTML += `
      <div class="link">
        <b>${p.nome}</b> (${p.categoria})<br>
        ${p.email}<br>
        <a href="${mailto}">📧 Enviar E-mail</a>
      </div>
    `;
  }
}

function mostrarHistorico(){
  if(prompt("Senha do administrador:")!==ADMIN){
    alert("Senha incorreta"); return;
  }
  const hist = JSON.parse(localStorage.getItem("historico"))||[];
  if(!hist.length){ alert("Nenhum histórico"); return; }

  let texto = hist.map((h,i)=>`${i+1} - ${h.data}`).join("\n");
  const escolha = prompt(texto+"\nDigite o número:");
  const idx = parseInt(escolha)-1;
  if(!hist[idx]) return;

  sorteioAtual = hist[idx].id;
  renderizarLinks(sorteioAtual, hist[idx].participantes);
}

function exportarExcel(){
  if(prompt("Senha do administrador:")!==ADMIN){
    alert("Senha incorreta"); return;
  }
  if(!sorteioAtual){
    alert("Nenhum sorteio ativo"); return;
  }

  const dados = JSON.parse(localStorage.getItem("sorteio_"+sorteioAtual));
  const linhas = [["Nome","Email","Categoria","Amigo Oculto","Senha","Link"]];

  for(let id in dados){
    const p = dados[id];
    const link = location.origin + location.pathname + `?s=${sorteioAtual}&p=${id}`;
    linhas.push([p.nome,p.email,p.categoria,p.resultado,p.senha,link]);
  }

  const wb = XLSX.utils.book_new();
  const ws = XLSX.utils.aoa_to_sheet(linhas);
  XLSX.utils.book_append_sheet(wb, ws, "Resultados");
  XLSX.writeFile(wb, "amigo-oculto.xlsx");
}

/* PARTICIPANTE */
const params = new URLSearchParams(location.search);
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
      <input id="senha" placeholder="Senha">
      <button onclick="ver()">Ver Resultado</button>
      <div id="res"></div>
    `;
    window.ver=()=>{
      if(document.getElementById("senha").value!==p.senha){
        alert("Senha incorreta"); return;
      }
      p.visto=true;
      localStorage.setItem("sorteio_"+params.get("s"), JSON.stringify(dados));
      document.getElementById("res").innerHTML =
        `<h3>🎉 Você tirou:</h3><h2>${p.resultado}</h2>`;
    }
  }
}
</script>
</body>
</html>
