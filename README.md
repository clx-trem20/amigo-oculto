<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>🎄 Amigo Oculto</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

<style>
body{
  margin:0;
  font-family:Arial;
  background:linear-gradient(135deg,#b30000,#0f7a3a);
  min-height:100vh;
  display:flex;
  justify-content:center;
  align-items:center;
}
.card{
  background:#fff;
  width:100%;
  max-width:450px;
  padding:20px;
  border-radius:18px;
  box-shadow:0 15px 40px rgba(0,0,0,.3);
}
textarea,input,button{
  width:100%;
  padding:12px;
  margin-top:10px;
  border-radius:10px;
}
button{
  border:none;
  background:#c62828;
  color:#fff;
  font-size:16px;
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
<h2>🎄 Amigo Oculto</h2>

<div id="setup">
<textarea id="dados" placeholder="Nome,email@email.com"></textarea>
<button onclick="criarSorteio()">🎁 Criar Sorteio</button>
<button onclick="mostrarHistorico()">🗂️ Histórico</button>
<button onclick="exportarExcel()">📥 Exportar Excel</button>
</div>

<div id="links"></div>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import {
  getFirestore, doc, setDoc, getDoc, getDocs, collection
} from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyCgV3hmEfZX8dzPMxNoFiC9YURNboJkWf4",
  authDomain: "amigo-oculto-af918.firebaseapp.com",
  projectId: "amigo-oculto-af918"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
const ADMIN="clx";
let sorteioAtual=null;

function baseURL(){
  return window.location.href.split("?")[0];
}

function shuffle(a){
  let b=[...a];
  for(let i=b.length-1;i>0;i--){
    const j=Math.floor(Math.random()*(i+1));
    [b[i],b[j]]=[b[j],b[i]];
  }
  return b;
}

window.criarSorteio = async ()=>{
  const linhas=dados.value.split("\n").filter(Boolean);
  if(linhas.length<2){alert("Mínimo 2");return;}

  const base=linhas.map(l=>{
    const [n,e]=l.split(",");
    return {nome:n.trim(),email:e.trim()};
  });

  let nomes=base.map(p=>p.nome), sorteados;
  do{sorteados=shuffle(nomes);}while(!nomes.every((n,i)=>n!==sorteados[i]));

  let participantes={};
  base.forEach((p,i)=>{
    participantes[crypto.randomUUID()]={
      ...p,
      senha:Math.random().toString(36).substring(2,8).toUpperCase(),
      resultado:sorteados[i],
      visto:false
    };
  });

  sorteioAtual=crypto.randomUUID();
  await setDoc(doc(db,"sorteios",sorteioAtual),participantes);
  await setDoc(doc(db,"historico",sorteioAtual),{data:new Date().toLocaleString()});

  renderizarLinks(sorteioAtual,participantes);
};

function renderizarLinks(id,part){
  setup.style.display="none";
  links.innerHTML="";
  for(let pid in part){
    const p=part[pid];
    const link=baseURL()+`?s=${id}&p=${pid}`;
    links.innerHTML+=`
      <div class="link">
        <b>${p.nome}</b><br>${p.email}<br><br>
        <a href="mailto:${p.email}?subject=🎄 Amigo Oculto&body=Senha: ${p.senha}%0A${link}">
        Enviar por e-mail</a>
      </div>`;
  }
}

window.mostrarHistorico = async ()=>{
  if(prompt("Senha")!==ADMIN) return;
  const snap=await getDocs(collection(db,"historico"));
  let ids=[]; snap.forEach(d=>ids.push(d.id));
  const id=prompt(ids.join("\n"));
  if(!id) return;
  const s=await getDoc(doc(db,"sorteios",id));
  sorteioAtual=id;
  renderizarLinks(id,s.data());
};

window.exportarExcel = async ()=>{
  if(prompt("Senha")!==ADMIN) return;
  const s=await getDoc(doc(db,"sorteios",sorteioAtual));
  const dados=s.data();
  const linhas=[["Nome","Email","Resultado","Senha"]];
  Object.values(dados).forEach(p=>{
    linhas.push([p.nome,p.email,p.resultado,p.senha]);
  });
  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,XLSX.utils.aoa_to_sheet(linhas),"Sorteio");
  XLSX.writeFile(wb,"amigo-oculto.xlsx");
};

async function init(){
  const params=new URLSearchParams(location.search);
  if(!params.get("s")) return;

  const snap=await getDoc(doc(db,"sorteios",params.get("s")));
  const dados=snap.data();
  const p=dados?.[params.get("p")];
  if(!p){card.innerHTML="<h2>Link inválido</h2>";return;}

  card.innerHTML=`
    <h2>Área segura</h2>
    <input id="senha" placeholder="Senha">
    <button id="ver">Ver</button>
    <div id="res"></div>`;

  ver.onclick=async()=>{
    if(senha.value!==p.senha){alert("Senha errada");return;}
    p.visto=true;
    await setDoc(doc(db,"sorteios",params.get("s")),dados);
    res.innerHTML="<h2>🎉 "+p.resultado+"</h2>";
  };
}
init();
</script>
</body>
</html>
