<template>
  <div style="background: #f5f5f5; min-height: 100vh; padding: 2rem 1rem;">
    <!-- CAIXA PRINCIPAL DO SITE -->
    <div style="background: white; text-align: center; font-family: sans-serif; padding: 2rem; width: 100%; max-width: 800px; margin: auto; border-radius: 15px; box-shadow: 0 5px 20px rgba(0,0,0,0.1); box-sizing: border-box; position: relative;">
      
      <!-- IDENTIFICAÇÃO DO USUÁRIO (Canto Superior Direito) -->
      <div style="position: absolute; top: 15px; right: 20px;">
        <select v-if="!nomeSalvo" v-model="nomeSelecionadoGlobally" @change="salvarNomeGlobal" style="padding: 5px 10px; border-radius: 5px; border: 1px solid #ccc; font-size: 0.9rem; background: #f0f0f0;">
          <option value="" disabled selected>👤 Entrar como...</option>
          <option v-for="p in participants" :key="p.name" :value="p.name">{{ p.name }}</option>
        </select>
        <div v-else style="display: flex; align-items: center; gap: 8px;">
          <span style="font-size: 0.9rem; font-weight: bold; color: #1565c0;">👤 {{ nomeSalvo }}</span>
          <button @click="trocarUsuario" style="background: transparent; border: none; font-size: 0.8rem; color: #f44336; cursor: pointer; text-decoration: underline; padding: 0;">Sair</button>
          <button @click="openAdmin" style="background: transparent; border: none; font-size: 1.2rem; cursor: pointer; padding: 0;" title="Acesso Admin">🔒</button>
        </div>
      </div>
      
      <!-- TÍTULO -->
      <h1 style="color: #c62828; font-size: 2.2rem; margin-bottom: 20px;">🎅 Natal 2026 🎄</h1>
      
      <!-- MENU DE NAVEGAÇÃO -->
      <div style="display: flex; justify-content: center; gap: 10px; margin-bottom: 25px; flex-wrap: wrap;">
        <button @click="activeTab = 'sorteio'" :style="activeTab === 'sorteio' ? activeStyle : inactiveStyle">🎁 Amigo Secreto</button>
        <button @click="activeTab = 'ceia'" :style="activeTab === 'ceia' ? activeStyle : inactiveStyle">🍽️ Cardápio</button>
        <button @click="activeTab = 'presentes'" :style="activeTab === 'presentes' ? activeStyle : inactiveStyle">🛍️ Vitrine de Presentes</button>
      </div>

      <!-- ========================================== -->
      <!-- ABA 1: SORTEIO DO AMIGO SECRETO -->
      <!-- ========================================== -->
      <div v-if="activeTab === 'sorteio'">

        <div style="background: #fff3e0; padding: 15px; border-radius: 8px; border: 2px dashed #ff9800; margin-bottom: 25px; text-align: left;">
          <h3 style="color: #e65100; margin: 0 0 10px 0; text-align: center;">📜 Regra do Jogo</h3>
          <ul style="margin: 0; padding-left: 20px; font-size: 1.1rem; color: #d84315; font-weight: bold; line-height: 1.5;">
            <li>🎁 Valor do presente: Mínimo de R$ 150,00</li>
          </ul>
        </div>

        <div v-if="isAdmin" style="background: #eceff1; padding: 20px; border: 2px solid #607d8b; border-radius: 8px; margin-bottom: 20px; text-align: left;">
          <h3 style="color: #263238; margin-top: 0;">⚙️ Painel do Administrador</h3>
          <p style="font-size: 14px; color: #546e7a;">Edite a lista abaixo e clique no botão para apagar e gerar novo sorteio.</p>
          <textarea v-model="adminNamesList" rows="8" style="width: 100%; padding: 10px; border-radius: 5px; border: 1px solid #ccc;"></textarea>
          <button @click="regerarSorteio" style="margin-top: 15px; padding: 12px; background: #E53935; color: white; border: none; border-radius: 5px; cursor: pointer; width: 100%; font-weight: bold;">🔄 Refazer Sorteio Oficial</button>
          <button @click="isAdmin = false" style="margin-top: 10px; padding: 10px; background: #90a4ae; color: white; border: none; border-radius: 5px; cursor: pointer; width: 100%;">Fechar Painel</button>
        </div>

        <div v-if="savedLocally && !revealedName && !isAdmin" style="background: #e3f2fd; padding: 20px; border-radius: 8px; border: 1px solid #90caf9; margin-top: 20px;">
          <p style="font-size: 1.2rem; color: #1565c0; font-weight: bold;">{{ nomeSalvo }}, você já tirou seu amigo secreto neste aparelho!</p>
          <button @click="showSavedResult" style="margin-top: 10px; padding: 15px; font-size: 16px; cursor: pointer; background: #1976D2; color: white; border: none; border-radius: 8px; width: 100%; font-weight: bold;">👀 Ver meu resultado novamente</button>
        </div>

        <div v-if="!revealedName && !savedLocally && !isAdmin">
          <div v-if="!nomeSalvo">
            <p style="font-size: 1.2rem; color: #555;">Selecione seu nome no canto superior direito para participar do sorteio.</p>
          </div>
          <div v-else>
            <p style="font-size: 1.2rem; color: #555;">Você está participando como <b>{{ nomeSalvo }}</b>.</p>
            
            <div v-if="participants.find(p => p.name === nomeSalvo)?.hasSeen" style="background: #ffebee; padding: 15px; border-radius: 8px; border: 1px solid #ef9a9a; margin-top: 15px;">
              <p style="color: #c62828; font-weight: bold; margin: 0;">⚠️ Você já realizou o sorteio em outro aparelho!</p>
              <p style="color: #e53935; font-size: 0.9rem; margin: 5px 0 0 0;">Se você não anotou quem tirou, peça para o administrador do sorteio verificar para você.</p>
            </div>
            
            <button v-else @click="revealSecretSanta" style="padding: 15px; font-size: 18px; cursor: pointer; background: #4CAF50; font-weight: bold; color: white; border: none; border-radius: 8px; width: 100%; margin-top: 15px;">Revelar meu Amigo Secreto</button>
          </div>
        </div>

        <div v-if="revealedName && !isAdmin">
          <h2>Você tirou:</h2>
          <h1 style="color: #E53935; font-size: 3.5rem; margin: 20px 0; background: #ffebee; padding: 20px; border-radius: 10px; border: 2px dashed #E53935;">{{ revealedName }}</h1>
          <p style="font-size: 1.3rem; font-weight: bold; color: #d32f2f;">📸 Tire um PRINT desta tela para não esquecer!</p>
          
          <button @click="irParaPresentes(revealedName)" style="margin-top: 20px; padding: 12px; font-size: 16px; background: #2196f3; color: white; border: none; border-radius: 5px; cursor: pointer; width: 100%; font-weight: bold;">🎁 Ver a lista de presentes de {{ revealedName }}</button>
          <button @click="resetView" style="margin-top: 10px; padding: 12px; font-size: 16px; background: #333; color: white; border: none; border-radius: 5px; cursor: pointer; width: 100%;">Sair e Ocultar</button>
        </div>

        <div v-if="!revealedName && !isAdmin" style="margin-top: 40px; border-top: 1px solid #ccc; padding-top: 20px;">
          <h3 style="color: #666; font-size: 1.1rem;">Ainda faltam tirar:</h3>
          <div style="display: flex; flex-wrap: wrap; justify-content: center; gap: 10px; margin-top: 15px;">
            <span v-for="p in participants.filter(p => !p.hasSeen)" :key="p.name" style="background: #e0e0e0; padding: 8px 16px; border-radius: 20px; font-size: 14px; color: #424242; font-weight: bold;">{{ p.name }}</span>
          </div>
        </div>
      </div>

      <!-- ABA 2: CEIA -->
      <div v-if="activeTab === 'ceia'">
        <Ceia :participants="participants" :usuario-atual="nomeSalvo" />
      </div>

      <!-- ABA 3: PRESENTES -->
      <div v-if="activeTab === 'presentes'">
        <Presentes :participants="participants" :usuario-atual="nomeSalvo" :filtro-inicial="filtroPresentesInicial" />
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import Swal from 'sweetalert2';
import Ceia from './components/Ceia.vue';
import Presentes from './components/Presentes.vue';

