<template>
  <div class="snow-container">
    <div class="snowflake" v-for="n in 50" :key="n" :style="getSnowflakeStyle()"></div>
  </div>

  <!-- TELA DE LOGIN INICIAL -->
  <div v-if="!nomeSalvo && !isAdmin" style="background: transparent; min-height: 100vh; padding: 2rem 1rem; display: flex; justify-content: center; align-items: center; position: relative; z-index: 1;">
    <div class="card-natalino" style="padding: 3rem 2rem; text-align: center; width: 100%; max-width: 400px; position: relative;">
      
      <button @click="openAdmin" style="position: absolute; top: 15px; right: 15px; background: transparent; border: none; font-size: 1.2rem; cursor: pointer;" title="Acesso Admin">🔒</button>

      <h1 style="color: #c62828; font-size: 2.2rem; margin-top: 0;">🎅 Natal 2026 🎄</h1>
      <p style="color: #555; margin-bottom: 2rem; font-size: 1.1rem;">Selecione seu nome e digite a senha</p>
      
      <select v-model="nomeSelecionadoGlobally" style="width: 100%; padding: 15px; font-size: 1.1rem; border-radius: 8px; border: 1px solid #ccc; margin-bottom: 1rem;">
        <option value="" disabled selected>👤 Quem é você?</option>
        <option v-for="p in participants" :key="p.name" :value="p.name">
          {{ p.name }}
        </option>
      </select>

      <input type="password" v-model="senhaInput" placeholder="Sua Senha (4 dígitos)" style="width: 100%; padding: 15px; font-size: 1.1rem; border-radius: 8px; border: 1px solid #ccc; margin-bottom: 1.5rem; box-sizing: border-box;" v-if="nomeSelecionadoGlobally" />
      
      <button @click="fazerLogin" class="btn-brilho" :disabled="!nomeSelecionadoGlobally || !senhaInput" style="width: 100%; padding: 15px; font-size: 1.1rem; cursor: pointer;" :style="(nomeSelecionadoGlobally && senhaInput) ? '' : 'opacity: 0.5; cursor: not-allowed;'">
        Entrar
      </button>
    </div>
  </div>

  <!-- TELA DO ADMINISTRADOR (Isolada) -->
  <div v-else-if="isAdmin" style="background: transparent; min-height: 100vh; padding: 2rem 1rem; display: flex; justify-content: center; align-items: flex-start; position: relative; z-index: 1;">
    <div class="card-natalino" style="padding: 2rem; width: 100%; max-width: 800px;">
        <h3 style="color: #263238; margin-top: 0; font-size: 1.8rem; text-align: center;">⚙️ Painel do Administrador</h3>
        
        <div style="display: flex; justify-content: center; gap: 10px; margin-bottom: 20px; flex-wrap: wrap;">
          <a href="?adminTab=sorteio" @click.prevent="adminTab = 'sorteio'" :style="adminTab === 'sorteio' ? activeStyle : inactiveStyle" style="text-decoration: none; display: inline-block; text-align: center;">Sorteio</a>
          <a href="?adminTab=senhas" @click.prevent="adminTab = 'senhas'" :style="adminTab === 'senhas' ? activeStyle : inactiveStyle" style="text-decoration: none; display: inline-block; text-align: center;">Senhas</a>
          <a href="?adminTab=presentes" @click.prevent="adminTab = 'presentes'" :style="adminTab === 'presentes' ? activeStyle : inactiveStyle" style="text-decoration: none; display: inline-block; text-align: center;">Presentes</a>
          <a href="?adminTab=ceia" @click.prevent="adminTab = 'ceia'" :style="adminTab === 'ceia' ? activeStyle : inactiveStyle" style="text-decoration: none; display: inline-block; text-align: center;">Cardápio</a>
        </div>

        <div v-if="adminTab === 'sorteio'">
          <h4 style="color: #fff; text-align: center; margin-top: 0;">Participantes do Sorteio</h4>
          
          <div style="display: flex; gap: 10px; margin-bottom: 20px;">
            <input v-model="novoParticipanteNome" placeholder="Nome do participante" style="flex: 1; padding: 10px; border-radius: 5px; border: 1px solid #ccc; font-size: 1rem;" @keyup.enter="adminAdicionarParticipante" />
            <button @click="adminAdicionarParticipante" style="padding: 10px 15px; background: #4CAF50; color: white; border: none; border-radius: 5px; cursor: pointer; font-weight: bold;">Adicionar</button>
          </div>

          <table class="admin-table">
            <thead>
              <tr>
                <th>Nome</th>
                <th style="width: 150px;">Ações</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="p in sortedAdminParticipantsSorteio" :key="p._id">
                <td>
                  <span v-if="p.isActive" style="color: #2e7d32; font-weight: bold;">
                    {{ p.name }}
                    <span v-if="p.hasSeen" title="Já viu quem tirou no sorteio"> 👀</span>
                  </span>
                  <span v-else style="color: #999; text-decoration: line-through;">{{ p.name }} (Inativo)</span>
                </td>
                <td style="text-align: center; white-space: nowrap;">
                  <button @click="adminToggleAtivo(p)" :style="{ background: p.isActive ? '#ff9800' : '#4CAF50' }" style="color: white; border: none; border-radius: 3px; padding: 6px; cursor: pointer; font-size: 1rem; margin-right: 5px;" :title="p.isActive ? 'Desativar Participante' : 'Ativar Participante'">{{ p.isActive ? '⏸️' : '▶️' }}</button>
                  <button @click="adminEditarParticipante(p)" style="background: #2196f3; color: white; border: none; border-radius: 3px; padding: 6px; cursor: pointer; font-size: 1rem; margin-right: 5px;" title="Editar Nome">✏️</button>
                  <button @click="adminDeletarParticipante(p)" style="background: #f44336; color: white; border: none; border-radius: 3px; padding: 6px; cursor: pointer; font-size: 1rem;" title="Apagar Permanentemente">🗑️</button>
                </td>
              </tr>
            </tbody>
          </table>

          <button @click="regerarSorteio" class="btn-brilho" style="margin-top: 15px; padding: 12px; width: 100%; font-size: 1.1rem;">🔄 Gerar Sorteio Oficial (Apenas Ativos)</button>
        </div>

        <div v-if="adminTab === 'senhas'">
          <h4 style="color: #fff;">Participantes e Senhas</h4>
          <table class="admin-table">
            <thead>
              <tr>
                <th>Nome</th>
                <th>Senha Atual</th>
                <th>Alterou a Senha?</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="p in sortedAdminParticipantsSenhas" :key="p._id">
                <td>
                  <span v-if="p.name === nomeSalvo" style="background: #ffebee; color: #c62828; padding: 2px 6px; border-radius: 4px; font-size: 0.8rem; font-weight: bold; margin-right: 5px;">Admin</span>
                  {{ p.name }}
                  <span v-if="p.isActive === false" style="color: #999; font-size: 0.8rem; margin-left: 5px;">(Inativo)</span>
                </td>
                <td style="font-family: monospace;">{{ p.password }}</td>
                <td>
                  {{ p.passwordChanged ? '✅ Sim' : '❌ Não' }}
                  <button @click="adminAlterarSenha(p.name)" style="margin-left: 10px; background: #2e7d32; color: white; border: none; border-radius: 3px; padding: 4px 8px; cursor: pointer; font-size: 0.8rem;">Alterar</button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div v-if="adminTab === 'presentes'">
          <h4 style="color: #fff;">Gerenciar Presentes</h4>
          <div style="margin-bottom: 10px; display: flex; gap: 10px;">
            <button @click="deletarPresentesSelecionados" style="padding: 8px 15px; background: #f44336; color: white; border: none; border-radius: 5px; cursor: pointer;">Apagar Selecionados</button>
            <button @click="deletarTodosPresentes" style="padding: 8px 15px; background: #b71c1c; color: white; border: none; border-radius: 5px; cursor: pointer;">Apagar TODOS</button>
          </div>
          <div v-for="p in adminPresentes" :key="p._id" style="padding: 10px; border-bottom: 1px solid #eee; display: flex; align-items: center; gap: 10px;">
            <input type="checkbox" v-model="p.selecionado" style="width: 20px; height: 20px;" />
            <div>
              <strong>{{ p.nomeFamiliar }}</strong>: {{ p.item }} <span v-if="p.valor">[Valor: {{ p.valor }}]</span> <span v-if="p.tamanhoEspecificacao">({{ p.tamanhoEspecificacao }})</span>
            </div>
          </div>
        </div>

        <div v-if="adminTab === 'ceia'">
          <h4 style="color: #fff;">Gerenciar Cardápio</h4>
          <div style="margin-bottom: 10px; display: flex; gap: 10px;">
            <button @click="deletarCeiaSelecionada" style="padding: 8px 15px; background: #f44336; color: white; border: none; border-radius: 5px; cursor: pointer;">Apagar Selecionados</button>
            <button @click="deletarTodaCeia" style="padding: 8px 15px; background: #b71c1c; color: white; border: none; border-radius: 5px; cursor: pointer;">Apagar TODOS</button>
          </div>
          <div v-for="c in adminCeia" :key="c._id" style="padding: 10px; border-bottom: 1px solid #eee; display: flex; align-items: center; gap: 10px;">
            <input type="checkbox" v-model="c.selecionado" style="width: 20px; height: 20px;" />
            <div>
              <strong>{{ c.categoria }}</strong> - {{ c.nomePrato }} (Resp: {{ c.responsavel || 'Ninguém' }})
            </div>
          </div>
        </div>

        <button @click="sairAdmin" class="btn-outline" style="margin-top: 20px; padding: 10px; width: 100%; font-size: 1rem;">Sair do Painel</button>
    </div>
  </div>

  <!-- SITE PRINCIPAL -->
  <div v-else style="background: transparent; min-height: 100vh; padding: 2rem 1rem; position: relative; z-index: 1;">
    <div class="card-natalino" style="text-align: center; font-family: sans-serif; padding: 2rem; width: 100%; max-width: 800px; margin: auto; box-sizing: border-box; position: relative;">
      
      <div style="position: absolute; top: 15px; right: 20px;">
        <div style="display: flex; align-items: center; gap: 15px; flex-wrap: wrap; justify-content: flex-end;">
          <span style="font-size: 0.9rem; font-weight: bold; color: #1565c0;">👤 {{ nomeSalvo }}</span>
          <button @click="abrirPerfil" style="background: transparent; border: none; font-size: 0.8rem; color: #2196F3; cursor: pointer; text-decoration: underline; padding: 0;">⚙️ Alterar Senha</button>
          <button @click="trocarUsuario" style="background: transparent; border: none; font-size: 0.8rem; color: #f44336; cursor: pointer; text-decoration: underline; padding: 0;">Sair</button>
          <button v-if="nomeSalvo === 'Guilherme'" @click="openAdmin" style="background: transparent; border: none; font-size: 1.2rem; cursor: pointer; padding: 0;" title="Acesso Admin">🔒</button>
        </div>
      </div>
      
      <h1 style="color: #c62828; font-size: 2.2rem; margin-bottom: 20px; margin-top: 30px;">🎅 Natal 2026 🎄</h1>
      
      <div style="display: flex; justify-content: center; gap: 10px; margin-bottom: 25px; flex-wrap: wrap;">
        <a href="?tab=sorteio" @click.prevent="activeTab = 'sorteio'" :style="activeTab === 'sorteio' ? activeStyle : inactiveStyle" style="text-decoration: none; display: inline-block; text-align: center;">🎁 Amigo Secreto</a>
        <a href="?tab=ceia" @click.prevent="activeTab = 'ceia'" :style="activeTab === 'ceia' ? activeStyle : inactiveStyle" style="text-decoration: none; display: inline-block; text-align: center;">🍽️ Cardápio</a>
        <a href="?tab=presentes" @click.prevent="activeTab = 'presentes'" :style="activeTab === 'presentes' ? activeStyle : inactiveStyle" style="text-decoration: none; display: inline-block; text-align: center;">🛍️ Vitrine de Presentes</a>
      </div>

      <div v-if="activeTab === 'sorteio'">

        <div v-if="amigoSorteadoCache && !revealedName" style="background-color: rgba(255, 255, 255, 0.15); backdrop-filter: blur(8px); padding: 20px; border-radius: 8px; border: 1px solid rgba(255, 255, 255, 0.3); margin-top: 20px; color: #ffffff; text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.5);">
          <p style="font-size: 1.2rem; font-weight: bold;">{{ nomeSalvo }}, você já tirou seu amigo secreto!</p>
          <button @click="mostrarAmigoSorteadoCache" style="margin-top: 10px; padding: 15px; font-size: 16px; cursor: pointer; background: #2e7d32; color: white; border: none; border-radius: 8px; width: 100%; font-weight: bold; box-shadow: 0 4px 10px rgba(0,0,0,0.3);">👀 Ver meu resultado novamente</button>
        </div>

        <div v-else-if="!revealedName" style="text-align: center; padding: 20px 0;">
          <button @click="revealSecretSanta" class="btn-reveal btn-brilho">
            🎁 Revelar meu Amigo Secreto 🎁
          </button>
        </div>

        <div v-if="revealedName">
          <h2>Você tirou:</h2>
          <h1 style="color: #ffffff; font-size: 3.5rem; font-weight: 900; margin: 20px 0; background-color: #8B0000; padding: 20px; border-radius: 10px; border: 2px dashed #ffffff; box-shadow: 0 4px 10px rgba(0,0,0,0.3);">{{ revealedName }}</h1>
          
          <a :href="`?tab=presentes&user=${revealedName}`" @click.prevent="irParaPresentes(revealedName)" style="margin-top: 20px; padding: 12px; font-size: 16px; background: #2e7d32; color: white; border: none; border-radius: 5px; cursor: pointer; width: 100%; font-weight: bold; text-decoration: none; display: block; text-align: center; box-sizing: border-box;">🎁 Ver a lista de presentes de {{ revealedName }}</a>
          <button @click="resetView" style="margin-top: 10px; padding: 12px; font-size: 16px; background: #333; color: white; border: none; border-radius: 5px; cursor: pointer; width: 100%;">Sair e Ocultar</button>
        </div>

        <div v-if="!revealedName" style="margin-top: 40px; border-top: 1px solid #ccc; padding-top: 20px;">
          <h3 style="color: #666; font-size: 1.1rem;">Ainda faltam tirar:</h3>
          <div style="display: flex; flex-wrap: wrap; justify-content: center; gap: 10px; margin-top: 15px;">
            <span v-for="p in participants.filter(p => !p.hasSeen)" :key="p.name" style="background: #e0e0e0; padding: 8px 16px; border-radius: 20px; font-size: 14px; color: #424242; font-weight: bold;">{{ p.name }}</span>
          </div>
        </div>
      </div>

      <div v-if="activeTab === 'ceia'">
        <Ceia :participants="participants" :usuario-atual="nomeSalvo" />
      </div>

      <div v-if="activeTab === 'presentes'">
        <Presentes :participants="participants" :usuario-atual="nomeSalvo" :filtro-inicial="filtroPresentesInicial" />
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue';
import Swal from 'sweetalert2';
import Ceia from './components/Ceia.vue';
import Presentes from './components/Presentes.vue';

