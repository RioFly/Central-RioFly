<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Diário de Bordo - RioFly Aviation</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: linear-gradient(135deg, #e6f0fa, #cfddee);
            margin: 0;
            padding: 2rem;
        }
        main {
            max-width: 600px;
            margin: auto;
            background: white;
            padding: 2rem;
            border-radius: 8px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
        }
        input, button, textarea {
            width: 100%;
            padding: 0.5rem;
            margin-top: 0.5rem;
            box-sizing: border-box;
        }
        .entrada {
            background: #f2f6fc;
            padding: 0.8rem;
            margin-bottom: 1rem;
            border-radius: 5px;
        }
        .logo {
            position: absolute;
            top: 30px;
            left: 50%;
            transform: translateX(-50%);
            width: 65px;
            height: auto;
            opacity: 0.8;
        }
        .menu-btn {
            position: absolute;
            top: 25px;
            left: 90%;
            transform: translateX(-50%);
            font-size: 24px;
            background: none;
            border: none;
            cursor: pointer;
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
        }
        #backupContent {
            background: white;
            padding: 20px;
            max-height: 80%;
            overflow-y: auto;
            border-radius: 8px;
            max-width: 90%;
        }
        #backupContent h3 {
            margin-top: 0;
        }
        #backupContent .entrada {
            background: #f9f9f9;
            margin-bottom: 10px;
            padding: 10px;
            border-radius: 5px;
        }
    </style>
    <script>
        document.addEventListener("DOMContentLoaded", function() {
            carregarHistorico();
        });

        function confirmarEnvio() {
            if (confirm("Tem certeza que deseja enviar este Diário de Bordo?")) {
                salvarNoHistorico();
                return true;
            }
            return false;
        }

        function salvarNoHistorico() {
            let historico = JSON.parse(localStorage.getItem("historico_diario")) || [];

            const entrada = {
                dataHoraEnvio: new Date().toLocaleString(),
                comandante: document.getElementById("comandante").value,
                dataVoo: document.getElementById("dataVoo").value,
                aeronave: document.getElementById("aeronave").value,
                horasTotais: document.getElementById("horasTotais").value,
                trajeto: document.getElementById("trajeto").value,
                tempoVoo: document.getElementById("tempoVoo").value,
                custos: document.getElementById("custos").value,
                observacoes: document.getElementById("observacoes").value
            };

            historico.push(entrada);
            localStorage.setItem("historico_diario", JSON.stringify(historico));
            carregarHistorico();
        }

        function carregarHistorico() {
            let historico = JSON.parse(localStorage.getItem("historico_diario")) || [];
            let historicoDiv = document.getElementById("historico");
            historicoDiv.innerHTML = "<h3>Histórico de Diários Enviados</h3>";

            if (historico.length === 0) {
                historicoDiv.innerHTML += "<p>Nenhum envio realizado ainda.</p>";
                return;
            }

            historico.slice().reverse().forEach(function(entrada, indexReverso) {
                let indexOriginal = historico.length - 1 - indexReverso;
                historicoDiv.innerHTML += `
                    <div class='entrada' style='cursor:pointer;' onclick='mostrarDetalhes(${indexOriginal})'>
                        <strong>Data do Envio:</strong> ${entrada.dataHoraEnvio}
                        <button style="float:right; background:#e74c3c; color:white; border:none; padding:2px 6px; border-radius:4px; cursor:pointer;"
                            onclick="event.stopPropagation(); excluirEntrada(${indexOriginal})">Apagar</button>
                    </div>
                `;
            });
        }

        function mostrarDetalhes(index) {
            let historico = JSON.parse(localStorage.getItem("historico_diario")) || [];
            const e = historico[index];
            let backupDiv = document.getElementById("backupContent");

            backupDiv.innerHTML = `
                <h3>Detalhes do Diário</h3>
                <div class='entrada'>
                    <strong>Data do Envio:</strong> ${e.dataHoraEnvio}<br>
                    <strong>Comandante:</strong> ${e.comandante}<br>
                    <strong>Data do Voo:</strong> ${e.dataVoo}<br>
                    <strong>Aeronave:</strong> ${e.aeronave}<br>
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

        function excluirEntrada(index) {
            if (confirm("Tem certeza que deseja apagar este diário?")) {
                let historico = JSON.parse(localStorage.getItem("historico_diario")) || [];
                historico.splice(index, 1);
                localStorage.setItem("historico_diario", JSON.stringify(historico));
                carregarHistorico();
            }
        }

        function abrirBackup() {
            let historico = JSON.parse(localStorage.getItem("historico_diario")) || [];
            let backupDiv = document.getElementById("backupContent");
            backupDiv.innerHTML = "<h3>Backup dos Diários Enviados</h3>";

            if (historico.length === 0) {
                backupDiv.innerHTML += "<p>Nenhum diário salvo.</p>";
            } else {
                historico.slice().reverse().forEach(function(entrada) {
                    backupDiv.innerHTML += `
                        <div class='entrada'>
                            <strong>Data do Envio:</strong> ${entrada.dataHoraEnvio}<br>
                            <strong>Comandante:</strong> ${entrada.comandante}<br>
                            <strong>Data do Voo:</strong> ${entrada.dataVoo}<br>
                            <strong>Aeronave:</strong> ${entrada.aeronave}<br>
                            <strong>Horas Totais:</strong> ${entrada.horasTotais}<br>
                            <strong>Trajeto:</strong> ${entrada.trajeto}<br>
                            <strong>Tempo de Voo:</strong> ${entrada.tempoVoo}<br>
                            <strong>Custos:</strong> ${entrada.custos}<br>
                            <strong>Observações:</strong> ${entrada.observacoes}<br>
                        </div>
                    `;
                });
            }

            document.getElementById("backupModal").style.display = "flex";
        }

        function fecharBackup() {
            document.getElementById("backupModal").style.display = "none";
        }
    </script>
</head>
<body>
    <img src="https://media.discordapp.net/attachments/1268995649362333760/1385640335224340571/1750373219340.png" alt="Logo RioFly Aviation" class="logo">

    <button class="menu-btn" title="Menu" onclick="abrirBackup()">&#8801;</button>

    <main>
        <h2 style="margin-top: 40px;">Diário de Bordo - RioFly Aviation</h2>
        <form onsubmit="return confirmarEnvio()">
            <label for="comandante">Comandante (Ex: Nome/Cód. IVAO):</label>
            <input type="text" id="comandante" required placeholder="Ex: João Silva / BR1234">

            <label for="dataVoo">Data do Voo:</label>
            <input type="text" id="dataVoo" required placeholder="Ex: 20/06/2025">

            <label for="aeronave">Aeronave:</label>
            <input type="text" id="aeronave" required placeholder="Ex: Cessna 172 / PR-ABC">

            <label for="horasTotais">Horas Totais da Aeronave:</label>
            <input type="text" id="horasTotais" required placeholder="Ex: 1200h">

            <label for="trajeto">Trajeto:</label>
            <input type="text" id="trajeto" required placeholder="Ex: SBJR → SBSP">

            <label for="tempoVoo">Tempo de Voo:</label>
            <input type="text" id="tempoVoo" required placeholder="Ex: 2h30min">

            <label for="custos">Custos:</label>
            <textarea id="custos" required placeholder="Ex: Combustível: R$500, Taxas: R$150"></textarea>

            <label for="observacoes">Observações:</label>
            <textarea id="observacoes" placeholder="Ex: Voo tranquilo, sem intercorrências."></textarea>

            <button type="submit">Enviar Diário de Bordo</button>
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
