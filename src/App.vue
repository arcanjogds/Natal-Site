<template>
  <div class="container" style="text-align: center; font-family: sans-serif; padding: 2rem; max-width: 600px; margin: auto;">
    <h1>🎄 Amigo Secreto Natal 2026 🎄</h1>
    
    <!-- Tela 1: Já tem um sorteio salvo -->
    <div v-if="savedLocally && !revealedName" style="background: #e3f2fd; padding: 20px; border-radius: 8px; border: 1px solid #90caf9; margin-top: 20px;">
      <p style="font-size: 1.2rem; color: #1565c0; font-weight: bold;">Você já tirou seu amigo secreto neste aparelho!</p>
      <button 
        @click="showSavedResult" 
        style="margin-top: 10px; padding: 15px 20px; font-size: 16px; cursor: pointer; background: #1976D2; color: white; border: none; border-radius: 8px; width: 100%;"
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

      <!-- Mensagem de sucesso quando todos finalizam -->
      <p v-if="participants.length > 0 && participants.filter(p => !p.hasSeen).length === 0" style="color: #4CAF50; font-weight: bold; font-size: 1.3rem; margin-top: 20px;">
        Todo mundo já tirou! 🎉
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

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
  if (!confirm(`Você confirma que é ${selectedName.value}? Só é possível ver na lista uma vez!`)) return;

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
      alert("📸 ATENÇÃO: Tire um PRINT da tela agora para não esquecer quem você tirou!");
    }, 300);

    await fetchParticipants();
  } catch (error) {
    console.error("Erro ao revelar:", error);
    alert("Erro ao conectar com o servidor.");
  }
};

const showSavedResult = () => {
  revealedName.value = localStorage.getItem('meuAmigoSecreto');
  setTimeout(() => {
    alert("📸 Lembrete: Tire um PRINT da tela para não esquecer!");
  }, 300);
};

const resetView = () => {
  revealedName.value = '';
};
</script>