const getSnowflakeStyle = () => ({
  left: Math.random() * 100 + 'vw',
  width: Math.random() * 8 + 2 + 'px',
  height: Math.random() * 8 + 2 + 'px',
  animationDuration: Math.random() * 3 + 2 + 's',
  animationDelay: Math.random() * 5 + 's',
  opacity: Math.random()
});

const BASE_URL = window.location.hostname === 'localhost' || window.location.hostname === '127.0.0.1' ? 'http://localhost:3000' : 'https://natal-bl3x.onrender.com';

const activeTab = ref(localStorage.getItem('activeTab') || 'sorteio');

watch(activeTab, (newTab) => {
  localStorage.setItem('activeTab', newTab);
  const url = new URL(window.location);
  url.searchParams.set('tab', newTab);
  window.history.pushState({}, '', url);
});

const activeStyle = 'background: #c62828; color: white; border: none; padding: 10px 15px; border-radius: 20px; font-weight: bold; cursor: pointer; transition: 0.3s;';
const inactiveStyle = 'background: #e0e0e0; color: #333; border: none; padding: 10px 15px; border-radius: 20px; font-weight: bold; cursor: pointer; transition: 0.3s;';

const participants = ref([]);
const revealedName = ref('');
const isAdmin = ref(false);
const adminTab = ref(localStorage.getItem('adminTab') || 'sorteio');

