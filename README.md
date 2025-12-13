<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sorteio Amigo Oculto</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin-top: 50px;
    }
    input[type="text"] {
      width: 300px;
      padding: 10px;
      margin: 5px;
      font-size: 16px;
    }
    button {
      padding: 10px 20px;
      font-size: 18px;
      background-color: #4CAF50;
      color: white;
      border: none;
      cursor: pointer;
      margin-top: 20px;
    }
    button:hover {
      background-color: #45a049;
    }
    #result {
      margin-top: 20px;
    }
  </style>
</head>
<body>

  <h1>Amigo Oculto - Sorteio</h1>
  <p>Insira os nomes dos participantes (separados por vírgula):</p>
  
  <input type="text" id="namesInput" placeholder="Ex: João, Maria, Carlos, Ana">
  
  <button onclick="sortear()">Sortear!</button>

  <div id="result"></div>

  <script>
    function sortear() {
      const namesInput = document.getElementById('namesInput').value;
      const names = namesInput.split(',').map(name => name.trim()).filter(name => name.length > 0);

      if (names.length < 2) {
        alert("Por favor, insira pelo menos dois nomes.");
        return;
      }

      const sorteados = [...names];
      const resultado = {};

      // Função para realizar o sorteio
      for (let i = 0; i < names.length; i++) {
        let sorteado;
        do {
          sorteado = sorteados[Math.floor(Math.random() * sorteados.length)];
        } while (sorteado === names[i] || resultado[sorteado]);

        resultado[names[i]] = sorteado;
        sorteados.splice(sorteados.indexOf(sorteado), 1); // Remove o nome sorteado para evitar repetição
      }

      // Exibir o resultado e gerar mensagem do WhatsApp
      let resultMessage = "<h3>Resultado do Sorteio:</h3>";
      let messageWhatsApp = "Amigo Oculto - Sorteio:\n";

      for (let [amigo, sorteado] of Object.entries(resultado)) {
        resultMessage += `<p><strong>${amigo}</strong> sorteou <strong>${sorteado}</strong></p>`;
        messageWhatsApp += `${amigo} sorteou ${sorteado}\n`;
      }

      document.getElementById('result').innerHTML = resultMessage;

      // Gerar link para WhatsApp
      const encodedMessage = encodeURIComponent(messageWhatsApp);
      const whatsappLink = `https://wa.me/?text=${encodedMessage}`;

      resultMessage += `<br><a href="${whatsappLink}" target="_blank"><button>Enviar no WhatsApp</button></a>`;
    }
  </script>

</body>
</html>
