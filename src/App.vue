<template>
  <div class="container" style="text-align: center; font-family: sans-serif; padding: 2rem; max-width: 600px; margin: auto;">
    
    <!-- TÍTULO NOVO -->
    <h1 style="color: #c62828; font-size: 2.2rem; margin-bottom: 10px;">🎅 Amigo Secreto Natal 2026 🎄</h1>
    
    <!-- MURAL DE REGRAS -->
    <div style="background: #fff3e0; padding: 15px; border-radius: 8px; border: 2px dashed #ff9800; margin-bottom: 25px; box-shadow: 0 4px 6px rgba(0,0,0,0.05);">
      <h3 style="color: #e65100; margin: 0 0 10px 0;">📜 Regra do Sorteio</h3>
      <p style="margin: 0; font-size: 1.2rem; color: #d84315; font-weight: bold;">
        🎁 Valor do presente: Mínimo de R$ 150,00
      </p>
    </div>

    <!-- PAINEL DO ADMINISTRADOR (Secreto) -->
    <div v-if="isAdmin" style="background: #eceff1; padding: 20px; border: 2px solid #607d8b; border-radius: 8px; margin-bottom: 20px; text-align: left;">
      <h3 style="color: #263238; margin-top: 0;">⚙️ Painel do Administrador</h3>
      <p style="font-size: 14px; color: #546e7a;">Edite a lista abaixo (um nome por linha) e clique no botão para apagar o sorteio atual e gerar um novo.</p>
      
      <textarea v-model="adminNamesList" rows="8" style="width: 100%; padding: 10px; font-family: sans-serif; border-radius: 5px; border: 1px solid #ccc;"></textarea>
      
      <button @click="regerarSorteio" style="margin-top: 15px; padding: 12px; background: #E53935; color: white; border: none; border-radius: 5px; cursor: pointer; width: 100%; font-weight: bold; font-size: 16px;">
        🔄 Refazer Sorteio Oficial
      </button>
      <button @click="isAdmin = false" style="margin-top: 10px; padding: 10px; background: #90a4ae; color: white; border: none; border-radius: 5px; cursor: pointer; width: 100%;">
        Fechar Painel
      </button>
    </div>

    <!-- Tela 1: Já tem um sorteio salvo -->
    <div v-if="savedLocally && !revealedName && !isAdmin" style="background: #e3f2fd; padding: 20px; border-radius: 8px; border: 1px solid #90caf9; margin-top: 20px;">
      <p style="font-size: 1.2rem; color: #1565c0; font-weight: bold;">Você já tirou seu amigo secreto neste aparelho!</p>
      <button 
        @click="showSavedResult" 
        style="margin-top: 10px; padding: 15px 20px; font-size: 16px; cursor: pointer; background: #1976D2; color: white; border: none; border-radius: 8px; width: 100%; font-weight: bold;"
      >
        👀 Ver meu resultado novamente
      </button>
    </div>

    <!-- Tela 2: Novo Sorteio -->
    <div v-if="!revealedName && !savedLocally && !isAdmin">
      <p style="font-size: 1.2rem; color: #555;">Selecione o seu nome na lista abaixo:</p>
      
      <select v-model="selectedName" style="padding: 12px; font-size: 16px; margin-bottom: 20px; width: 100%; border-radius: 8px; border: 1px solid #ccc;">
        <option disabled value="">Escolha seu nome...</option>
        <option v-for="p in participants" :key="p.name" :value="p.name" :disabled="p.hasSeen">
          {{ p.name }} {{ p.hasSeen ? '(Já viu o resultado)' : '' }}
        </option>
      </select>
      
      <br>
      <button 
        @click="revealSecretSanta" 
        :disabled="!selectedName"
        style="padding: 15px 20px; font-size: 18px; cursor: pointer; background: #4CAF50; font-weight: bold; color: white; border: none; border-radius: 8px; width: 100%; transition: background 0.3s;"
      >
        Revelar meu Amigo Secreto
      </button>
    </div>

    <!-- Tela 3: O Resultado -->
    <div v-if="revealedName && !isAdmin">
      <h2>Você tirou:</h2>
      <h1 style="color: #E53935; font-size: 3.5rem; margin: 20px 0; background: #ffebee; padding: 20px; border-radius: 10px; border: 2px dashed #E53935;">
        {{ revealedName }}
      </h1>
      <p style="font-size: 1.3rem; font-weight: bold; color: #d32f2f;">📸 Tire um PRINT desta tela para não esquecer!</p>
      
      <button 
        @click="resetView" 
        style="margin-top: 30px; padding: 12px 20px; font-size: 16px; background: #333; color: white; border: none; border-radius: 5px; cursor: pointer; width: 100%;"
      >
        Sair e Ocultar
      </button>
    </div>

    <!-- SEÇÃO: Quem falta tirar -->
    <div v-if="!revealedName && !isAdmin" style="margin-top: 40px; border-top: 1px solid #ccc; padding-top: 20px;">
      <h3 style="color: #666; font-size: 1.1rem;">Ainda faltam tirar:</h3>
      
      <div style="display: flex; flex-wrap: wrap; justify-content: center; gap: 10px; margin-top: 15px;">
        <span 
          v-for="p in participants.filter(p => !p.hasSeen)" 
          :key="p.name" 
          style="background: #e0e0e0; padding: 8px 16px; border-radius: 20px; font-size: 14px; color: #424242; font-weight: bold;"
        >
          {{ p.name }}
        </span>
      </div>
    </div>

    <!-- BOTÃO SECRETO DO ADMIN (Rodapé) -->
    <div style="margin-top: 60px; font-size: 12px; color: #aaa;">
      <span @click="openAdmin" style="cursor: pointer; opacity: 0.5;">🔒</span>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import Swal from 'sweetalert2';

