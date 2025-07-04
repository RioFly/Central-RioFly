<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Central RioFly - Aviation</title>
  
  <!-- Firebase SDK -->
  <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-database-compat.js"></script>
  
  <style>
    /* Reset e base */
    * {
      box-sizing: border-box;
    }
    body, html {
      margin: 0; padding: 0; height: 100%;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
      color: #333;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      overflow-x: hidden;
    }
    /* Containers */
    #app, #loginContainer, #registerContainer, #welcomeAnimation {
      width: 100%;
      max-width: 480px;
      background: white;
      border-radius: 12px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.1);
      padding: 2rem;
      box-sizing: border-box;
      display: none;
      flex-direction: column;
    }
    /* Mostrar o app na tela */
    #app.active, #loginContainer.active, #registerContainer.active {
      display: flex;
    }
    /* Welcome Animation */
    #welcomeAnimation {
      position: fixed;
      top: 0; left: 0; width: 100%; height: 100%;
      background: #0288d1;
      color: white;
      font-size: 2.2rem;
      font-weight: 700;
      justify-content: center;
      align-items: center;
      z-index: 9999;
      opacity: 0;
      transition: opacity 1.5s ease-in-out;
      display: flex;
      user-select: none;
    }
    #welcomeAnimation.show {
      opacity: 1;
    }

    /* Form Styles */
    form {
      display: flex;
      flex-direction: column;
      gap: 1rem;
    }
    input, select, textarea {
      padding: 0.75rem 1rem;
      border-radius: 8px;
      border: 1.5px solid #ccc;
      font-size: 1rem;
      transition: border-color 0.3s;
    }
    input:focus, select:focus, textarea:focus {
      outline: none;
      border-color: #0288d1;
      box-shadow: 0 0 5px rgba(2,136,209,0.4);
    }
    button {
      background: #0288d1;
      color: white;
      border: none;
      padding: 0.85rem 1.2rem;
      font-size: 1.1rem;
      border-radius: 8px;
      cursor: pointer;
      transition: background 0.3s ease;
      font-weight: 600;
    }
    button:hover {
      background: #026bb3;
    }
    /* Headings */
    h1, h2, h3 {
      margin: 0 0 1rem 0;
      color: #015f9f;
      text-align: center;
    }
    h1 {
      font-size: 2rem;
      font-weight: 800;
      margin-bottom: 2rem;
    }

    /* Error / Success Messages */
    .message {
      font-weight: 600;
      font-size: 0.9rem;
      margin-top: -0.5rem;
      margin-bottom: 1rem;
      text-align: center;
      user-select: none;
    }
    .error {
      color: #e53935;
    }
    .success {
      color: #43a047;
    }

    /* Tabs for Login/Register */
    .tabs {
      display: flex;
      justify-content: center;
      margin-bottom: 1.5rem;
      gap: 1rem;
    }
    .tab-btn {
      cursor: pointer;
      background: #e1f5fe;
      border: none;
      padding: 0.5rem 1.2rem;
      border-radius: 20px;
      font-weight: 600;
      color: #0288d1;
      transition: background-color 0.3s;
      user-select: none;
    }
    .tab-btn.active {
      background: #0288d1;
      color: white;
      box-shadow: 0 4px 10px rgba(2,136,209,0.4);
    }

    /* Main App Sections */
    #mainContent {
      display: flex;
      flex-direction: column;
      gap: 1rem;
    }
    /* Nav buttons */
    #navButtons {
      display: flex;
      justify-content: center;
      gap: 1rem;
      margin-bottom: 1rem;
    }
    #navButtons button {
      background: #0288d1;
      padding: 0.5rem 1rem;
      border-radius: 8px;
      font-weight: 600;
      font-size: 1rem;
      border: none;
      color: white;
      cursor: pointer;
      user-select: none;
      transition: background-color 0.3s ease;
    }
    #navButtons button:hover {
      background: #026bb3;
    }
    #navButtons button.active {
      background: #015f9f;
      box-shadow: 0 0 10px rgba(1,95,159,0.7);
    }

    /* Sections Diario e Financeiro */
    section {
      display: none;
      flex-direction: column;
      gap: 1rem;
      overflow-y: auto;
      max-height: 400px;
      padding-right: 8px; /* For scrollbar */
    }
    section.active {
      display: flex;
    }

    /* Histórico */
    .entrada {
      background: #e3f2fd;
      border-radius: 8px;
      padding: 1rem;
      box-shadow: 0 2px 6px rgba(0,0,0,0.1);
      font-size: 0.9rem;
      color: #015f9f;
    }
    .entrada strong {
      color: #013459;
    }

    /* Botão apagar */
    .btnDelete {
      background-color: #e53935 !important;
      color: white !important;
      border: none;
      padding: 0.4rem 0.8rem;
      border-radius: 6px;
      cursor: pointer;
      font-size: 0.85rem;
      margin-top: 0.5rem;
      user-select: none;
      transition: background-color 0.3s ease;
    }
    .btnDelete:hover {
      background-color: #ab2c25 !important;
    }

    /* Mensagem de sucesso */
    #mensagemSucesso {
      color: #43a047;
      font-weight: 700;
      font-size: 1rem;
      text-align: center;
      margin-bottom: 1rem;
      user-select: none;
    }

    /* Scrollbar customizada (Webkit) */
    section::-webkit-scrollbar {
      width: 8px;
    }
    section::-webkit-scrollbar-track {
      background: #f0f7fb;
      border-radius: 8px;
    }
    section::-webkit-scrollbar-thumb {
      background: #0288d1;
      border-radius: 8px;
    }

    /* Responsive */
    @media (max-width: 520px) {
      #app, #loginContainer, #registerContainer {
        max-width: 100%;
        border-radius: 0;
        height: 100vh;
        padding: 1.5rem 1rem;
      }
      section {
        max-height: none;
      }
      #welcomeAnimation {
        font-size: 1.6rem;
        padding: 0 1rem;
      }
    }
  </style>
