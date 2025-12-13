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
  max-width:420px;
  width:100%;
  padding:20px;
  border-radius:18px;
}
h2{text-align:center;color:#b30000}
textarea,input{
  width:100%;
  padding:12px;
  margin-top:10px;
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
<textarea id="nomes" placeholder="Digite um nome por linha"></textarea>
<button onclick="criarSorteio()">🎁 Criar Sorteio</button>
<button onclick="mostrarHistorico()">🗂️ Histórico</button>
<button onclick="exportarHistoricoExcel()">📥 Exportar Histórico (Excel)</button>
</div>

<div id="links"></div>
</div>

<script>
const ADMIN="clx";
let sorteioAtual=null;

function criarSorteio(){
  const nomes=document.getElementById("nomes").value
  .split("\n").map(n=>n.trim()).filter(n=>n);
  if(nomes.length<2)return alert("Mínimo 2 nomes");

  let disponiveis=[...nomes];
  let participantes={};

  nomes.forEach(nome=>{
    let possiveis=disponiveis.filter(n=>n!==nome);
    let escolhido=possiveis[Math.floor(Math.random()*possiveis.length)];
    participantes[crypto.randomUUID()]={
      nome,
      senha:Math.random().toString(36).substring(2,8).toUpperCase(),
      resultado:escolhido,
      visto:false
    };
    disponiveis.splice(disponiveis.indexOf(escolhido),1);
  });

  sorteioAtual=crypto.randomUUID();
  salvarSorteio(sorteioAtual,participantes);
  renderizarLinks(sorteioAtual,participantes);
}

function salvarSorteio(id,participantes){
  localStorage.setItem("sorteio_"+id,JSON.stringify(participantes));
  const hist=JSON.parse(localStorage.getItem("historico"))||[];
  hist.push({id,data:new Date().toLocaleString(),participantes});
  localStorage.setItem("historico",JSON.stringify(hist));
}

function renderizarLinks(id,participantes){
  setup.style.display="none";
  links.innerHTML=`<button onclick="baixarExcel()">📥 Baixar Excel deste sorteio</button>`;
  for(let pid in participantes){
    const p=participantes[pid];
    const link=location.href.split("?")[0]+`?s=${id}&p=${pid}`;
    const msg=
`🎄 Amigo Oculto 🎄

Olá ${p.nome}! ✨

Chegou o momento de espalhar alegria, carinho e boas surpresas 🎁❤️  
Preparamos este amigo oculto com muito cuidado especialmente para você!

🔐 Sua senha: ${p.senha}

Clique no link abaixo para descobrir quem você tirou 🤫👇
${link}

🎅 Que esse Natal seja cheio de amor, risadas e bons presentes!`;

    const wpp = "https://wa.me/?text=" + encodeURIComponent(mensagem);
    
🤫 Guarde segredo!`;
    links.innerHTML+=`
    <div class="link">
      ${p.nome}<br>
      <a href="https://wa.me/?text=${encodeURIComponent(msg)}" target="_blank">
      📲 Enviar WhatsApp
      </a>
    </div>`;
  }
}

function baixarExcel(){
  if(prompt("Senha do administrador:")!==ADMIN)return alert("Senha incorreta");
  const dados=JSON.parse(localStorage.getItem("sorteio_"+sorteioAtual));
  const linhas=[["Nome","Senha","Amigo Oculto","Visualizado","Link"]];
  for(let pid in dados){
    const p=dados[pid];
    const link=location.href.split("?")[0]+`?s=${sorteioAtual}&p=${pid}`;
    linhas.push([p.nome,p.senha,p.resultado,p.visto?"Sim":"Não",link]);
  }
  const wb=XLSX.utils.book_new();
  const ws=XLSX.utils.aoa_to_sheet(linhas);
  XLSX.utils.book_append_sheet(wb,ws,"Sorteio");
  XLSX.writeFile(wb,"amigo-oculto.xlsx");
}

function exportarHistoricoExcel(){
  if(prompt("Senha do administrador:")!==ADMIN)return alert("Senha incorreta");
  const hist=JSON.parse(localStorage.getItem("historico"))||[];
  if(!hist.length)return alert("Nenhum histórico");
  const linhas=[["Data","Nome","Amigo Oculto","Senha","Visualizado","Link"]];
  hist.forEach(h=>{
    for(let pid in h.participantes){
      const p=h.participantes[pid];
      const link=location.href.split("?")[0]+`?s=${h.id}&p=${pid}`;
      linhas.push([h.data,p.nome,p.resultado,p.senha,p.visto?"Sim":"Não",link]);
    }
  });
  const wb=XLSX.utils.book_new();
  const ws=XLSX.utils.aoa_to_sheet(linhas);
  XLSX.utils.book_append_sheet(wb,ws,"Histórico");
  XLSX.writeFile(wb,"historico-amigo-oculto.xlsx");
}

function mostrarHistorico(){
  if(prompt("Senha do administrador:")!==ADMIN)return alert("Senha incorreta");
  const hist=JSON.parse(localStorage.getItem("historico"))||[];
  if(!hist.length)return alert("Nenhum sorteio");
  let texto="";
  hist.forEach((h,i)=>texto+=`${i+1} - ${h.data}\n`);
  const escolha=prompt(texto+"\nDigite o número do sorteio:");
  const idx=parseInt(escolha)-1;
  if(!hist[idx])return;
  sorteioAtual=hist[idx].id;
  renderizarLinks(sorteioAtual,hist[idx].participantes);
}

// PARTICIPANTE
const params=new URLSearchParams(location.search);
if(params.get("s")&&params.get("p")){
  const dados=JSON.parse(localStorage.getItem("sorteio_"+params.get("s")));
  const p=dados?.[params.get("p")];
  if(!p)card.innerHTML="<h2>Link inválido</h2>";
  else if(p.visto)card.innerHTML="<h2>⛔ Já visualizado</h2>";
  else{
    card.innerHTML=`
    <h2>🔒 Área Segura</h2>
    <p>${p.nome}</p>
    <input type="password" id="senha">
    <button onclick="ver()">Ver Resultado</button>
    <div id="res"></div>`;
    window.ver=()=>{
      if(senha.value!==p.senha)return alert("Senha incorreta");
      p.visto=true;
      localStorage.setItem("sorteio_"+params.get("s"),JSON.stringify(dados));
      res.innerHTML=`<h3>🎉 Você tirou:</h3><h2>${p.resultado}</h2>`;
    }
  }
}
</script>
</body>
</html>
