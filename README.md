<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Central RioFly - Login & Diário de Bordo</title>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-database-compat.js"></script>

<style>
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #e0f7fa, #80deea);
    margin: 0; padding: 0; min-height: 100vh; display: flex; justify-content: center; align-items: center;
  }
  #app {
    background: white;
    border-radius: 12px;
    padding: 2rem;
    width: 380px;
    box-shadow: 0 6px 15px rgba(0,0,0,0.15);
  }
  h2 {
    margin-top: 0;
    text-align: center;
    color: #007bb8;
    margin-bottom: 1.5rem;
  }
  label {
    font-weight: 600;
    margin-top: 0.8rem;
    display: block;
    color: #007bb8;
  }
  input, select, textarea, button {
    width: 100%;
    padding: 0.6rem;
    margin-top: 0.3rem;
    border-radius: 6px;
    border: 1px solid #ccc;
    box-sizing: border-box;
    font-size: 1rem;
  }
  button {
    background-color: #007bb8;
    color: white;
    font-weight: bold;
    border: none;
    cursor: pointer;
    margin-top: 1.2rem;
    transition: background-color 0.3s ease;
  }
  button:hover:not(:disabled) {
    background-color: #005f8a;
  }
  button:disabled {
    background-color: #aacde6;
    cursor: not-allowed;
  }
  .link-btn {
    background: none;
    border: none;
    color: #007bb8;
    cursor: pointer;
    margin-top: 0.8rem;
    text-decoration: underline;
    font-size: 0.9rem;
  }
  .error {
    color: #e74c3c;
    font-weight: 600;
    margin-top: 0.5rem;
  }
  .success {
    color: #27ae60;
    font-weight: 600;
    margin-top: 0.5rem;
  }
  #menu {
    display: flex;
    justify-content: space-between;
    margin-bottom: 1rem;
  }
  #menu button {
    width: 48%;
    font-weight: 600;
  }
  #logoutBtn {
    background-color: #e74c3c;
  }
  #logoutBtn:hover {
    background-color: #b63126;
  }
  #mensagemSucesso {
    color: #27ae60;
    font-weight: 600;
    text-align: center;
    margin-bottom: 1rem;
  }
  #alertaManutencao {
    background: #ffc107;
    padding: 0.6rem;
    border-radius: 6px;
    color: #7a5900;
    margin-bottom: 1rem;
    text-align: center;
    font-weight: 700;
  }
  .entrada {
    background: #e3f2fd;
    padding: 1rem;
    margin-bottom: 1rem;
    border-radius: 6px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.05);
  }
  #financeiroHistorico, #manutencoesHistorico {
    max-height: 250px;
    overflow-y: auto;
    margin-bottom: 1rem;
  }
  #saldoAtual {
    font-weight: bold;
    font-size: 1.2rem;
    margin-bottom: 1rem;
    text-align: center;
    color: #007bb8;
  }
  nav {
    margin-bottom: 1rem;
    text-align: center;
  }
  nav button {
    margin: 0 0.3rem;
    padding: 0.5rem 1rem;
  }
  /* Scrollbar */
  #financeiroHistorico::-webkit-scrollbar,
  #manutencoesHistorico::-webkit-scrollbar {
    width: 8px;
  }
  #financeiroHistorico::-webkit-scrollbar-thumb,
  #manutencoesHistorico::-webkit-scrollbar-thumb {
    background-color: #007bb8;
    border-radius: 4px;
  }
