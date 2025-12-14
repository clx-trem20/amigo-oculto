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
  max-width:430px;
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
</style>
</head>

<body>
<div class="card" id="card">
<h2 style="text-align:center">🎄 Amigo Oculto</h2>

<div id="setup">
<textarea id="dados" placeholder="Nome,email@email.com,Categoria"></textarea>
<button id="btnSortear">🎁 Criar Sorteio</button>
<button id="btnExcel">📥 Exportar Excel</button>
</div>

<div id="links"></div>
</div>

<script>
const ADMIN = "clx";
let sorteioAtual = null;

const setupEl = document.getElementById("setup");
const linksEl = document.getElementById("links");
const dadosEl = document.getElementById("dados");

function shuffle(arr){
  let a=[...arr];
  for(let i=a.length-1;i>0;i--){
    const j=Math.floor(Math.random()*(i+1));
    [a[i],a[j]]=[a[j],a[i]];
  }
  return a;
}

document.getElementById("btnSortear").onclick = criarSorteio;
document.getElementById("btnExcel").onclick = exportarExcel;

function criarSorteio(){
  const linhas = dadosEl.value.split("\n").map(l=>l.trim()).filter(Boolean);
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
  renderizarLinks(sorteioAtual, participantes);
}

function renderizarLinks(id, participantes){
  setupEl.style.display="none";
  linksEl.innerHTML="";

  for(let pid in participantes){
    const p = participantes[pid];
    const link = location.href.split("#")[0] + `#s=${id}&p=${pid}`;

    const assunto = "🎄 Seu Amigo Oculto chegou!";
    const corpo =
`Olá ${p.nome}!

Categoria: ${p.categoria}

🔐 Senha: ${p.senha}

Descubra quem você tirou:
${link}

Guarde segredo 🎁`;

    const mailto = `mailto:${p.email}?subject=${encodeURIComponent(assunto)}&body=${encodeURIComponent(corpo)}`;

    linksEl.innerHTML += `
      <div class="link">
        <b>${p.nome}</b> (${p.categoria})<br>
        ${p.email}<br>
        <a href="${mailto}">📧 Enviar E-mail</a>
      </div>
    `;
  }
}

function exportarExcel(){
  if(prompt("Senha do administrador:") !== ADMIN){
    alert("Senha incorreta");
    return;
  }
  if(!sorteioAtual){
    alert("Nenhum sorteio");
    return;
  }

  const dados = JSON.parse(localStorage.getItem("sorteio_"+sorteioAtual));
  const linhas = [["Nome","Email","Categoria","Amigo Oculto","Senha","Link"]];

  for(let id in dados){
    const p = dados[id];
    const link = location.href.split("#")[0] + `#s=${sorteioAtual}&p=${id}`;
    linhas.push([p.nome,p.email,p.categoria,p.resultado,p.senha,link]);
  }

  const wb = XLSX.utils.book_new();
  const ws = XLSX.utils.aoa_to_sheet(linhas);
  XLSX.utils.book_append_sheet(wb, ws, "Sorteio");
  XLSX.writeFile(wb, "amigo-oculto.xlsx");
}

/* PARTICIPANTE */
if(location.hash){
  const params = new URLSearchParams(location.hash.replace("#",""));
  const dados = JSON.parse(localStorage.getItem("sorteio_"+params.get("s")));
  const p = dados?.[params.get("p")];

  if(p){
    document.getElementById("card").innerHTML = `
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
      document.getElementById("res").innerHTML =
        "<h3>🎉 Você tirou:</h3><h2>"+p.resultado+"</h2>";
    }
  }
}
</script>
</body>
</html>
