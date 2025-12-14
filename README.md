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
  max-width:460px;
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
</style>
</head>

<body>
<div class="card" id="card">
<h2>🎄 Amigo Oculto</h2>

<div id="setup">
<textarea id="dados" placeholder="Nome,email@email.com (um por linha)"></textarea>
<button onclick="criarSorteio()">🎁 Criar Sorteio</button>
<button onclick="mostrarHistorico()">🗂️ Histórico</button>
<button onclick="exportarExcel()">📥 Exportar Excel</button>
</div>

<div id="links"></div>
</div>

<!-- 🔥 FIREBASE -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import {
  getFirestore, doc, setDoc, getDoc, getDocs, collection
} from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

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

const ADMIN = "clx";
let sorteioAtual = null;

/* 🔀 Shuffle seguro */
function shuffle(arr){
  let a=[...arr];
  for(let i=a.length-1;i>0;i--){
    const j=Math.floor(Math.random()*(i+1));
    [a[i],a[j]]=[a[j],a[i]];
  }
  return a;
}

/* 🎁 Criar sorteio */
window.criarSorteio = async ()=>{
  const linhas = dados.value.split("\n").map(l=>l.trim()).filter(Boolean);
  if(linhas.length < 2){
    alert("Mínimo 2 participantes");
    return;
  }

  const base = linhas.map(l=>{
    const [nome,email]=l.split(",");
    return { nome:nome.trim(), email:email.trim() };
  });

  let nomes = base.map(p=>p.nome);
  let sorteados;
  do{
    sorteados = shuffle(nomes);
  }while(!nomes.every((n,i)=>n!==sorteados[i]));

  let participantes = {};
  base.forEach((p,i)=>{
    const id = crypto.randomUUID();
    participantes[id]={
      nome:p.nome,
      email:p.email,
      senha:Math.random().toString(36).substring(2,8).toUpperCase(),
      resultado:sorteados[i],
      visto:false
    };
  });

  sorteioAtual = crypto.randomUUID();
  await setDoc(doc(db,"sorteios",sorteioAtual),participantes);
  await setDoc(doc(db,"historico",sorteioAtual),{
    data:new Date().toLocaleString()
  });

  renderizarLinks(sorteioAtual, participantes);
};

/* 📧 Links de e-mail (iPhone OK) */
function renderizarLinks(id, part){
  setup.style.display="none";
  links.innerHTML="";

  for(let pid in part){
    const p = part[pid];
    const link = location.href.split("?")[0] + `?s=${id}&p=${pid}`;

    const assunto = "🎄 Seu Amigo Oculto chegou!";
    const corpo =
`Olá ${p.nome}!

Chegou o momento de espalhar alegria 🎁✨

🔐 Sua senha: ${p.senha}

Clique no link abaixo para descobrir quem você tirou:
${link}

Guarde segredo 🤫
Que seu Natal seja cheio de amor ❤️🎅`;

    links.innerHTML += `
      <div class="link">
        <strong>${p.nome}</strong><br>
        ${p.email}<br><br>
        <a href="mailto:${p.email}?subject=${encodeURIComponent(assunto)}&body=${encodeURIComponent(corpo)}">
          Enviar por e-mail
        </a>
      </div>
    `;
  }
}

/* 🗂️ Histórico */
window.mostrarHistorico = async ()=>{
  if(prompt("Senha do administrador:") !== ADMIN) return;

  const snap = await getDocs(collection(db,"historico"));
  let lista=[];
  snap.forEach(d=>lista.push(d.id));

  const escolha = prompt("ID do sorteio:\n"+lista.join("\n"));
  if(!escolha) return;

  const s = await getDoc(doc(db,"sorteios",escolha));
  sorteioAtual = escolha;
  renderizarLinks(escolha, s.data());
};

/* 📥 Exportar Excel */
window.exportarExcel = async ()=>{
  if(prompt("Senha do administrador:") !== ADMIN) return;
  if(!sorteioAtual){ alert("Nenhum sorteio ativo"); return; }

  const s = await getDoc(doc(db,"sorteios",sorteioAtual));
  const dados = s.data();

  const linhas=[["Nome","Email","Resultado","Senha","Visualizado"]];
  for(let i in dados){
    const p=dados[i];
    linhas.push([p.nome,p.email,p.resultado,p.senha,p.visto?"Sim":"Não"]);
  }

  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,XLSX.utils.aoa_to_sheet(linhas),"Sorteio");
  XLSX.writeFile(wb,"amigo-oculto.xlsx");
};

/* 👤 Tela do participante */
const params = new URLSearchParams(location.search);
if(params.get("s") && params.get("p")){
  const snap = await getDoc(doc(db,"sorteios",params.get("s")));
  const p = snap.data()?.[params.get("p")];

  if(!p){
    card.innerHTML="<h2>❌ Link inválido</h2>";
  }else if(p.visto){
    card.innerHTML="<h2>⛔ Resultado já visualizado</h2>";
  }else{
    card.innerHTML=`
      <h2>🔒 Área Segura</h2>
      <p>${p.nome}</p>
      <input id="senha" placeholder="Senha">
      <button id="ver">Ver Resultado</button>
      <div id="res"></div>
    `;
    ver.onclick = async ()=>{
      if(senha.value !== p.senha){
        alert("Senha incorreta");
        return;
      }
      p.visto = true;
      await setDoc(doc(db,"sorteios",params.get("s")), snap.data());
      res.innerHTML = `<h2>🎉 Você tirou:</h2><h3>${p.resultado}</h3>`;
    };
  }
}
</script>
</body>
</html>
