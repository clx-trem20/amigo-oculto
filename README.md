<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>🎄 Amigo Oculto Natalino</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

<style>
*{box-sizing:border-box}
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
  max-width:420px;
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
button:hover{opacity:.9}
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
<textarea id="nomes" placeholder="Digite um nome por linha"></textarea>
<button onclick="criarSorteio()">🎁 Criar Sorteio</button>
<button onclick="mostrarHistorico()">🗂️ Histórico</button>
<button onclick="exportarHistoricoExcel()">📥 Exportar Histórico (Excel)</button>
</div>

<div id="links"></div>
</div>

<script>
// ================= CONFIG =================
const ADMIN = "clx";
const SECRET_KEY = "AMIGO_OCULTO_CLX_2025";
let sorteioAtual = null;

// ================= CRIPTO =================
function encrypt(text){
  return btoa(
    text.split("")
    .map((c,i)=>String.fromCharCode(
      c.charCodeAt(0) ^ SECRET_KEY.charCodeAt(i % SECRET_KEY.length)
    )).join("")
  );
}
function decrypt(text){
  return atob(text)
    .split("")
    .map((c,i)=>String.fromCharCode(
      c.charCodeAt(0) ^ SECRET_KEY.charCodeAt(i % SECRET_KEY.length)
    )).join("");
}

// ================= SHUFFLE REAL =================
function shuffle(arr){
  let a=[...arr];
  for(let i=a.length-1;i>0;i--){
    const j=Math.floor(Math.random()*(i+1));
    [a[i],a[j]]=[a[j],a[i]];
  }
  return a;
}

// ================= SORTEIO =================
function criarSorteio(){
  const nomes=document.getElementById("nomes").value
    .split("\n").map(n=>n.trim()).filter(n=>n);

  if(nomes.length<2) return alert("Mínimo 2 participantes");

  let sorteados;
  do{
    sorteados=shuffle(nomes);
  }while(!nomes.every((n,i)=>n!==sorteados[i]));

  let participantes={};
  nomes.forEach((nome,i)=>{
    const pid=crypto.randomUUID();
    participantes[pid]={
      nome:encrypt(nome),
      senha:encrypt(Math.random().toString(36).substring(2,8).toUpperCase()),
      resultado:encrypt(sorteados[i]),
      visto:false
    };
  });

  sorteioAtual=crypto.randomUUID();
  salvarSorteio(sorteioAtual,participantes);
  renderizarLinks(sorteioAtual,participantes);
}

// ================= STORAGE =================
function salvarSorteio(id,participantes){
  localStorage.setItem("sorteio_"+id,JSON.stringify(participantes));
  const hist=JSON.parse(localStorage.getItem("historico"))||[];
  hist.push({id,data:new Date().toLocaleString(),participantes});
  localStorage.setItem("historico",JSON.stringify(hist));
}

// ================= EMAIL =================
function renderizarLinks(id,participantes){
  setup.style.display="none";
  links.innerHTML=`<button onclick="baixarExcel()">📥 Baixar Excel deste sorteio</button>`;

  for(let pid in participantes){
    const p=participantes[pid];
    const nome=decrypt(p.nome);
    const senha=decrypt(p.senha);
    const link=location.href.split("?")[0]+`?s=${id}&p=${pid}`;

    const assunto = "🎄 Seu Amigo Oculto chegou!";
    const corpo = `Olá ${nome}!

Que alegria ter você nesse amigo oculto 🎁✨

🔐 Sua senha: ${senha}

Clique no link abaixo para descobrir quem você tirou 🤫
${link}

🤫 Guarde segredo!
Desejo um Natal cheio de amor e boas surpresas 🎅❤️`;

    const mailto = `mailto:?subject=${encodeURIComponent(assunto)}&body=${encodeURIComponent(corpo)}`;

    links.innerHTML+=`
      <div class="link">
        ${nome}<br>
        <a href="${mailto}">
          📧 Enviar por E-mail
        </a>
      </div>`;
  }
}

// ================= EXCEL =================
function baixarExcel(){
  if(prompt("Senha do administrador:")!==ADMIN) return alert("Senha incorreta");
  const dados=JSON.parse(localStorage.getItem("sorteio_"+sorteioAtual));
  const linhas=[["Nome","Senha","Amigo Oculto","Visualizado","Link"]];
  for(let pid in dados){
    const p=dados[pid];
    linhas.push([
      decrypt(p.nome),
      decrypt(p.senha),
      decrypt(p.resultado),
      p.visto?"Sim":"Não",
      location.href.split("?")[0]+`?s=${sorteioAtual}&p=${pid}`
    ]);
  }
  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,XLSX.utils.aoa_to_sheet(linhas),"Sorteio");
  XLSX.writeFile(wb,"amigo-oculto.xlsx");
}

// ================= HISTÓRICO =================
function exportarHistoricoExcel(){
  if(prompt("Senha do administrador:")!==ADMIN) return alert("Senha incorreta");
  const hist=JSON.parse(localStorage.getItem("historico"))||[];
  if(!hist.length) return alert("Sem histórico");

  const linhas=[["Data","Nome","Amigo Oculto","Senha","Visualizado","Link"]];
  hist.forEach(h=>{
    for(let pid in h.participantes){
      const p=h.participantes[pid];
      linhas.push([
        h.data,
        decrypt(p.nome),
        decrypt(p.resultado),
        decrypt(p.senha),
        p.visto?"Sim":"Não",
        location.href.split("?")[0]+`?s=${h.id}&p=${pid}`
      ]);
    }
  });

  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,XLSX.utils.aoa_to_sheet(linhas),"Histórico");
  XLSX.writeFile(wb,"historico-amigo-oculto.xlsx");
}

function mostrarHistorico(){
  if(prompt("Senha do administrador:")!==ADMIN) return alert("Senha incorreta");
  const hist=JSON.parse(localStorage.getItem("historico"))||[];
  if(!hist.length) return alert("Sem sorteios");

  let lista="";
  hist.forEach((h,i)=>lista+=`${i+1} - ${h.data}\n`);
  const idx=parseInt(prompt(lista+"\nDigite o número:"))-1;
  if(!hist[idx]) return;

  sorteioAtual=hist[idx].id;
  renderizarLinks(sorteioAtual,hist[idx].participantes);
}

// ================= PARTICIPANTE =================
const params=new URLSearchParams(location.search);
if(params.get("s")&&params.get("p")){
  const dados=JSON.parse(localStorage.getItem("sorteio_"+params.get("s")));
  const p=dados?.[params.get("p")];

  if(!p) card.innerHTML="<h2>Link inválido</h2>";
  else if(p.visto) card.innerHTML="<h2>⛔ Já visualizado</h2>";
  else{
    card.innerHTML=`
      <h2>🔒 Área Segura</h2>
      <p>${decrypt(p.nome)}</p>
      <input type="password" id="senha" placeholder="Senha">
      <button onclick="ver()">Ver Resultado</button>
      <div id="res"></div>`;
    window.ver=()=>{
      if(senha.value!==decrypt(p.senha)) return alert("Senha incorreta");
      p.visto=true;
      localStorage.setItem("sorteio_"+params.get("s"),JSON.stringify(dados));
      res.innerHTML=`<h3>🎉 Você tirou:</h3><h2>${decrypt(p.resultado)}</h2>`;
    }
  }
}
</script>
</body>
</html>
