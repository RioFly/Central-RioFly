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
                    document.getElementById("mensagemSucesso").innerText = "✅ Diário salvo com sucesso!";
                    document.getElementById("mensagemSucesso").style.display = "block";
                    setTimeout(() => {
                        document.getElementById("mensagemSucesso").style.display = "none";
                    }, 5000);
                    carregarHistoricoCompleto();
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
                            <strong>Data:</strong> ${entrada.dataHoraEnvio}<br>
                            <strong>Comandante:</strong> ${entrada.comandante}<br>
                            <strong>Trajeto:</strong> ${entrada.trajeto}<br>
                            <strong>Tempo:</strong> ${entrada.tempoVoo}<br>
                        </div>
                    `;
                });
            });
        }

        function carregarHistoricoCompleto() {
            const aeronave = document.getElementById("aeronave").value;
            if (!aeronave) return;
            db.ref("diarios/" + aeronave).once('value', function(snapshot) {
                let historicoDiv = document.getElementById("historico");
                historicoDiv.innerHTML = `<h3>Histórico Completo - ${aeronave}</h3>`;
                if (!snapshot.exists()) {
                    historicoDiv.innerHTML += "<p>Nenhum diário enviado ainda para esta aeronave.</p>";
                    return;
                }
                let totalVoos = 0;
                let totalHoras = 0;
                document.querySelector("form").style.display = "block";
                snapshot.forEach(function(childSnapshot) {
                    const entrada = childSnapshot.val();
                    const key = childSnapshot.key;
                    totalVoos++;
                    let tempo = entrada.tempoVoo.replace(/[^0-9hmin]/g, "");
                    let horas = 0;
                    let minutos = 0;
                    if (tempo.includes("h")) {
                        horas = parseInt(tempo.split("h")[0]) || 0;
                        if (tempo.includes("min")) {
                            minutos = parseInt(tempo.split("h")[1].replace("min", "")) || 0;
                        }
                    } else if (tempo.includes("min")) {
                        minutos = parseInt(tempo.replace("min", "")) || 0;
                    }
                    totalHoras += horas + (minutos / 60);
                    historicoDiv.innerHTML += `
                        <div class='entrada'>
                            <strong>Data do Envio:</strong> ${entrada.dataHoraEnvio}<br>
                            <strong>Comandante:</strong> ${entrada.comandante}<br>
                            <strong>Data do Voo:</strong> ${entrada.dataVoo}<br>
                            <strong>Horas Totais:</strong> ${entrada.horasTotais}<br>
                            <strong>Trajeto:</strong> ${entrada.trajeto}<br>
                            <strong>Tempo de Voo:</strong> ${entrada.tempoVoo}<br>
                            <strong>Custos:</strong> ${entrada.custos}<br>
                            <strong>Observações:</strong> ${entrada.observacoes}<br>
                            <button onclick="apagarEntrada('${aeronave}', '${key}')" style="background-color:#e74c3c; color:white; margin-top:8px; border:none; padding:6px 10px; border-radius:5px;">Apagar</button>
                        </div>
                    `;
                });
                historicoDiv.innerHTML = `<h3>Histórico Completo - ${aeronave}</h3>
                    <p><strong>Total de voos:</strong> ${totalVoos}</p>
                    <p><strong>Total de horas acumuladas:</strong> ${totalHoras.toFixed(2)} horas</p>
                ` + historicoDiv.innerHTML;
            });
        }

        function apagarEntrada(aeronave, key) {
            if (confirm("Tem certeza que deseja apagar este diário?")) {
                db.ref("diarios/" + aeronave + "/" + key).remove(function(error) {
                    if (error) {
                        alert('Erro ao apagar.');
                    } else {
                        carregarHistoricoCompleto();
                    }
                });
            }
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
        <div id="mensagemSucesso" style="display:none; color: green; font-weight: bold; margin-bottom: 1rem;"></div>
        <button class="menu-btn" title="Ver Histórico Completo" onclick="carregarHistoricoCompleto()">📖 Histórico</button>
        <h2 style="margin-top: 40px;">Diário de Bordo - RioFly Aviation</h2>
        <form onsubmit="return confirmarEnvio()">
            <label for="comandante">Comandante (Ex: Nome/Cód. IVAO):</label>
            <input type="text" id="comandante" required placeholder="Ex: João Silva / BR1234">

            <label for="dataVoo">Data do Voo:</label>
            <input type="text" id="dataVoo" required placeholder="Ex: 20/06/2025">

            <label for="aeronave">Aeronave:</label>
            <select id="aeronave" required onchange="carregarHistorico()">
                <option value="" disabled selected>Selecione a aeronave</option>
                <option value="Robinson R66 PP-JMB">Robinson R66 PP-JMB</option>
                <option value="Robinson R66 PS-JRF">Robinson R66 PS-JRF</option>
                <option value="Bell 407 PS-RIO">Bell 407 PS-RIO</option>
                <option value="Bell 407 PS-RFL">Bell 407 PS-RFL</option>
                <option value="H125 PP-HZB">H125 PP-HZB</option>
                <option value="Phenom 100 PP-EMB">Phenom 100 PP-EMB</option>
                <option value="Phenom 300 PR-NGM">Phenom 300 PR-NGM</option>
                <option value="Citation CJ3 PS-SCC">Citation CJ3 PS-SCC</option>
                <option value="Pilatus PC-12 PR-CBJ">Pilatus PC-12 PR-CBJ</option>
                <option value="Epic PS-VTT">Epic PS-VTT</option>
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
