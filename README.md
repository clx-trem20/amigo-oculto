<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>🎁 Amigo Oculto</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body {
  font-family: Arial, sans-serif;
  background: linear-gradient(135deg,#ff4e50,#1cba6b);
  min-height: 100vh;
  margin: 0;
  padding: 20px;
  color: #fff;
}
.container {
  background: rgba(0,0,0,0.25);
  border-radius: 12px;
  padding: 20px;
  max-width: 700px;
  margin: auto;
}
h1 { text-align: center; }
input, button, select {
  width: 100%;
  padding: 10px;
  margin: 6px 0;
  border-radius: 6px;
  border: none;
}
button {
  background: #fff;
  color: #000;
  font-weight: bold;
  cursor: pointer;
}
button:hover { opacity: 0.9; }
.list {
  margin-top: 10px;
  background: rgba(255,255,255,0.15);
  padding: 10px;
  border-radius: 6px;
}
.item {
  border-bottom: 1px solid rgba(255,255,255,0.2);
  padding: 6px 0;
}
.hidden { display: none; }
</style>
</head>

<body>
<div class="container">
  <h1>🎄 Amigo Oculto</h1>

  <input id="nome" placeholder="Nome da pessoa">
  <input id="email" placeholder="Email">
  <select id="categoria">
    <option value="">Categoria</option>
    <option>Secretaria</option>
    <option>Meio Ambiente</option>
    <option>Cultura</option>
    <option>Educação</option>
    <option>Outro</option>
  </select>
  <button onclick="addPessoa()">➕ Adicionar pessoa</button>

  <div class="list" id="lista"></div>

  <button onclick="sortear()">🎲 Sortear</button>

  <button onclick="baixarExcel()">📥 Baixar Excel (Admin)</button>
</div>

<!-- Firebase -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import { getFirestore, collection, addDoc, getDocs, deleteDoc, doc } 
from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyCgV3hmEfZX8dzPMxNoFiC9YURNboJkWf4",
  authDomain: "amigo-oculto-af918.firebaseapp.com",
  projectId: "amigo-oculto-af918",
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
const col = collection(db, "pessoas");

let pessoas = [];

async function carregar() {
  pessoas = [];
  const snap = await getDocs(col);
  snap.forEach(d => pessoas.push({ id: d.id, ...d.data() }));
  render();
}
carregar();

window.addPessoa = async () => {
  const nome = nomeInput.value.trim();
  const email = emailInput.value.trim();
  const categoria = categoriaInput.value;

  if (!nome || !email || !categoria) {
    alert("Preencha tudo");
    return;
  }

  await addDoc(col, { nome, email, categoria });
  nomeInput.value = emailInput.value = "";
  categoriaInput.value = "";
  carregar();
};

function render() {
  lista.innerHTML = "";
  pessoas.forEach(p => {
    lista.innerHTML += `
      <div class="item">
        ${p.nome} — ${p.categoria}
      </div>
    `;
  });
}

function shuffle(array) {
  let arr = [...array];
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
}

window.sortear = () => {
  if (pessoas.length < 3) {
    alert("Mínimo 3 pessoas");
    return;
  }

  let sorteados;
  do {
    sorteados = shuffle(pessoas);
  } while (sorteados.some((p, i) => p.nome === pessoas[i].nome));

  pessoas = pessoas.map((p, i) => ({
    ...p,
    tirou: sorteados[i].nome,
    categoriaSorteada: sorteados[i].categoria
  }));

  alert("🎉 Sorteio realizado com sucesso!");
};

window.baixarExcel = () => {
  const senha = prompt("Senha do administrador:");
  if (senha !== "clx") {
    alert("Senha incorreta");
    return;
  }

  let csv = "Quem sorteou,Email,Categoria,Tirou,Categoria sorteada\n";
  pessoas.forEach(p => {
    csv += `${p.nome},${p.email},${p.categoria},${p.tirou || ""},${p.categoriaSorteada || ""}\n`;
  });

  const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "amigo-oculto.xlsx";
  link.click();
};
</script>

<script>
const nomeInput = document.getElementById("nome");
const emailInput = document.getElementById("email");
const categoriaInput = document.getElementById("categoria");
const lista = document.getElementById("lista");
</script>

</body>
</html>