watch(adminTab, (newTab) => {
  localStorage.setItem('adminTab', newTab);
  const url = new URL(window.location);
  url.searchParams.set('adminTab', newTab);
  window.history.pushState({}, '', url);
});
const adminNamesList = ref('');
const adminParticipants = ref([]);
const adminPresentes = ref([]);
const adminCeia = ref([]);
const novoParticipanteNome = ref('');

const sortedAdminParticipantsSorteio = computed(() => {
  return [...adminParticipants.value].sort((a, b) => {
    if (a.isActive && !b.isActive) return -1;
    if (!a.isActive && b.isActive) return 1;
    return a.name.localeCompare(b.name);
  });
});

const sortedAdminParticipantsSenhas = computed(() => {
  return [...adminParticipants.value].sort((a, b) => {
    if (a.name === nomeSalvo.value && b.name !== nomeSalvo.value) return -1;
    if (b.name === nomeSalvo.value && a.name !== nomeSalvo.value) return 1;
    if (a.isActive && !b.isActive) return -1;
    if (!a.isActive && b.isActive) return 1;
    return a.name.localeCompare(b.name);
  });
});

const nomeSalvo = ref('');
const amigoSorteadoCache = ref('');
const senhaInput = ref('');
const nomeSelecionadoGlobally = ref('');
const filtroPresentesInicial = ref('');
const adminPass = ref('');

