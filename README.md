<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>🎄 Amigo Oculto Online</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

<!-- Firebase -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import { getFirestore, doc, setDoc, getDoc, updateDoc, collection, getDocs } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyCgV3hmEfZX8dzPMxNoFiC9YURNboJkWf4",
  authDomain: "amigo-oculto-af918.firebaseapp.com",
  projectId: "amigo-oculto-af918",
  storageBucket: "amigo-oculto-af918.firebasestorage.app",
  messagingSenderId: "485780565818",
  appId: "1:485780565818:web:3d28376965576d57cc2a03"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

// ================= CONFIG =================
const ADMIN = "clx";

// ================= UTILS =================
function shuffle(arr){
  let a=[...arr];
  for(let i=a.length-1;i>0;i--){
    const j=Math.floor(Math.random()*(i+1));
    [a[i],a[j]]=[a[j],a[i]];
  }
  return a;
}

// ================= CRIAR SORTEIO =================
window.criarSorteio = async function(){
  const linhas = dados.value.split("\n").map(l=>l.trim()).filter(l=>l);
  if(linhas.length < 2){
    alert("Mínimo 2 participantes");
    return;
  }

  const base = linhas.map(l=>{
    const [nome,email] = l.split(",");
    return { nome:nome.trim(), email:email.trim() };
  });

  let nomes = base.map(p=>p.nome);
  let sorteados;
  do{
    sorteados = shuffle(nomes);
  }while(!nomes.every((n,i)=>n!==sorteados[i]));

  const sorteioId = crypto.randomUUID();
  const participantes = {};

  base.forEach((p,i)=>{
    const pid = crypto.randomUUID();
    participantes[pid] = {
      nome:p.nome,
      email:p.email,
      resultado:sorteados[i],
      senha:Math.random().toString(36).substring(2,8).toUpperCase(),
      visto:false
    };
  });

  await setDoc(doc(db,"sorteios",sorteioId),{
    criadoEm:new Date().toISOString(),
    participantes
  });

  mostrarLinks(sorteioId, participantes);
};

// ================= LINKS =================
function mostrarLinks(id, participantes){
  setup.style.display="none";
  links.innerHTML = "";

  for(const pid in participantes){
    const p = participantes[pid];
    const link = location.origin + location.pathname + `?s=${id}&p=${pid}`;

    links.innerHTML += `
      <div class="box">
        <b>${p.nome}</b><br>
        ${p.email}<br><br>
        <button onclick="navigator.clipboard.writeText('${link}')">📋 Copiar Link</button>
      </div>
    `;
  }

  links.innerHTML += `
    <button onclick="baixarExcel('${id}')">📥 Baixar Excel (Admin)</button>
  `;
}

// ================= EXCEL =================
window.baixarExcel = async function(id){
  const senha = prompt("Senha do administrador:");
  if(senha !== ADMIN) return alert("Senha incorreta");

  const snap = await getDoc(doc(db,"sorteios",id));
  const dados = snap.data().participantes;

  const linhas = [["Nome","Email","Resultado","Senha","Visualizado","Link"]];
  for(const pid in dados){
    const p = dados[pid];
    const link = location.origin + location.pathname + `?s=${id}&p=${pid}`;
    linhas.push([p.nome,p.email,p.resultado,p.senha,p.visto?"Sim":"Não",link]);
  }

  const wb = XLSX.utils.book_new();
  const ws = XLSX.utils.aoa_to_sheet(linhas);
  XLSX.utils.book_append_sheet(wb, ws, "Amigo Oculto");
  XLSX.writeFile(wb,"amigo-oculto.xlsx");
};

// ================= PARTICIPANTE =================
async function carregarParticipante(){
  const params = new URLSearchParams(location.search);
  if(!params.get("s") || !params.get("p")) return;

  const snap = await getDoc(doc(db,"sorteios",params.get("s")));
  if(!snap.exists()) return card.innerHTML="<h2>Link inválido</h2>";

  const dados = snap.data().participantes;
  const p = dados[params.get("p")];
  if(!p) return card.innerHTML="<h2>Link inválido</h2>";

  if(p.visto){
    card.innerHTML="<h2>⛔ Resultado já visualizado</h2>";
    return;
  }

  card.innerHTML = `
    <h2>🔒 Área Segura</h2>
    <p>${p.nome}</p>
    <input id="senha" placeholder="Senha">
    <button id="btn">Ver Resultado</button>
    <div id="res"></div>
  `;

  btn.onclick = async ()=>{
    if(senha.value !== p.senha) return alert("Senha incorreta");
    p.visto = true;
    await updateDoc(doc(db,"sorteios",params.get("s")),{ participantes:dados });
    res.innerHTML = `<h3>🎉 Você tirou:</h3><h2>${p.resultado}</h2>`;
  };
}

window.onload = carregarParticipante;
</script>

<style>
body{
  margin:0;
  background:linear-gradient(135deg,#b30000,#0f7a3a);
  font-family:Arial;
  display:flex;
  justify-content:center;
  align-items:center;
  min-height:100vh;
}
.card{
  background:#fff;
  width:100%;
  max-width:430px;
  padding:20px;
  border-radius:20px;
}
textarea,input,button{
  width:100%;
  padding:12px;
  margin-top:10px;
  border-radius:10px;
}
button{
  background:#c62828;
  color:#fff;
  border:none;
}
.box{
  background:#f5f5f5;
  padding:10px;
  border-radius:10px;
  margin-top:10px;
}
</style>
</head>

<body>
<div class="card" id="card">
<h2>🎄 Amigo Oculto</h2>

<div id="setup">
<textarea id="dados" placeholder="Nome,email@email.com"></textarea>
<button onclick="criarSorteio()">🎁 Sortear</button>
</div>

<div id="links"></div>
</div>
</body>
</html>