</style>
</head>
<body>
<div id="app">

  <!-- Login/Register -->
  <div id="authContainer">
    <h2>Central RioFly - Acesso</h2>

    <div id="authForms">

      <!-- Login Form -->
      <form id="loginForm">
        <label for="loginEmail">Email:</label>
        <input type="email" id="loginEmail" required placeholder="exemplo@exemplo.com" />
        <label for="loginSenha">Senha:</label>
        <input type="password" id="loginSenha" required minlength="6" placeholder="mínimo 6 caracteres" />
        <button type="submit">Entrar</button>
        <button type="button" class="link-btn" id="showRegister">Não tem conta? Registrar</button>
        <div id="loginError" class="error"></div>
      </form>

      <!-- Register Form -->
      <form id="registerForm" style="display:none;">
        <label for="regNome">Nome Completo:</label>
        <input type="text" id="regNome" required placeholder="Seu nome" />
        <label for="regEmail">Email:</label>
        <input type="email" id="regEmail" required placeholder="exemplo@exemplo.com" />
        <label for="regSenha">Senha:</label>
        <input type="password" id="regSenha" required minlength="6" placeholder="mínimo 6 caracteres" />
        <button type="submit">Registrar</button>
        <button type="button" class="link-btn" id="showLogin">Já tem conta? Entrar</button>
        <div id="registerError" class="error"></div>
        <div id="registerSuccess" class="success"></div>
      </form>

    </div>
  </div>

  <!-- Main App -->
  <div id="mainApp" style="display:none;">
    <nav>
      <button id="btnDiario" style="background-color:#007bb8; color:white;">Diário de Bordo</button>
      <button id="btnFinanceiro" style="background-color:#007bb8; color:white; display:none;">Financeiro</button>
      <button id="logoutBtn">Sair</button>
    </nav>

    <div id="alertaManutencao" style="display:none;">⚠️ Aeronave em manutenção! Não é possível enviar novos diários até a liberação.</div>

    <section id="diarioSection">
      <h2>Diário de Bordo - Central RioFly</h2>
      <form id="diarioForm">
        <label for="comandanteInput">Comandante (Nome do Usuário):</label>
        <input type="text" id="comandanteInput" readonly />

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
        <input type="text" id="horasTotais" required placeholder="Ex: 1200h" />

        <label for="trajeto">Trajeto:</label>
        <input type="text" id="trajeto" required placeholder="Ex: SBJR → SBSP / 200NM" />

        <label for="tempoVoo">Tempo de Voo:</label>
        <input type="text" id="tempoVoo" required placeholder="Ex: 2h30min" />

        <label for="tipoCombustivel">Tipo de Combustível:</label>
        <select id="tipoCombustivel" required>
          <option value="" disabled selected>Selecione o combustível</option>
          <option value="avgas">AVGAS</option>
          <option value="jet-a">JET-A</option>
        </select>

        <label for="litrosCombustivel">Litros de Combustível:</label>
        <input type="number" id="litrosCombustivel" min="0" step="0.1" required placeholder="Ex: 100" />

        <label for="custos">Custos:</label>
        <textarea id="custos" required placeholder="Ex: Taxas: R$150"></textarea>

        <label for="observacoes">Observações:</label>
        <textarea id="observacoes" placeholder="Ex: Voo tranquilo, sem intercorrências."></textarea>

        <button type="submit" id="btnEnviarDiario">Enviar Diário de Bordo</button>
      </form>

      <div id="mensagemSucesso" style="display:none; margin-top:1rem; color: #27ae60; font-weight: 700; text-align: center;"></div>

      <div id="historico" style="margin-top:2rem; max-height: 280px; overflow-y: auto;"></div>

      <button id="btnManutencao" style="display:none; background:#ffc107; color:#7a5900; font-weight:bold; border:none; border-radius:6px; padding:0.6rem; margin-top:1rem; width:100%;">Finalizar Manutenção e Liberar Aeronave</button>
    </section>

    <section id="financeiroSection" style="display:none;">
      <h2>Financeiro - Central RioFly</h2>
      <div id="saldoAtual">Saldo Atual: R$ 0,00</div>
      <h3>Histórico de Voos</h3>
      <div id="financeiroHistorico"></div>
      <h3>Histórico de Manutenções</h3>
      <div id="manutencoesHistorico"></div>
    </section>

  </div>
</div>