const irParaPresentes = (nome) => {
  filtroPresentesInicial.value = nome;
  activeTab.value = 'presentes';
};

const fetchParticipants = async () => {
  try {
    const res = await fetch(`${BASE_URL}/api/participants`);
    participants.value = await res.json();
    
    if (nomeSalvo.value) {
      const me = participants.value.find(p => p.name === nomeSalvo.value);
      if (me && !me.hasSeen && amigoSorteadoCache.value) {
        amigoSorteadoCache.value = '';
        revealedName.value = '';
        const storedUser = localStorage.getItem('loggedInUser');
        if (storedUser) {
          const userObj = JSON.parse(storedUser);
          userObj.drawnName = '';
          localStorage.setItem('loggedInUser', JSON.stringify(userObj));
        }
      }
    }
  } catch (error) {
    console.error("Erro ao buscar participantes:", error);
  }
};

onMounted(() => {
  const urlParams = new URLSearchParams(window.location.search);
  const tabParam = urlParams.get('tab');
  if (tabParam) activeTab.value = tabParam;
  
  const adminTabParam = urlParams.get('adminTab');
  if (adminTabParam) adminTab.value = adminTabParam;
  
  const userParam = urlParams.get('user');
  if (userParam) filtroPresentesInicial.value = userParam;

  fetchParticipants();
  const storedUser = localStorage.getItem('loggedInUser');
  if (storedUser) {
    const userObj = JSON.parse(storedUser);
    nomeSalvo.value = userObj.name;
    amigoSorteadoCache.value = userObj.drawnName || '';
  }
});

