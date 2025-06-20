<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Diário de Bordo - RioFly Aviation</title>
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
            position: relative;
        }
        .entrada button.apagar {
            position: absolute;
            top: 8px;
            right: 8px;
            background: #e74c3c;
            color: white;
            border: none;
            padding: 4px 8px;
            border-radius: 4px;
            cursor: pointer;
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
        #backupModal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.5);
            justify-content: center;
            align-items: center;
            z-index: 1001;
        }
        #backupContent {
            background: white;
            padding: 20px;
            max-height: 80%;
            overflow-y: auto;
            border-radius: 8px;
            max-width: 90%;
            box-shadow: 0 4px 10px rgba(0,0,0,0.2);
        }
    </style>
    <script>
        document.addEventListener("DOMContentLoaded", function() {
            carregarHistorico();
        });

        document.addEventListener("keydown", function(event) {
            if (event.key === "Escape") {
                fecharBackup();
            }
        });

        function confirmarEnvio() {
            const campos = ["comandante", "dataVoo", "aeronave", "horasTotais", "trajeto", "tempoVoo", "custos"];
            for (let id of campos) {
                if (!document.getElementById(id).value.trim()) {
                    alert("Por favor, preencha todos os campos obrigatórios.");
                    return false;
                }
            }
            if (confirm("Tem certeza que deseja enviar este Diário de Bordo?")) {
                salvarNoHistorico();
                return true;
            }
            return false;
        }

        function salvarNoHistorico() {
            const aeronaveSelecionada = document.getElementById("aeronave").value;
            const chaveHistorico = "historico_diario_" + aeronaveSelecionada;
            let historico = JSON.parse(localStorage.getItem(chaveHistorico)) || [];

            const entrada = {
                dataHoraEnvio: new Date().toLocaleString(),
                comandante: document.getElementById("comandante").value,
                dataVoo: document.getElementById("dataVoo").value,
                aeronave: aeronaveSelecionada,
                horasTotais: document.getElementById("horasTotais").value,
                trajeto: document.getElementById("trajeto").value,
                tempoVoo: document.getElementById("tempoVoo").value,
                custos: document.getElementById("custos").value,
                observacoes: document.getElementById("observacoes").value
            };

            historico.push(entrada);
            localStorage.setItem(chaveHistorico, JSON.stringify(historico));
            carregarHistorico();
        }

        function carregarHistorico() {
            const aeronaveSelecionada = document.getElementById("aeronave").value;
            if (!aeronaveSelecionada) return;

            const chaveHistorico = "historico_diario_" + aeronaveSelecionada;
            let historico = JSON.parse(localStorage.getItem(chaveHistorico)) || [];
            let historicoDiv = document.getElementById("historico");
            historicoDiv.innerHTML = `<h3>Histórico - ${aeronaveSelecionada}</h3>`;

            if (historico.length === 0) {
                historicoDiv.innerHTML += "<p>Nenhum envio realizado ainda para esta aeronave.</p>";
                return;
            }

            historico.slice().reverse().forEach(function(entrada, indexReverso) {
                let indexOriginal = historico.length - 1 - indexReverso;
                historicoDiv.innerHTML += `
                    <div class='entrada' onclick='mostrarDetalhes("${aeronaveSelecionada}", ${indexOriginal})'>
                        <strong>Data do Envio:</strong> ${entrada.dataHoraEnvio}
                        <button class='apagar' onclick="event.stopPropagation(); excluirEntrada('${aeronaveSelecionada}', ${indexOriginal})">Apagar</button>
                    </div>
                `;
            });
        }

        function mostrarDetalhes(aeronave, index) {
            const chaveHistorico = "historico_diario_" + aeronave;
            let historico = JSON.parse(localStorage.getItem(chaveHistorico)) || [];
            const e = historico[index];
            let backupDiv = document.getElementById("backupContent");

            backupDiv.innerHTML = `
                <h3>Detalhes do Diário - ${aeronave}</h3>
                <div class='entrada'>
                    <strong>Data do Envio:</strong> ${e.dataHoraEnvio}<br>
                    <strong>Comandante:</strong> ${e.comandante}<br>
                    <strong>Data do Voo:</strong> ${e.dataVoo}<br>
                    <strong>Horas Totais:</strong> ${e.horasTotais}<br>
                    <strong>Trajeto:</strong> ${e.trajeto}<br>
                    <strong>Tempo de Voo:</strong> ${e.tempoVoo}<br>
                    <strong>Custos:</strong> ${e.custos}<br>
                    <strong>Observações:</strong> ${e.observacoes}<br>
                </div>
                <button onclick='fecharBackup()' style='margin-top:10px;'>Fechar</button>
            `;

            document.getElementById("backupModal").style.display = "flex";
        }

        function excluirEntrada(aeronave, index) {
            if (confirm("Tem certeza que deseja apagar este diário?")) {
                const chaveHistorico = "historico_diario_" + aeronave;
                let historico = JSON.parse(localStorage.getItem(chaveHistorico)) || [];
                historico.splice(index, 1);
                localStorage.setItem(chaveHistorico, JSON.stringify(historico));
                carregarHistorico();
            }
        }

        function abrirBackup() {
            let backupDiv = document.getElementById("backupContent");
            backupDiv.innerHTML = "<h3>Selecione uma aeronave para visualizar o histórico.</h3>";
            document.getElementById("backupModal").style.display = "flex";
        }

        function fecharBackup() {
            document.getElementById("backupModal").style.display = "none";
        }
    </script>
</head>
<body>

    <main>
        <button class="menu-btn" title="Menu" onclick="abrirBackup()">&#8801;</button>
        <h2 style="margin-top: 40px;">Diário de Bordo - RioFly Aviation</h2>
        <form onsubmit="return confirmarEnvio()">
            <label for="comandante">Comandante (Ex: Nome/Cód. IVAO):</label>
            <input type="text" id="comandante" required placeholder="Ex: João Silva / BR1234">

            <label for="dataVoo">Data do Voo:</label>
            <input type="text" id="dataVoo" required placeholder="Ex: 20/06/2025">

            <label for="aeronave">Aeronave:</label>
            <select id="aeronave" required onchange="carregarHistorico()">
                <option value="" disabled selected>Selecione a aeronave</option>
                <option>Robinson R66 PP-JMPB</option>
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

    <div id="backupModal" onclick="fecharBackup()">
        <div id="backupContent" onclick="event.stopPropagation()">
            <button onclick="fecharBackup()" style="margin-bottom:10px;">Fechar</button>
        </div>
    </div>

</body>
</html>
