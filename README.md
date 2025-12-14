<!DOCTYPE html>
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
  max-width:480px;
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
  margin-top:10px;
  border:none;
  border-radius:12px;
  background:#c62828;
  color:#fff;
  font-size:16px;
  cursor:pointer;
}
.excel{ background:#2e7d32 }
.link{
  background:#f5f5f5;
  padding:12px;
  margin-top:12px;
  border-radius:12px;
}
</style>
</head>

<body>
<div class="card" id="card">
<h2>🎄 Amigo Oculto</h2>

<div id="setup">
<textarea id="dados" placeholder="Nome,email@email.com (um por linha)"></textarea>
<button onclick="criarSorteio()">🎁 Criar Sorteio</button>
</div>

<div id="links"></div>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import { getFirestore, doc, setDoc, getDoc } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyCgV3hmEfZX8dzPMxNoFiC9YURNboJkWf4",
  authDomain: "amigo-oculto-af918.firebaseapp.com",
  projectId: "amigo-oculto-af918",
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

const ADMIN = "clx";

// 🔀 SORTEIO ALEATÓRIO
function shuffle(arr){
  let a=[...arr];
  for(let i=a.length-1;i>0;i--){
    const j=Math.floor(Math.random()*(i+1));
    [a[i],a[j]]=[a[j],a[i]];
  }
  return a;
}

// 🎁 CRIAR SORTEIO
window.criarSorteio = async ()=>{
  const linhas = dados.value.split("\n").map(l=>l.trim()).filter(Boolean);
  if(linhas.length < 2){ alert("Mínimo 2 participantes"); return; }

  const base = linhas.map(l=>{
    const [nome,email]=l.split(",");
    return {nome:nome.trim(), email:email.trim()};
  });

  const nomes = base.map(p=>p.nome);
  let sorteados;
  do { sorteados = shuffle(nomes); }
  while(!nomes.every((n,i)=>n !== sorteados[i]));

  const participantes = {};
  base.forEach((p,i)=>{
    participantes[crypto.randomUUID()] = {
      nome:p.nome,
      email:p.email,
      senha:Math.random().toString(36).substring(2,8).toUpperCase(),
      resultado:sorteados[i],
      visto:false
    };
  });

  const sorteioID = crypto.randomUUID();
  await setDoc(doc(db,"sorteios",sorteioID), participantes);
  renderizarLinks(sorteioID, participantes);
};

// 📧 TELA ADMIN
function renderizarLinks(id, part){
  setup.style.display="none";
  links.innerHTML="";

  const linkExcel = `${location.origin}${location.pathname}?excel=${id}&admin=${ADMIN}`;

  links.innerHTML += `
    <button class="excel" onclick="window.open('${linkExcel}')">
      📥 Baixar Excel Completo
    </button>
  `;

  for(let pid in part){
    const p = part[pid];
    const linkPessoa = `${location.origin}${location.pathname}?s=${id}&p=${pid}`;

    const mensagem = `
🎄 Olá ${p.nome}!

O sorteio do Amigo Oculto já aconteceu 🎁

👉 Descubra quem você tirou:
${linkPessoa}

🔐 Senha: ${p.senha}

Guarde segredo 🤫
`;

    links.innerHTML += `
      <div class="link">
        <b>${p.nome}</b><br>${p.email}<br><br>
        <a href="mailto:${p.email}?subject=🎄 Amigo Oculto&body=${encodeURIComponent(mensagem)}">
          <button>📧 Enviar e-mail</button>
        </a>
      </div>
    `;
  }
}

// 👤 PARTICIPANTE
const params = new URLSearchParams(location.search);
if(params.get("s") && params.get("p")){
  const snap = await getDoc(doc(db,"sorteios",params.get("s")));
  const dados = snap.data();
  const p = dados?.[params.get("p")];

  if(!p){
    card.innerHTML="<h2>Link inválido</h2>";
  } else {
    card.innerHTML=`
      <h2>🔐 Área Segura</h2>
      <input id="senha" placeholder="Senha">
      <button id="ver">Ver resultado</button>
      <div id="res"></div>
    `;
    ver.onclick = async ()=>{
      if(senha.value !== p.senha){ alert("Senha incorreta"); return; }
      p.visto = true;
      await setDoc(doc(db,"sorteios",params.get("s")), dados);
      res.innerHTML = `<h2>🎉 ${p.resultado}</h2>`;
    };
  }
}

// 📥 EXCEL COMPLETO
if(params.get("excel") && params.get("admin") === ADMIN){
  const snap = await getDoc(doc(db,"sorteios",params.get("excel")));
  const dados = snap.data();

  const linhas = [["Nome","Email","Quem Tirou","Senha","Link","Já viu"]];
  for(let id in dados){
    const p = dados[id];
    const link = `${location.origin}${location.pathname}?s=${params.get("excel")}&p=${id}`;
    linhas.push([
      p.nome,
      p.email,
      p.resultado,
      p.senha,
      link,
      p.visto ? "SIM" : "NÃO"
    ]);
  }

  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, XLSX.utils.aoa_to_sheet(linhas),"Sorteio");
  XLSX.writeFile(wb,"amigo-oculto-completo.xlsx");
}
</script>
</body>
</html>