const fazerLogin = async () => {
  try {
    const res = await fetch(`${BASE_URL}/api/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: nomeSelecionadoGlobally.value, password: senhaInput.value })
    });
    const data = await res.json();
    if (!res.ok) {
      return Swal.fire('Erro', data.error || 'Senha incorreta', 'error');
    }
    
    // Sucesso
    nomeSalvo.value = data.participant.name;
    if (data.participant.hasSeen) {
      amigoSorteadoCache.value = data.participant.drawnName;
    }
    localStorage.setItem('loggedInUser', JSON.stringify({
      name: data.participant.name,
      drawnName: data.participant.hasSeen ? data.participant.drawnName : ''
    }));
    senhaInput.value = '';
    
  } catch(e) {
    Swal.fire('Erro', 'Falha ao conectar ao servidor', 'error');
  }
};

const trocarUsuario = () => {
  localStorage.removeItem('loggedInUser');
  nomeSalvo.value = '';
  nomeSelecionadoGlobally.value = '';
  senhaInput.value = '';
  revealedName.value = '';
  amigoSorteadoCache.value = '';
  activeTab.value = 'sorteio';
};

const abrirPerfil = async () => {
  const { value: formValues } = await Swal.fire({
    title: 'Alterar Senha',
    html:
      '<input id="swal-old-pass" class="swal2-input" placeholder="Senha Atual" type="password" style="box-sizing: border-box; width: calc(100% - 2em); max-width: 100%;">' +
      '<input id="swal-new-pass" class="swal2-input" placeholder="Nova Senha (min 4 caracteres)" type="password" style="box-sizing: border-box; width: calc(100% - 2em); max-width: 100%;">',
    focusConfirm: false,
    showCancelButton: true,
    confirmButtonText: 'Salvar',
    cancelButtonText: 'Cancelar',
    preConfirm: () => {
      return [
        document.getElementById('swal-old-pass').value,
        document.getElementById('swal-new-pass').value
      ]
    }
  });

  if (formValues) {
    const [oldPassword, newPassword] = formValues;
    if (!oldPassword || !newPassword) return Swal.fire('Erro', 'Preencha os campos', 'warning');

    try {
      const res = await fetch(`${BASE_URL}/api/change-password`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name: nomeSalvo.value, oldPassword, newPassword })
      });
      const data = await res.json();
      if (!res.ok) return Swal.fire('Erro', data.error, 'error');
      
      Swal.fire('Sucesso!', 'Sua senha foi alterada.', 'success');
    } catch(e) {
      Swal.fire('Erro', 'Erro ao alterar senha', 'error');
    }
  }
};

const revealSecretSanta = async () => {
  try {
    const res = await fetch(`${BASE_URL}/api/reveal`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: nomeSalvo.value })
    });
    const data = await res.json();
    if (!res.ok) return Swal.fire('Acesso Bloqueado', data.error || 'Erro ao revelar.', 'error');
    
    revealedName.value = data.drawnName;
    amigoSorteadoCache.value = data.drawnName;
    
    localStorage.setItem('loggedInUser', JSON.stringify({
      name: nomeSalvo.value,
      drawnName: data.drawnName
    }));
    
    await fetchParticipants();
  } catch (error) { Swal.fire('Erro!', 'Problema ao conectar com o servidor.', 'error'); }
};

const mostrarAmigoSorteadoCache = () => {
  revealedName.value = amigoSorteadoCache.value;
};

const resetView = () => { revealedName.value = ''; };

// ===============================
// PAINEL DE ADMINISTRAÇÃO
// ===============================
const openAdmin = async () => {
  const { value: password } = await Swal.fire({ title: 'Área Restrita', input: 'password', inputPlaceholder: 'Digite a senha', showCancelButton: true });
  if (password === 'admin123') { 
    isAdmin.value = true; 
    adminPass.value = password;
    adminNamesList.value = participants.value.map(p => p.name).join('\n');
    await carregarDadosAdmin();
  }
  else if (password) { Swal.fire('Erro', 'Senha incorreta', 'error'); }
};

const sairAdmin = () => {
  isAdmin.value = false;
  adminPass.value = '';
};

const carregarDadosAdmin = async () => {
  try {
    // Participantes/Senhas
    let res = await fetch(`${BASE_URL}/api/admin/participants`, {
      method: 'POST', headers: {'Content-Type': 'application/json'}, body: JSON.stringify({ password: adminPass.value })
    });
    adminParticipants.value = await res.json();

    // Presentes
    res = await fetch(`${BASE_URL}/api/presentes`);
    let pData = await res.json();
    adminPresentes.value = pData.map(x => ({...x, selecionado: false}));

    // Ceia
    res = await fetch(`${BASE_URL}/api/ceia`);
    let cData = await res.json();
    adminCeia.value = cData.map(x => ({...x, selecionado: false}));

  } catch(e) {
    console.error(e);
  }
};

const adminAlterarSenha = async (name) => {
  const { value: newPassword } = await Swal.fire({
    title: `Alterar senha de ${name}`,
    input: 'password',
    inputPlaceholder: 'Nova Senha',
    showCancelButton: true
  });
  
  if (newPassword) {
    try {
      const res = await fetch(`${BASE_URL}/api/admin/change-password`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ password: adminPass.value, name, newPassword })
      });
      if (res.ok) {
        Swal.fire('Sucesso', 'Senha alterada.', 'success');
        await carregarDadosAdmin();
      } else {
        const data = await res.json();
        Swal.fire('Erro', data.error, 'error');
      }
    } catch(e) {
      Swal.fire('Erro', 'Erro ao alterar.', 'error');
    }
  }
};

const regerarSorteio = async () => {
  const confirm = await Swal.fire({ title: 'Atenção!', text: 'Isso apaga o sorteio de todos e cria um novo apenas com os participantes ativos!', icon: 'warning', showCancelButton: true, confirmButtonColor: '#E53935', confirmButtonText: 'Refazer Sorteio' });
  if (!confirm.isConfirmed) return;
  try {
    const res = await fetch(`${BASE_URL}/api/admin/shuffle`, {
      method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ password: adminPass.value })
    });
    const data = await res.json();
    if (data.success) {
      Swal.fire('Sucesso!', 'Novo sorteio gerado!', 'success');
      await fetchParticipants();
      await carregarDadosAdmin();
    } else { Swal.fire('Erro', data.error, 'error'); }
  } catch (e) { Swal.fire('Erro', 'Falha no servidor.', 'error'); }
};

const adminAdicionarParticipante = async () => {
  const nome = novoParticipanteNome.value.trim();
  if (!nome) return;
  try {
    const res = await fetch(`${BASE_URL}/api/admin/participant`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ password: adminPass.value, name: nome })
    });
    const data = await res.json();
    if (data.success) {
      novoParticipanteNome.value = '';
      await carregarDadosAdmin();
    } else {
      Swal.fire('Erro', data.error, 'error');
    }
  } catch (e) {
    Swal.fire('Erro', 'Erro ao adicionar', 'error');
  }
};

const adminToggleAtivo = async (p) => {
  const novoStatus = !p.isActive;
  try {
    const res = await fetch(`${BASE_URL}/api/admin/participant/${p._id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ password: adminPass.value, name: p.name, isActive: novoStatus })
    });
    const data = await res.json();
    if (data.success) {
      p.isActive = novoStatus;
      await carregarDadosAdmin();
      await fetchParticipants();
    } else {
      Swal.fire('Erro', data.error, 'error');
    }
  } catch (e) {
    Swal.fire('Erro', 'Erro ao alterar status', 'error');
  }
};