const activeTab = ref('sorteio');

const activeStyle = 'background: #c62828; color: white; border: none; padding: 10px 15px; border-radius: 20px; font-weight: bold; cursor: pointer; transition: 0.3s;';
const inactiveStyle = 'background: #e0e0e0; color: #333; border: none; padding: 10px 15px; border-radius: 20px; font-weight: bold; cursor: pointer; transition: 0.3s;';

const participants = ref([]);
const selectedName = ref('');
const revealedName = ref('');
const savedLocally = ref(false);
const isAdmin = ref(false);
const adminNamesList = ref('');
const nomeSalvo = ref('');
const nomeSelecionadoGlobally = ref('');
const filtroPresentesInicial = ref('');

const irParaPresentes = (nome) => {
  filtroPresentesInicial.value = nome;
  activeTab.value = 'presentes';
};

const salvarNomeGlobal = () => {
  if (nomeSelecionadoGlobally.value) {
    localStorage.setItem('meuNome', nomeSelecionadoGlobally.value);
    nomeSalvo.value = nomeSelecionadoGlobally.value;
  }
};

const trocarUsuario = () => {
  localStorage.removeItem('meuNome');
  nomeSalvo.value = '';
  nomeSelecionadoGlobally.value = '';
  selectedName.value = '';
};