const participants = ref([]);
const selectedName = ref('');
const revealedName = ref('');
const savedLocally = ref(false);

// Variáveis do Admin
const isAdmin = ref(false);
const adminNamesList = ref('');

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
  }
});

const revealSecretSanta = async () => {
  const result = await Swal.fire({
    title: 'É você mesmo?',
    html: `Você confirma que é <b>${selectedName.value}</b>?<br><br><small style="color: #d33;">Atenção: Só é possível sortear uma vez!</small>`,
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
      body: JSON.stringify({ name: selectedName.value })
    });
    
    const data = await res.json();
    revealedName.value = data.drawnName;
    
    localStorage.setItem('meuNome', selectedName.value);
    localStorage.setItem('meuAmigoSecreto', data.drawnName);
    savedLocally.value = true;
    
    setTimeout(() => {
      Swal.fire({
        title: '📸 Hora do Print!',
        text: 'Tire um PRINT da tela agora para não esquecer quem você tirou!',
        icon: 'warning',
        confirmButtonColor: '#3085d6',
        confirmButtonText: 'Entendido!'
      });
    }, 500);

    await fetchParticipants();
  } catch (error) {
    console.error("Erro ao revelar:", error);
    Swal.fire('Erro!', 'Problema ao conectar com o servidor.', 'error');
  }
};

const showSavedResult = () => {
  revealedName.value = localStorage.getItem('meuAmigoSecreto');
  setTimeout(() => {
    Swal.fire({ title: 'Lembrete!', text: 'Não esqueça de tirar o PRINT desta tela!', icon: 'info', confirmButtonColor: '#3085d6', confirmButtonText: 'Ok' });
  }, 300);
};

const resetView = () => {
  revealedName.value = '';
};

// ==========================================
// FUNÇÕES DO ADMINISTRADOR
// ==========================================
const openAdmin = async () => {
  const { value: password } = await Swal.fire({
    title: 'Área Restrita',
    input: 'password',
    inputPlaceholder: 'Digite a senha',
    showCancelButton: true,
    confirmButtonText: 'Entrar',
    cancelButtonText: 'Sair'
  });

  // A SENHA PARA VOCÊ ENTRAR É: admin123
  if (password === 'admin123') {
    isAdmin.value = true;
    adminNamesList.value = participants.value.map(p => p.name).join('\n');
  } else if (password) {
    Swal.fire('Erro', 'Senha incorreta', 'error');
  }
};

const regerarSorteio = async () => {
  const nomesArray = adminNamesList.value.split('\n').map(n => n.trim()).filter(n => n !== '');
  
  if (nomesArray.length < 3) {
    return Swal.fire('Erro', 'Precisa de pelo menos 3 nomes para sortear.', 'warning');
  }

  const confirm = await Swal.fire({
    title: 'Atenção!',
    text: 'Isso vai apagar o sorteio atual de todo mundo e gerar uma nova lista. Tem certeza?',
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#E53935',
    confirmButtonText: 'Sim, refazer sorteio!'
  });

  if (!confirm.isConfirmed) return;

  try {
    const res = await fetch('https://natal-bl3x.onrender.com/api/admin/shuffle', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ password: 'admin123', names: nomesArray })
    });
    
    const data = await res.json();
    
    if (data.success) {
      Swal.fire('Sucesso!', 'Novo sorteio gerado com sucesso. O banco de dados foi atualizado!', 'success');
      isAdmin.value = false;
      
      // Limpa seu próprio navegador para você poder testar
      localStorage.removeItem('meuNome');
      localStorage.removeItem('meuAmigoSecreto');
      savedLocally.value = false;
      revealedName.value = '';
      selectedName.value = '';
      
      await fetchParticipants();
    } else {
      Swal.fire('Erro', data.error, 'error');
    }
  } catch (e) {
    Swal.fire('Erro', 'Falha ao conectar com o servidor.', 'error');
  }
};
</script>