</head>
<body>

  <!-- Welcome Animation -->
  <div id="welcomeAnimation">Bem-Vindo a RioFly Aviation</div>

  <!-- Login -->
  <div id="loginContainer" class="active" aria-label="Tela de login">
    <h1>Login - RioFly Aviation</h1>
    <form id="loginForm" aria-describedby="loginError">
      <input type="email" id="loginEmail" placeholder="Email" required autocomplete="username" />
      <input type="password" id="loginSenha" placeholder="Senha" required minlength="6" autocomplete="current-password" />
      <button type="submit">Entrar</button>
      <div id="loginError" class="message error" role="alert" aria-live="assertive"></div>
    </form>
    <p style="text-align:center; margin-top:1rem;">
      Não tem conta? <button class="tab-btn" id="btnShowRegister" aria-controls="registerContainer">Registrar</button>
    </p>
  </div>

  <!-- Registro -->
  <div id="registerContainer" aria-label="Tela de registro" style="display:none;">
    <h1>Registrar - RioFly Aviation</h1>
    <form id="registerForm" aria-describedby="registerError registerSuccess">
      <input type="email" id="registerEmail" placeholder="Email" required autocomplete="username" />
      <input type="password" id="registerSenha" placeholder="Senha (mín. 6 caracteres)" required minlength="6" autocomplete="new-password" />
      <button type="submit">Registrar</button>
      <div id="registerError" class="message error" role="alert" aria-live="assertive"></div>
      <div id="registerSuccess" class="message success" role="alert" aria-live="polite"></div>
    </form>
    <p style="text-align:center; margin-top:1rem;">
      Já tem conta? <button class="tab-btn" id="btnShowLogin" aria-controls="loginContainer">Fazer login</button>
    </p>
  </div>

  <!-- Main App -->
  <div id="app" aria-label="Aplicação Principal" role="main">
    <h1>Central RioFly Aviation</h1>

    <!-- Navegação -->
    <nav id="navButtons" role="navigation" aria-label="Navegação principal">
      <button id="btnDiario" class="active" aria-controls="diarioSection" aria-selected="true">Diário de Bordo</button>
      <button id="btnFinanceiro" aria-controls="financeiroSection" aria-selected="false">Financeiro</button>
      <button id="btnLogout" style="margin-left:auto; background:#e53935;">Sair</button>
    </nav>

    <div id="mainContent">
      <!-- Diário de Bordo -->
      <section id="diarioSection" class="active" aria-label="Seção Diário de Bordo">
        <div id="mensagemSucesso"></div>
        <form id="diarioForm">
          <label for="comandante">Comandante:</label>
          <input type="text" id="comandante" readonly aria-readonly="true" />
          
          <label for="dataVoo">Data do Voo:</label>
          <input type="date" id="dataVoo" required />
          
          <label for="aeronave">Aeronave:</label>
          <select id="aeronave" required>
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
          <input type="text" id="horasTotais" readonly aria-readonly="true" placeholder="Ex: 1200.00h" />

          <label for="trajeto">Trajeto:</label>
          <input type="text" id="trajeto" required placeholder="Ex: SBJR → SBSP / 200NM" />

          <label for="tempoVoo">Tempo de Voo:</label>
          <input type="text" id="tempoVoo" required placeholder="Ex: 2h30min" />

          <label for="tipoCombustivel">Tipo de Combustível:</label>
          <select id="tipoCombustivel" required>
            <option value="" disabled selected>Selecione</option>
            <option value="AVGAS">AVGAS</option>
            <option value="JET-A">JET-A</option>
          </select>

          <label for="litrosCombustivel">Litros de Combustível:</label>
          <input type="number" id="litrosCombustivel" min="0" step="0.1" required placeholder="Ex: 150.5" />

          <label for="custos">Custos:</label>
          <textarea id="custos" required placeholder="Ex: Taxas: R$150"></textarea>

          <label for="observacoes">Observações:</label>
          <textarea id="observacoes" placeholder="Ex: Voo tranquilo, sem intercorrências."></textarea>

          <button type="submit">Enviar Diário de Bordo</button>
        </form>

        <div id="historico" aria-live="polite" aria-atomic="true" style="margin-top: 1rem;"></div>

        <button id="btnManutencao" style="display:none; margin-top:1rem; background:#f57c00; color:#fff; border:none; padding:0.7rem; border-radius:8px; cursor:pointer;">
          Finalizar Manutenção e Liberar Aeronave
        </button>
        <div id="alertaManutencao" style="display:none; margin-top:1rem; padding: 0.75rem; background:#ffcc80; border-radius:8px; color:#5d4037; font-weight:700;">
          ⚠️ Aeronave em manutenção. Novos registros não são permitidos até liberação.
        </div>
      </section>

      <!-- Financeiro -->
      <section id="financeiroSection" aria-label="Seção Financeiro">
        <h2>Financeiro</h2>
        <div id="saldoAtual" style="font-weight: 700; margin-bottom: 1rem; font-size: 1.2rem;">Saldo Atual: R$ 0,00</div>

        <h3>Histórico de Voos</h3>
        <div id="financeiroHistorico" style="max-height: 180px; overflow-y: auto; margin-bottom: 1rem;"></div>

        <h3>Histórico de Manutenções</h3>
        <div id="manutencoesHistorico" style="max-height: 180px; overflow-y: auto;"></div>
      </section>
    </div>
  </div>

  <script>
    // Firebase Config
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
    const auth = firebase.auth();
    const db = firebase.database();

    // Elementos
    const loginContainer = document.getElementById("loginContainer");
    const registerContainer = document.getElementById("registerContainer");
    const appContainer = document.getElementById("app");
    const welcomeAnimation = document.getElementById("welcomeAnimation");

    const loginForm = document.getElementById("loginForm");
    const registerForm = document.getElementById("registerForm");
    const loginEmail = document.getElementById("loginEmail");
    const loginSenha = document.getElementById("loginSenha");
    const registerEmail = document.getElementById("registerEmail");
    const registerSenha = document.getElementById("registerSenha");

    const loginError = document.getElementById("loginError");
    const registerError = document.getElementById("registerError");
    const registerSuccess = document.getElementById("registerSuccess");

    const btnShowRegister = document.getElementById("btnShowRegister");
    const btnShowLogin = document.getElementById("btnShowLogin");

    const comandanteInput = document.getElementById("comandante");
    const horasTotaisInput = document.getElementById("horasTotais");
    const aeronaveSelect = document.getElementById("aeronave");
    const diarioForm = document.getElementById("diarioForm");
    const mensagemSucesso = document.getElementById("mensagemSucesso");
    const historicoDiv = document.getElementById("historico");

    const tipoCombustivelSelect = document.getElementById("tipoCombustivel");
    const litrosCombustivelInput = document.getElementById("litrosCombustivel");
    const custosInput = document.getElementById("custos");
    const observacoesInput = document.getElementById("observacoes");

    const btnDiario = document.getElementById("btnDiario");
    const btnFinanceiro = document.getElementById("btnFinanceiro");
    const btnLogout = document.getElementById("btnLogout");

    const diarioSection = document.getElementById("diarioSection");
    const financeiroSection = document.getElementById("financeiroSection");

    const saldoAtualDiv = document.getElementById("saldoAtual");
    const financeiroHistoricoDiv = document.getElementById("financeiroHistorico");
    const manutencoesHistoricoDiv = document.getElementById("manutencoesHistorico");

    const btnManutencao = document.getElementById("btnManutencao");
    const alertaManutencaoDiv = document.getElementById("alertaManutencao");

    let usuarioAtual = null;
    let manutencaoAtiva = false;
    let manutencaoAeronave = null;

    // Configurações preço por litro (valores reais aproximados em R$)
    const precoLitro = {
      "AVGAS": 7.50,  // preço médio litro AVGAS (pode ajustar)
      "JET-A": 5.30   // preço médio litro JET-A
    };

    // Email autorizado para acesso financeiro (somente você)
    const emailAutorizado = "henriqueetak@gmail.com";

    // ---- Funções ---- //

    // Alternar visibilidade entre login e registro
    btnShowRegister.onclick = () => {
      loginContainer.style.display = "none";
      registerContainer.style.display = "flex";
      clearMessages();
    };
    btnShowLogin.onclick = () => {
      registerContainer.style.display = "none";
      loginContainer.style.display = "flex";
      clearMessages();
    };

    function clearMessages() {
      loginError.innerText = "";
      registerError.innerText = "";
      registerSuccess.innerText = "";
      mensagemSucesso.innerText = "";
    }

    // Registro
    registerForm.onsubmit = async (e) => {
      e.preventDefault();
      clearMessages();
      const email = registerEmail.value.trim();
      const senha = registerSenha.value.trim();
      if (senha.length < 6) {
        registerError.innerText = "Senha deve ter pelo menos 6 caracteres.";
        return;
      }
      try {
        await auth.createUserWithEmailAndPassword(email, senha);
        registerSuccess.innerText = "Registro feito com sucesso! Agora faça login.";
        registerForm.reset();
        // Voltar para login
        setTimeout(() => {
          btnShowLogin.click();
        }, 1500);
      } catch (error) {
        registerError.innerText = "Erro: " + error.message;
      }
    };

    // Login
    loginForm.onsubmit = async (e) => {
      e.preventDefault();
      clearMessages();
      const email = loginEmail.value.trim();
      const senha = loginSenha.value.trim();
      try {
        await auth.signInWithEmailAndPassword(email, senha);
        loginForm.reset();
      } catch (error) {
        loginError.innerText = "Erro: " + error.message;
      }
    };

    // Logout
    btnLogout.onclick = async () => {
      await auth.signOut();
    };

    // Estado da autenticação
    auth.onAuthStateChanged(async (user) => {
      if (user) {
        usuarioAtual = user;

        // Exibir animação de boas-vindas
        welcomeAnimation.style.display = "flex";
        setTimeout(() => {
          welcomeAnimation.classList.add("show");
        }, 10);

        setTimeout(() => {
          welcomeAnimation.classList.remove("show");
          setTimeout(() => {
            welcomeAnimation.style.display = "none";
            // Mostrar app
            loginContainer.style.display = "none";
            registerContainer.style.display = "none";
            appContainer.classList.add("active");

            comandanteInput.value = usuarioAtual.email;
            limparFormularioDiario();
            carregarHorasTotaisAer();
            carregarHistorico();
            carregarFinanceiro();
            checkManutencaoStatus();
          }, 500);
        }, 3500);

      } else {
        usuarioAtual = null;
        appContainer.classList.remove("active");
        loginContainer.style.display = "flex";
        registerContainer.style.display = "none";
      }
    });

    // Limpar formulário diário
    function limparFormularioDiario() {
      diarioForm.reset();
      horasTotaisInput.value = "";
      mensagemSucesso.innerText = "";
    }

    // Carregar horas totais da aeronave selecionada
    async function carregarHorasTotaisAer() {
      const aeronave = aeronaveSelect.value;
      if (!aeronave) {
        horasTotaisInput.value = "";
        return;
      }
      const snapshot = await db.ref("horasTotais/" + aeronave).once("value");
      const horas = snapshot.val();
      horasTotaisInput.value = horas ? parseFloat(horas).toFixed(2) + "h" : "0.00h";
    }

    // Salvar horas totais atualizadas no DB
    async function salvarHorasTotaisAer(aeronave, horas) {
      await db.ref("horasTotais/" + aeronave).set(horas);
    }

    // Adiciona horas ao total
    function somarHorasTotais(horasAtual, horasParaSomar) {
      return (parseFloat(horasAtual) + parseFloat(horasParaSomar)).toFixed(2);
    }

    // Extrair horas decimais do formato "2h30min" -> 2.5
    function converterTempoParaHorasDecimal(tempoStr) {
      let h = 0, m = 0;
      const hMatch = tempoStr.match(/(\d+)h/);
      if (hMatch) h = parseInt(hMatch[1]);
      const mMatch = tempoStr.match(/(\d+)min/);
      if (mMatch) m = parseInt(mMatch[1]);
      return h + m / 60;
    }

    // Verificar se está em manutenção
    async function checkManutencaoStatus() {
      const aeronave = aeronaveSelect.value;
      if (!aeronave) {
        manutencaoAtiva = false;
        manutencaoAeronave = null;
        btnManutencao.style.display = "none";
        alertaManutencaoDiv.style.display = "none";
        return;
      }
      const manutSnap = await db.ref("manutencoes/" + aeronave).once("value");
      if (manutSnap.exists() && manutSnap.val().ativa) {
        manutencaoAtiva = true;
        manutencaoAeronave = aeronave;
        btnManutencao.style.display = "block";
        alertaManutencaoDiv.style.display = "block";
      } else {
        manutencaoAtiva = false;
        manutencaoAeronave = null;
        btnManutencao.style.display = "none";
        alertaManutencaoDiv.style.display = "none";
      }
    }

    // Evento para botão liberar manutenção
    btnManutencao.onclick = async () => {
      if (!manutencaoAeronave) return;
      await db.ref("manutencoes/" + manutencaoAeronave).remove();
      manutencaoAtiva = false;
      manutencaoAeronave = null;
      btnManutencao.style.display = "none";
      alertaManutencaoDiv.style.display = "none";
      alert("Manutenção finalizada. Aeronave liberada para novos voos.");
      carregarHorasTotaisAer();
      carregarHistorico();
    };

    // Ao mudar aeronave, atualizar horas e verificar manutenção + histórico
    aeronaveSelect.onchange = () => {
      carregarHorasTotaisAer();
      carregarHistorico();
      checkManutencaoStatus();
    };

    // Enviar Diário de Bordo
    diarioForm.onsubmit = async (e) => {
      e.preventDefault();
      mensagemSucesso.innerText = "";

      if (manutencaoAtiva) {
        alert("A aeronave está em manutenção. Não é possível registrar novos voos.");
        return;
      }

      // Valida campos obrigatórios
      const campos = [
        "comandante", "dataVoo", "aeronave", "trajeto",
        "tempoVoo", "tipoCombustivel", "litrosCombustivel", "custos"
      ];
      for (const id of campos) {
        const el = document.getElementById(id);
        if (!el.value || !el.value.trim()) {
          alert("Por favor, preencha todos os campos obrigatórios.");
          return;
        }
      }

      const aeronave = aeronaveSelect.value;
      const comandante = comandanteInput.value.trim();
      const dataVoo = document.getElementById("dataVoo").value;
      const trajeto = document.getElementById("trajeto").value.trim();
      const tempoVoo = document.getElementById("tempoVoo").value.trim();
      const tipoCombustivel = tipoCombustivelSelect.value;
      const litrosCombustivel = parseFloat(litrosCombustivelInput.value);
      const custos = custosInput.value.trim();
      const observacoes = observacoesInput.value.trim();

      // Calcula custo combustível real
      const precoCombustivel = precoLitro[tipoCombustivel] || 0;
      const custoCombustivel = litrosCombustivel * precoCombustivel;

      // Cria registro
      const novaEntradaRef = db.ref("diarios/" + aeronave).push();
      const dataHoraEnvio = new Date().toISOString();

      // Converte tempo voo para decimal para somar horas
      const tempoHorasDecimal = converterTempoParaHorasDecimal(tempoVoo);

      // Salvar no DB
      await novaEntradaRef.set({
        dataHoraEnvio,
        comandante,
        dataVoo,
        horasTotais: "",  // Atualizaremos depois
        trajeto,
        tempoVoo,
        tipoCombustivel,
        litrosCombustivel,
        custoCombustivel: custoCombustivel.toFixed(2),
        custos,
        observacoes
      });

      // Atualiza horas totais da aeronave
      const snapshotHoras = await db.ref("horasTotais/" + aeronave).once("value");
      const horasAtuais = parseFloat(snapshotHoras.val()) || 0;
      const novasHoras = horasAtuais + tempoHorasDecimal;

      await salvarHorasTotaisAer(aeronave, novasHoras.toFixed(2));

      // Atualiza horasTotais no diário (pode ser útil para histórico)
      await novaEntradaRef.update({ horasTotais: novasHoras.toFixed(2) });

      mensagemSucesso.innerText = "✅ Diário salvo com sucesso!";

      // Atualiza horas totais no input e histórico
      horasTotaisInput.value = novasHoras.toFixed(2) + "h";

      carregarHistorico();
      carregarFinanceiro();
      checkManutencaoStatus();

      // Checa se passou 70h para manutenção
      if (novasHoras >= 70 && !manutencaoAtiva) {
        manutencaoAtiva = true;
        manutencaoAeronave = aeronave;
        btnManutencao.style.display = "block";
        alertaManutencaoDiv.style.display = "block";

        await db.ref("manutencoes/" + aeronave).set({ ativa: true, dataInicio: new Date().toISOString() });
        alert("⚠️ A aeronave ultrapassou 70 horas e entrou em manutenção!");
      }

      diarioForm.reset();
      comandanteInput.value = comandante; // manter o comandante
      horasTotaisInput.value = novasHoras.toFixed(2) + "h";
    };

    // Carregar histórico resumido
    async function carregarHistorico() {
      const aeronave = aeronaveSelect.value;
      historicoDiv.innerHTML = "";
      if (!aeronave) return;
      const snapshot = await db.ref("diarios/" + aeronave).orderByChild("dataHoraEnvio").once("value");
      if (!snapshot.exists()) {
        historicoDiv.innerHTML = "<p>Nenhum diário enviado ainda para esta aeronave.</p>";
        return;
      }
      const entries = [];
      snapshot.forEach(child => {
        entries.push(child.val());
      });
      entries.sort((a,b) => new Date(b.dataHoraEnvio) - new Date(a.dataHoraEnvio));
      historicoDiv.innerHTML = `<h3>Histórico Resumido - ${aeronave}</h3>`;
      entries.forEach(e => {
        historicoDiv.innerHTML += `
          <div class="entrada">
            <strong>Data do Envio:</strong> ${new Date(e.dataHoraEnvio).toLocaleString()}<br>
            <strong>Comandante:</strong> ${e.comandante}<br>
            <strong>Data do Voo:</strong> ${e.dataVoo}<br>
            <strong>Trajeto:</strong> ${e.trajeto}<br>
            <strong>Tempo de Voo:</strong> ${e.tempoVoo}<br>
            <strong>Combustível:</strong> ${e.tipoCombustivel} (${e.litrosCombustivel} L) - R$ ${e.custoCombustivel}<br>
            <strong>Custos:</strong> ${e.custos}<br>
            <strong>Observações:</strong> ${e.observacoes}<br>
          </div>
        `;
      });
    }

    // Carregar financeiro (saldo + histórico)
    async function carregarFinanceiro() {
      if (!usuarioAtual) return;
      if (usuarioAtual.email !== emailAutorizado) {
        financeiroSection.innerHTML = "<p>Você não tem permissão para acessar esta área.</p>";
        return;
      }

      // Saldo acumulado (exemplo: soma dos custos dos voos)
      const diariosSnap = await db.ref("diarios").once("value");
      let saldo = 0;
      financeiroHistoricoDiv.innerHTML = "";
      manutencoesHistoricoDiv.innerHTML = "";

      diariosSnap.forEach(aeronaveSnap => {
        const aeronave = aeronaveSnap.key;
        aeronaveSnap.forEach(entradaSnap => {
          const e = entradaSnap.val();
          saldo -= parseFloat(e.custos.replace(/[^\d.,]/g, '').replace(',', '.') || 0);
          // Mostrar histórico de voos
          financeiroHistoricoDiv.innerHTML += `
            <div class="entrada">
              <strong>Data:</strong> ${new Date(e.dataHoraEnvio).toLocaleString()}<br>
              <strong>Comandante:</strong> ${e.comandante}<br>
              <strong>Aeronave:</strong> ${aeronave}<br>
              <strong>Custos:</strong> ${e.custos}
            </div>
          `;
        });
      });

      // Mostrar histórico de manutenções
      const manutSnap = await db.ref("manutencoes").once("value");
      if (manutSnap.exists()) {
        manutSnap.forEach((manutAero) => {
          const a = manutAero.key;
          const m = manutAero.val();
          manutencoesHistoricoDiv.innerHTML += `
            <div class="entrada">
              <strong>Aeronave:</strong> ${a}<br>
              <strong>Status:</strong> ${m.ativa ? "Em manutenção" : "Finalizada"}<br>
              <strong>Data início:</strong> ${new Date(m.dataInicio).toLocaleString()}
            </div>
          `;
        });
      }

      saldoAtualDiv.innerText = `Saldo Atual: R$ ${saldo.toFixed(2).replace('.', ',')}`;
    }

    // Navegação Diário / Financeiro
    btnDiario.onclick = () => {
      btnDiario.classList.add("active");
      btnFinanceiro.classList.remove("active");
      diarioSection.classList.add("active");
      financeiroSection.classList.remove("active");
    };
    btnFinanceiro.onclick = () => {
      if (usuarioAtual.email !== emailAutorizado) {
        alert("Você não tem permissão para acessar o financeiro.");
        return;
      }
      btnFinanceiro.classList.add("active");
      btnDiario.classList.remove("active");
      financeiroSection.classList.add("active");
      diarioSection.classList.remove("active");
    };
    <html lang="pt-BR">