<script>
  // Config Firebase - troque aqui pelo seu config
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

  // Admin definido por email
  const adminEmail = "henriqueetak@gmail.com";

  // Preço litro combustíveis fixos reais (exemplo valores - atualize conforme real)
  const precoLitro = {
    "avgas": 8.50,
    "jet-a": 7.50
  };

  // Elementos DOM
  const authContainer = document.getElementById("authContainer");
  const mainApp = document.getElementById("mainApp");

  const loginForm = document.getElementById("loginForm");
  const registerForm = document.getElementById("registerForm");
  const showRegisterBtn = document.getElementById("showRegister");
  const showLoginBtn = document.getElementById("showLogin");
  const loginError = document.getElementById("loginError");
  const registerError = document.getElementById("registerError");
  const registerSuccess = document.getElementById("registerSuccess");

  const btnDiario = document.getElementById("btnDiario");
  const btnFinanceiro = document.getElementById("btnFinanceiro");
  const logoutBtn = document.getElementById("logoutBtn");

  const comandanteInput = document.getElementById("comandanteInput");
  const diarioForm = document.getElementById("diarioForm");
  const mensagemSucesso = document.getElementById("mensagemSucesso");
  const historicoDiv = document.getElementById("historico");
  const btnManutencao = document.getElementById("btnManutencao");
  const alertaManutencao = document.getElementById("alertaManutencao");

  const financeiroSection = document.getElementById("financeiroSection");
  const diarioSection = document.getElementById("diarioSection");
  const financeiroHistorico = document.getElementById("financeiroHistorico");
  const manutencoesHistorico = document.getElementById("manutencoesHistorico");
  const saldoAtual = document.getElementById("saldoAtual");

  let emManutencao = false;
  let manutencaoAtiva = null;
  let usuarioAtual = null;

  // Toggle forms
  showRegisterBtn.onclick = () => {
    loginForm.style.display = "none";
    registerForm.style.display = "block";
    loginError.innerText = "";
    registerError.innerText = "";
    registerSuccess.innerText = "";
  };
  showLoginBtn.onclick = () => {
    loginForm.style.display = "block";
    registerForm.style.display = "none";
    loginError.innerText = "";
    registerError.innerText = "";
    registerSuccess.innerText = "";
  };

  // Registro
  registerForm.onsubmit = async (e) => {
    e.preventDefault();
    registerError.innerText = "";
    registerSuccess.innerText = "";

    const nome = document.getElementById("regNome").value.trim();
    const email = document.getElementById("regEmail").value.trim();
    const senha = document.getElementById("regSenha").value;

    if (!nome) {
      registerError.innerText = "Por favor, insira seu nome.";
      return;
    }

    if (!validateEmail(email)) {
      registerError.innerText = "Email inválido.";
      return;
    }

    if (senha.length < 6) {
      registerError.innerText = "Senha deve ter no mínimo 6 caracteres.";
      return;
    }

    try {
      const userCredential = await auth.createUserWithEmailAndPassword(email, senha);
      await userCredential.user.updateProfile({ displayName: nome });
      registerSuccess.innerText = "Registrado com sucesso! Agora faça login.";
      registerForm.reset();
    } catch (error) {
      registerError.innerText = traduzirErro(error.code);
    }
  };

  // Login
  loginForm.onsubmit = async (e) => {
    e.preventDefault();
    loginError.innerText = "";
    const email = document.getElementById("loginEmail").value.trim();
    const senha = document.getElementById("loginSenha").value;

    try {
      await auth.signInWithEmailAndPassword(email, senha);
    } catch (error) {
      loginError.innerText = traduzirErro(error.code);
    }
  };

  // Logout
  logoutBtn.onclick = () => {
    auth.signOut();
  };

  // Traduz erros Firebase
  function traduzirErro(codigo) {
    switch(codigo) {
      case "auth/email-already-in-use": return "Email já está em uso.";
      case "auth/invalid-email": return "Email inválido.";
      case "auth/user-not-found": return "Usuário não encontrado.";
      case "auth/wrong-password": return "Senha incorreta.";
      case "auth/weak-password": return "Senha muito fraca.";
      default: return "Erro: " + codigo;
    }
  }

  // Validação email básica
  function validateEmail(email) {
    return /\S+@\S+\.\S+/.test(email);
  }

  // Quando o estado da autenticação muda (login/logout)
  auth.onAuthStateChanged(async (user) => {
    if (user) {
      usuarioAtual = user;
      authContainer.style.display = "none";
      mainApp.style.display = "block";

      comandanteInput.value = user.displayName || user.email;

      // Mostra botão financeiro só para admin
      if (user.email.toLowerCase() === adminEmail.toLowerCase()) {
        btnFinanceiro.style.display = "inline-block";
      } else {
        btnFinanceiro.style.display = "none";
      }

      // Inicial: mostra diário e esconde financeiro
      btnDiario.click();

      carregarHistorico();

    } else {
      usuarioAtual = null;
      authContainer.style.display = "block";
      mainApp.style.display = "none";
      // limpa formulário
      loginForm.reset();
      registerForm.reset();
      loginError.innerText = "";
      registerError.innerText = "";
      registerSuccess.innerText = "";
    }
  });

  // Navegação entre abas
  btnDiario.onclick = () => {
    diarioSection.style.display = "block";
    financeiroSection.style.display = "none";
    mensagemSucesso.style.display = "none";
  };
  btnFinanceiro.onclick = () => {
    diarioSection.style.display = "none";
    financeiroSection.style.display = "block";
    carregarFinanceiro();
  };

  // Função para enviar diário de bordo
  diarioForm.onsubmit = (e) => {
    e.preventDefault();
    if (emManutencao) {
      alert("A aeronave está em manutenção e não pode receber novos registros.");
      return;
    }
    const camposObrigatorios = ["dataVoo", "aeronave", "horasTotais", "trajeto", "tempoVoo", "tipoCombustivel", "litrosCombustivel", "custos"];
    for (const id of camposObrigatorios) {
      if (!document.getElementById(id).value.trim()) {
        alert("Por favor, preencha todos os campos obrigatórios.");
        return;
      }
    }

    if (!confirm("Confirma o envio deste Diário de Bordo?")) return;

    salvarNoFirebase();
  };

  // Função que salva diário no Firebase
  async function salvarNoFirebase() {
    const aeronave = document.getElementById("aeronave").value;
    const litros = parseFloat(document.getElementById("litrosCombustivel").value);
    const tipoCombustivel = document.getElementById("tipoCombustivel").value;
    const preco = precoLitro[tipoCombustivel];
    const custoCombustivel = litros * preco;

    const novaEntradaRef = db.ref("diarios/" + aeronave).push();

    const tempoVooStr = document.getElementById("tempoVoo").value.trim();
    const horasVoo = converterTempoVooParaHoras(tempoVooStr);

    // Verificar manutenção com base em horas totais
    let horasTotaisTexto = document.getElementById("horasTotais").value.trim();
    // extrair valor numérico das horas totais, ex: "1200h" -> 1200
    let horasTotais = parseFloat(horasTotaisTexto.replace(/[^\d.,]/g, "").replace(",", "."));
    if (isNaN(horasTotais)) horasTotais = 0;

    const novaHorasTotais = horasTotais + horasVoo;

    // Se já estiver em manutenção não deixa enviar e mostra alerta
    if (emManutencao) {
      alert("A aeronave está em manutenção. Libere para poder enviar novos diários.");
      return;
    }

    // Salvar diário
    const dataEnvio = new Date().toLocaleString("pt-BR");
    const comandante = usuarioAtual.displayName || usuarioAtual.email;

    const entry = {
      dataHoraEnvio: dataEnvio,
      comandante,
      dataVoo: document.getElementById("dataVoo").value,
      horasTotais: novaHorasTotais.toFixed(2) + "h",
      trajeto: document.getElementById("trajeto").value,
      tempoVoo: tempoVooStr,
      tipoCombustivel,
      litrosCombustivel: litros,
      custoCombustivel: custoCombustivel.toFixed(2),
      custos: document.getElementById("custos").value,
      observacoes: document.getElementById("observacoes").value
    };

    try {
      await novaEntradaRef.set(entry);

      // Salvar histórico financeiro (voo)
      const novoLancFinanceiroRef = db.ref("financeiro/voos").push();
      await novoLancFinanceiroRef.set({
        data: dataEnvio,
        comandante,
        aeronave,
        custoTotal: custoCombustivel.toFixed(2)
      });

      mensagemSucesso.innerText = "✅ Diário salvo com sucesso!";
      mensagemSucesso.style.display = "block";

      // Atualiza horas totais no input para próximo registro
      document.getElementById("horasTotais").value = novaHorasTotais.toFixed(2) + "h";

      // Verifica se deve ativar manutenção (a partir de 70h)
      if (novaHorasTotais >= 70 && !emManutencao) {
        emManutencao = true;
        manutencaoAtiva = aeronave;
        alertaManutencao.style.display = "block";
        setTimeout(() => alertaManutencao.style.display = "none", 3000);
        btnManutencao.style.display = "block";
      }

      carregarHistorico();
      diarioForm.reset();
      comandanteInput.value = comandante; // mantém nome do usuário

    } catch (error) {
      alert("Erro ao salvar diário: " + error.message);
    }
  }

  // Converter string "2h30min" para horas float (ex: 2.5)
  function converterTempoVooParaHoras(str) {
    let h = 0, m = 0;
    const hMatch = str.match(/(\d+)h/);
    const mMatch = str.match(/(\d+)min/);
    if (hMatch) h = parseInt(hMatch[1]);
    if (mMatch) m = parseInt(mMatch[1]);
    return h + m / 60;
  }

  // Carregar histórico diário de bordo (simplificado)
  function carregarHistorico() {
    const aeronave = document.getElementById("aeronave").value;
    if (!aeronave) {
      historicoDiv.innerHTML = "<p>Selecione uma aeronave para ver o histórico.</p>";
      return;
    }
    db.ref("diarios/" + aeronave).once("value", snapshot => {
      historicoDiv.innerHTML = `<h3>Histórico - ${aeronave}</h3>`;
      if (!snapshot.exists()) {
        historicoDiv.innerHTML += "<p>Nenhum diário enviado ainda para esta aeronave.</p>";
        return;
      }
      snapshot.forEach(childSnap => {
        const e = childSnap.val();
        historicoDiv.innerHTML += `
          <div class="entrada">
            <strong>Data:</strong> ${e.dataHoraEnvio}<br>
            <strong>Comandante:</strong> ${e.comandante}<br>
            <strong>Trajeto:</strong> ${e.trajeto}<br>
            <strong>Tempo:</strong> ${e.tempoVoo}<br>
            <strong>Custos:</strong> ${e.custos}<br>
          </div>
        `;
      });
    });
  }

  // Botão liberar manutenção
  btnManutencao.onclick = () => {
    if (!confirm("Deseja finalizar a manutenção e liberar a aeronave?")) return;
    emManutencao = false;
    manutencaoAtiva = null;
    btnManutencao.style.display = "none";
    alertaManutencao.style.display = "none";
    alert("Manutenção finalizada. Aeronave liberada para novos registros.");
  };

  // Carregar financeiro (apenas admin)
  async function carregarFinanceiro() {
    if (!usuarioAtual || usuarioAtual.email.toLowerCase() !== adminEmail.toLowerCase()) {
      financeiroSection.innerHTML = "<p>Acesso negado.</p>";
      return;
    }

    saldoAtual.innerText = "Saldo Atual: R$ 0,00";
    financeiroHistorico.innerHTML = "";
    manutencoesHistorico.innerHTML = "";

    // Calcular saldo e listar histórico voos
    const voosSnap = await db.ref("financeiro/voos").once("value");
    let saldo = 0;
    if (voosSnap.exists()) {
      voosSnap.forEach(child => {
        const voo = child.val();
        saldo -= parseFloat(voo.custoTotal);
        financeiroHistorico.innerHTML += `
          <div class="entrada">
            <strong>Data:</strong> ${voo.data}<br>
            <strong>Comandante:</strong> ${voo.comandante}<br>
            <strong>Aeronave:</strong> ${voo.aeronave}<br>
            <strong>Custo:</strong> R$ ${parseFloat(voo.custoTotal).toFixed(2)}
          </div>
        `;
      });
    }

    // Calcular saldo e listar histórico manutenções
    const manutSnap = await db.ref("financeiro/manutencoes").once("value");
    if (manutSnap.exists()) {
      manutSnap.forEach(child => {
        const m = child.val();
        saldo -= parseFloat(m.custo) || 0;
        manutencoesHistorico.innerHTML += `
          <div class="entrada">
            <strong>Data:</strong> ${m.data}<br>
            <strong>Aeronave:</strong> ${m.aeronave}<br>
            <strong>Custo:</strong> R$ ${parseFloat(m.custo).toFixed(2)}<br>
            <strong>Observações:</strong> ${m.observacoes || ""}
          </div>
        `;
      });
    }

    saldoAtual.innerText = `Saldo Atual: R$ ${saldo.toFixed(2)}`;
  }

  // Você pode adicionar aqui uma função para registrar uma manutenção paga, se quiser.

</script>
</body>
</html>
