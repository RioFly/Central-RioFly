<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Diário de Bordo - RioFly Aviation</title>

    <!-- Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-database-compat.js"></script>
    <script>
        const firebaseConfig = {
            apiKey: "AIzaSyBp-zGlT0absTC1u1yj-Cat_AgKKw5LlBQ",
            authDomain: "riofly-b8fde.firebaseapp.com",
            databaseURL: "https://riofly-b8fde-default-rtdb.firebaseio.com",
            projectId: "riofly-b8fde",
            storageBucket: "riofly-b8fde.appspot.com",
            messagingSenderId: "223853711888",
            appId: "1:223853711888:web:96fe01f8c7baca28a33095"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        function confirmarEnvio() {
            const campos = ["comandante", "dataVoo", "aeronave", "horasTotais", "trajeto", "tempoVoo", "custos"];
            for (let id of campos) {
                if (!document.getElementById(id).value.trim()) {
                    alert("Por favor, preencha todos os campos obrigatórios.");
                    return false;
                }
            }
            if (confirm("Tem certeza que deseja enviar este Diário de Bordo?")) {
                salvarNoFirebase();
                return false;
            }
            return false;
        }

        function salvarNoFirebase() {
            const aeronave = document.getElementById("aeronave").value;
            const novaEntradaRef = db.ref("diarios/" + aeronave).push();
            novaEntradaRef.set({
                dataHoraEnvio: new Date().toLocaleString(),
                comandante: document.getElementById("comandante").value,
                dataVoo: document.getElementById("dataVoo").value,
                horasTotais: document.getElementById("horasTotais").value,
                trajeto: document.getElementById("trajeto").value,
                tempoVoo: document.getElementById("tempoVoo").value,
                custos: document.getElementById("custos").value,
                observacoes: document.getElementById("observacoes").value
            }, function(error) {
                if (error) {
                    alert('Erro ao salvar no Firebase.');
                } else {
                    carregarHistorico();
                }
            });
        }

        function carregarHistorico() {
            const aeronave = document.getElementById("aeronave").value;
            if (!aeronave) return;
            db.ref("diarios/" + aeronave).once('value', function(snapshot) {
                let historicoDiv = document.getElementById("historico");
                historicoDiv.innerHTML = `<h3>Histórico - ${aeronave}</h3>`;
                if (!snapshot.exists()) {
                    historicoDiv.innerHTML += "<p>Nenhum diário enviado ainda para esta aeronave.</p>";
                    return;
                }
                snapshot.forEach(function(childSnapshot) {
                    const entrada = childSnapshot.val();
                    historicoDiv.innerHTML += `
                        <div class='entrada'>
                            <strong>Data do Envio:</strong> ${entrada.dataHoraEnvio}<br>
                            <strong>Comandante:</strong> ${entrada.comandante}<br>
                            <strong>Data do Voo:</strong> ${entrada.dataVoo}<br>
                            <strong>Trajeto:</strong> ${entrada.trajeto}<br>
                            <strong>Tempo de Voo:</strong> ${entrada.tempoVoo}<br>
                        </div>
                    `;
                });
            });
        }
    </script>

    <style>
        body {
            font-family: Arial, sans-serif;
            background: linear-gradient(135deg, #e0f7fa, #80deea);
            margin: 0;
            padding: 1rem;
        }
        main {
            max-width: 700px;
            margin: auto;
            background: white;
            padding: 1.5rem;
            border-radius: 12px;
            box-shadow: 0 6px 15px rgba(0,0,0,0.15);
        }
        input, button, textarea, select {
            width: 100%;
            padding: 0.75rem;
            margin-top: 0.75rem;
            box-sizing: border-box;
            border-radius: 6px;
            border: 1px solid #ccc;
        }
        .entrada {
            background: #e3f2fd;
            padding: 1rem;
            margin-bottom: 1rem;
            border-radius: 6px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.05);
        }
        .menu-btn {
            position: fixed;
            top: 10px;
            right: 10px;
            font-size: 24px;
            background: #2196f3;
            color: white;
            border: none;
            padding: 8px 12px;
            border-radius: 5px;
            cursor: pointer;
            z-index: 1000;
        }
    </style>
</head>
<body>
    <main>
        <button class="menu-btn" title="Carregar Histórico" onclick="carregarHistorico()">&#8801;</button>
        <h2 style="margin-top: 40px;">Diário de Bordo - RioFly Aviation</h2>
        <form onsubmit="return confirmarEnvio()">
            <label for="comandante">Comandante (Ex: Nome/Cód. IVAO):</label>
            <input type="text" id="comandante" required placeholder="Ex: João Silva / BR1234">

            <label for="dataVoo">Data do Voo:</label>
            <input type="text" id="dataVoo" required placeholder="Ex: 20/06/2025">

            <label for="aeronave">Aeronave:</label>
            <select id="aeronave" required onchange="carregarHistorico()">
                <option value="" disabled selected>Selecione a aeronave</option>
                <option>Robinson R66 PP-JMB</option>
                <option>Robinson R66 PS-JRF</option>
                <option>Bell 407 PS-RIO</option>
                <option>AW109 PS-FPS</option>
                <option>H125 PP-HZB</option>
                <option>Phenom 100 PP-EMB</option>
                <option>Phenom 300 PR-NGM</option>
                <option>Citation CJ3 PS-SCC</option>
                <option>B200 King Air PR-FVP</option>
            </select>

            <label for="horasTotais">Horas Totais da Aeronave:</label>
            <input type="text" id="horasTotais" required placeholder="Ex: 1200h">

            <label for="trajeto">Trajeto:</label>
            <input type="text" id="trajeto" required placeholder="Ex: SBJR → SBSP / 200NM">

            <label for="tempoVoo">Tempo de Voo:</label>
            <input type="text" id="tempoVoo" required placeholder="Ex: 2h30min">

            <label for="custos">Custos:</label>
            <textarea id="custos" required placeholder="Ex: Combustível: R$500, Taxas: R$150"></textarea>

            <label for="observacoes">Observações:</label>
            <textarea id="observacoes" placeholder="Ex: Voo tranquilo, sem intercorrências."></textarea>

            <button type="submit" style="background-color: #4caf50; color: white; font-weight: bold;">Enviar Diário de Bordo</button>
        </form>

        <div id="historico"></div>
    </main>
</body>
</html>