<head>
  <!-- ... (restante do <head> permanece igual) ... -->
</head>
<body>
  <!-- ... (restante do <body> permanece igual) ... -->

  <script>
    // ... (todo o script Firebase permanece igual até a parte abaixo) ...

    auth.onAuthStateChanged(async (user) => {
      if (user) {
        usuarioAtual = user;

        welcomeAnimation.style.display = "flex";
        setTimeout(() => {
          welcomeAnimation.classList.add("show");
        }, 10);

        setTimeout(() => {
          welcomeAnimation.classList.remove("show");
          setTimeout(() => {
            welcomeAnimation.style.display = "none";
            loginContainer.style.display = "none";
            registerContainer.style.display = "none";
            appContainer.classList.add("active");

            comandanteInput.value = usuarioAtual.email;
            limparFormularioDiario();
            carregarHorasTotaisAer();
            carregarHistorico();
            carregarFinanceiro();
            checkManutencaoStatus();
            mostrarVoosDoPiloto(usuarioAtual.email); // <-- Adicionado aqui
          }, 500);
        }, 3500);

      } else {
        usuarioAtual = null;
        appContainer.classList.remove("active");
        loginContainer.style.display = "flex";
        registerContainer.style.display = "none";
      }
    });

    // ... (restante das funções já existentes, como limparFormularioDiario etc.) ...

    // Função para mostrar os voos do piloto logado
    async function mostrarVoosDoPiloto(email) {
      const diariosSnap = await db.ref("diarios").once("value");
      const entradas = [];

      diariosSnap.forEach((aeronaveSnap) => {
        const aeronave = aeronaveSnap.key;
        aeronaveSnap.forEach((entradaSnap) => {
          const e = entradaSnap.val();
          if (e.comandante === email) {
            entradas.push({ ...e, aeronave });
          }
        });
      });

      if (entradas.length > 0) {
        historicoDiv.innerHTML = `<h3>Seus Últimos Voos</h3>`;
        entradas.sort((a, b) => new Date(b.dataHoraEnvio) - new Date(a.dataHoraEnvio));
        entradas.forEach(e => {
          historicoDiv.innerHTML += `
            <div class="entrada">
              <strong>Aeronave:</strong> ${e.aeronave}<br>
              <strong>Data do Envio:</strong> ${new Date(e.dataHoraEnvio).toLocaleString()}<br>
              <strong>Data do Voo:</strong> ${e.dataVoo}<br>
              <strong>Trajeto:</strong> ${e.trajeto}<br>
              <strong>Tempo:</strong> ${e.tempoVoo}<br>
              <strong>Combustível:</strong> ${e.tipoCombustivel} (${e.litrosCombustivel}L)<br>
              <strong>Custos:</strong> ${e.custos}<br>
              <strong>Observações:</strong> ${e.observacoes}
            </div>
          `;
        });
      } else {
        historicoDiv.innerHTML = "<p>Você ainda não registrou nenhum voo.</p>";
      }
    }
  </script>
</body>
</html>