const fetchParticipants = async () => {
  try {
    const res = await fetch('https://natal-bl3x.onrender.com/api/participants');
    participants.value = await res.json();
  } catch (error) {
    console.error("Erro ao buscar participantes:", error);
  }
};

onMounted(() => {
  fetchParticipants();
  if (localStorage.getItem('meuAmigoSecreto')) {
    savedLocally.value = true;
    nomeSalvo.value = localStorage.getItem('meuNome');
  }
});

const revealSecretSanta = async () => {
  const nameToUse = nomeSalvo.value || selectedName.value;
  const result = await Swal.fire({
    title: 'É você mesmo?',
    html: `Você confirma que é <b>${nameToUse}</b>?<br><br><small style="color: #d33;">Atenção: Só é possível sortear uma vez!</small>`,
    icon: 'question',
    showCancelButton: true,
    confirmButtonColor: '#4CAF50',
    cancelButtonColor: '#d33',
    confirmButtonText: 'Sim, sou eu!',
    cancelButtonText: 'Não, errei o nome'
  });

  if (!result.isConfirmed) return;

  try {
    const res = await fetch('https://natal-bl3x.onrender.com/api/reveal', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: nameToUse })
    });
    const data = await res.json();
    if (!res.ok) return Swal.fire('Acesso Bloqueado', data.error || 'Erro ao revelar.', 'error');
    
    revealedName.value = data.drawnName;
    localStorage.setItem('meuNome', nameToUse);
    nomeSalvo.value = nameToUse;
    localStorage.setItem('meuAmigoSecreto', data.drawnName);
    savedLocally.value = true;
    setTimeout(() => { Swal.fire({ title: '📸 Hora do Print!', text: 'Tire um PRINT da tela agora!', icon: 'warning', confirmButtonColor: '#3085d6' }); }, 500);
    await fetchParticipants();
  } catch (error) { Swal.fire('Erro!', 'Problema ao conectar com o servidor.', 'error'); }
};

const showSavedResult = () => {
  revealedName.value = localStorage.getItem('meuAmigoSecreto');
  setTimeout(() => { Swal.fire({ title: 'Lembrete!', text: 'Não esqueça de tirar o PRINT desta tela!', icon: 'info', confirmButtonColor: '#3085d6' }); }, 300);
};

const resetView = () => { revealedName.value = ''; };

const openAdmin = async () => {
  const { value: password } = await Swal.fire({ title: 'Área Restrita', input: 'password', inputPlaceholder: 'Digite a senha', showCancelButton: true });
  if (password === 'admin123') { isAdmin.value = true; adminNamesList.value = participants.value.map(p => p.name).join('\n'); }
  else if (password) { Swal.fire('Erro', 'Senha incorreta', 'error'); }
};

const regerarSorteio = async () => {
  const nomesArray = adminNamesList.value.split('\n').map(n => n.trim()).filter(n => n !== '');
  if (nomesArray.length < 3) return Swal.fire('Erro', 'Mínimo de 3 nomes.', 'warning');
  const confirm = await Swal.fire({ title: 'Atenção!', text: 'Isso apaga o sorteio de todos.', icon: 'warning', showCancelButton: true, confirmButtonColor: '#E53935', confirmButtonText: 'Refazer' });
  if (!confirm.isConfirmed) return;
  try {
    const res = await fetch('https://natal-bl3x.onrender.com/api/admin/shuffle', {
      method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ password: 'admin123', names: nomesArray })
    });
    const data = await res.json();
    if (data.success) {
      Swal.fire('Sucesso!', 'Novo sorteio gerado!', 'success');
      isAdmin.value = false; localStorage.clear(); savedLocally.value = false; revealedName.value = ''; selectedName.value = '';
      await fetchParticipants();
    } else { Swal.fire('Erro', data.error, 'error'); }
  } catch (e) { Swal.fire('Erro', 'Falha no servidor.', 'error'); }
};
</script>