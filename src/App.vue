<template>
  <div class="container" style="text-align: center; font-family: sans-serif; padding: 2rem; max-width: 600px; margin: auto;">
    <h1>🎄 Amigo Secreto da Galera 🎄</h1>
    
    <!-- MURAL DE REGRAS (NOVO) -->
    <div style="background: #fff3e0; padding: 15px; border-radius: 8px; border: 2px dashed #ff9800; margin-bottom: 25px; box-shadow: 0 4px 6px rgba(0,0,0,0.05);">
      <h3 style="color: #e65100; margin: 0 0 10px 0;">📜 Regra do Sorteio</h3>
      <p style="margin: 0; font-size: 1.2rem; color: #d84315; font-weight: bold;">
        🎁 Valor do presente: Mínimo de R$ 150,00
      </p>
    </div>

    <!-- Tela 1: Já tem um sorteio salvo -->
    <div v-if="savedLocally && !revealedName" style="background: #e3f2fd; padding: 20px; border-radius: 8px; border: 1px solid #90caf9; margin-top: 20px;">
      <p style="font-size: 1.2rem; color: #1565c0; font-weight: bold;">Você já tirou seu amigo secreto neste aparelho!</p>
      <button 
        @click="showSavedResult" 
        style="margin-top: 10px; padding: 15px 20px; font-size: 16px; cursor: pointer; background: #1976D2; color: white; border: none; border-radius: 8px; width: 100%; font-weight: bold;"
      >
        👀 Ver meu resultado novamente
      </button>
    </div>

    <!-- Tela 2: Novo Sorteio -->
    <div v-if="!revealedName && !savedLocally">
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
    <div v-if="revealedName">
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
    <div v-if="!revealedName" style="margin-top: 40px; border-top: 1px solid #ccc; padding-top: 20px;">
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

      <p v-if="participants.length > 0 && participants.filter(p => !p.hasSeen).length === 0" style="color: #4CAF50; font-weight: bold; font-size: 1.3rem; margin-top: 20px;">
        Todo mundo já tirou! 🎉
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import Swal from 'sweetalert2'; // Importando a biblioteca de pop-ups maravilhosos!

const participants = ref([]);
const selectedName = ref('');
const revealedName = ref('');
const savedLocally = ref(false);

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
  // POP-UP ANIMADO DE CONFIRMAÇÃO
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
    
    // POP-UP ANIMADO COBRANDO O PRINT
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
    Swal.fire({
      title: 'Lembrete!',
      text: 'Não esqueça de tirar o PRINT desta tela!',
      icon: 'info',
      confirmButtonColor: '#3085d6',
      confirmButtonText: 'Ok'
    });
  }, 300);
};

const resetView = () => {
  revealedName.value = '';
};
</script>