const adminEditarParticipante = async (p) => {
  const { value: newName } = await Swal.fire({
    title: 'Editar Nome',
    input: 'text',
    inputValue: p.name,
    showCancelButton: true,
    confirmButtonText: 'Salvar',
    inputValidator: (value) => {
      if (!value.trim()) {
        return 'O nome não pode ser vazio'
      }
    }
  });

  if (newName && newName.trim() !== p.name) {
    try {
      const res = await fetch(`${BASE_URL}/api/admin/participant/${p._id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ password: adminPass.value, name: newName.trim(), isActive: p.isActive })
      });
      const data = await res.json();
      if (data.success) {
        Swal.fire('Sucesso', 'Nome atualizado.', 'success');
        await carregarDadosAdmin();
        await fetchParticipants(); // Update frontend state
      } else {
        Swal.fire('Erro', data.error, 'error');
      }
    } catch (e) {
      Swal.fire('Erro', 'Erro ao atualizar', 'error');
    }
  }
};

const adminDeletarParticipante = async (p) => {
  const { value: passConfirm } = await Swal.fire({
    title: 'Apagar Permanentemente',
    html: `<p style="color: #d33;">Você tem certeza que deseja apagar <b>${p.name}</b>?<br>Isso removerá a senha e não poderá ser desfeito.</p>`,
    input: 'password',
    inputPlaceholder: 'Confirme a senha de Admin',
    showCancelButton: true,
    confirmButtonColor: '#d33',
    confirmButtonText: 'Apagar para sempre'
  });

  if (passConfirm) {
    if (passConfirm !== adminPass.value) {
      return Swal.fire('Erro', 'Senha incorreta.', 'error');
    }
    try {
      const res = await fetch(`${BASE_URL}/api/admin/participant/${p._id}`, {
        method: 'DELETE',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ password: adminPass.value })
      });
      const data = await res.json();
      if (data.success) {
        Swal.fire('Apagado', 'Participante removido do site.', 'success');
        await carregarDadosAdmin();
        await fetchParticipants();
      } else {
        Swal.fire('Erro', data.error, 'error');
      }
    } catch (e) {
      Swal.fire('Erro', 'Erro ao apagar', 'error');
    }
  }
};

const deletarPresentesSelecionados = async () => {
  const selecionados = adminPresentes.value.filter(p => p.selecionado).map(p => p._id);
  if (selecionados.length === 0) return Swal.fire('Aviso', 'Nenhum presente selecionado.', 'info');
  if (!(await Swal.fire({ title: 'Confirmar', text: 'Apagar presentes selecionados?', showCancelButton: true })).isConfirmed) return;

  await fetch(`${BASE_URL}/api/admin/presentes/delete`, {
    method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ password: adminPass.value, ids: selecionados })
  });
  await carregarDadosAdmin();
};

const deletarTodosPresentes = async () => {
  if (!(await Swal.fire({ title: 'Atenção!', text: 'Apagar TODOS os presentes?', icon: 'warning', showCancelButton: true })).isConfirmed) return;
  await fetch(`${BASE_URL}/api/admin/presentes/delete`, {
    method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ password: adminPass.value, all: true })
  });
  await carregarDadosAdmin();
};

const deletarCeiaSelecionada = async () => {
  const selecionados = adminCeia.value.filter(c => c.selecionado).map(c => c._id);
  if (selecionados.length === 0) return Swal.fire('Aviso', 'Nenhum item selecionado.', 'info');
  if (!(await Swal.fire({ title: 'Confirmar', text: 'Apagar itens do cardápio selecionados?', showCancelButton: true })).isConfirmed) return;

  await fetch(`${BASE_URL}/api/admin/ceia/delete`, {
    method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ password: adminPass.value, ids: selecionados })
  });
  await carregarDadosAdmin();
};

const deletarTodaCeia = async () => {
  if (!(await Swal.fire({ title: 'Atenção!', text: 'Apagar TODO o cardápio?', icon: 'warning', showCancelButton: true })).isConfirmed) return;
  await fetch(`${BASE_URL}/api/admin/ceia/delete`, {
    method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ password: adminPass.value, all: true })
  });
  await carregarDadosAdmin();
};
</script>

<style>
.btn-reveal {
  padding: 18px 30px;
  font-size: 22px;
  cursor: pointer;
  background: linear-gradient(135deg, #4CAF50, #2E7D32);
  color: white;
  border: none;
  border-radius: 50px;
  width: 100%;
  max-width: 400px;
  font-weight: bold;
  box-shadow: 0 4px 15px rgba(76, 175, 80, 0.4);
  transition: transform 0.2s, box-shadow 0.2s;
  animation: pulse 2s infinite;
}
.btn-reveal:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 20px rgba(76, 175, 80, 0.6);
}
@keyframes pulse {
  0% { box-shadow: 0 0 0 0 rgba(76, 175, 80, 0.7); }
  70% { box-shadow: 0 0 0 15px rgba(76, 175, 80, 0); }
  100% { box-shadow: 0 0 0 0 rgba(76, 175, 80, 0); }
}
</style>