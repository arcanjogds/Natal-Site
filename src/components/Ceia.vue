<template>
  <div style="background: #f1f8e9; padding: 20px; border-radius: 8px; border: 2px solid #8bc34a; margin-top: 20px; text-align: center;">
    <img src="/img/ceia.jpg" alt="Ceia de Natal" style="width: 100%; border-radius: 10px; margin-bottom: 15px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); object-fit: cover; max-height: 200px;">
    
    <h2 style="color: #33691e; margin-top: 0;">🍽️ Cardápio da Ceia</h2>
    <p style="color: #558b2f; margin-bottom: 20px; font-size: 1.1rem;">Escolha o que você vai levar para a nossa ceia!</p>

    <div v-for="prato in pratos" :key="prato._id" style="background: white; padding: 15px; margin-bottom: 12px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); display: flex; flex-direction: column; gap: 12px;">
      
      <!-- Linha Superior: Nome, Categoria e Controles -->
      <div style="display: flex; justify-content: space-between; align-items: flex-start; width: 100%;">
        <div style="text-align: left;">
          <strong style="font-size: 1.2rem; color: #2e7d32;">{{ prato.nomePrato }}</strong>
          <span style="display: block; font-size: 0.85rem; color: #757575; text-transform: uppercase; margin-top: 3px;">{{ prato.categoria }}</span>
        </div>
        <div style="display: flex; gap: 8px;">
          <button @click="editarPrato(prato)" style="background: #e3f2fd; border: 1px solid #bbdefb; color: #1976d2; cursor: pointer; font-size: 1rem; border-radius: 5px; padding: 4px 8px;" title="Editar Prato">✏️</button>
          <button @click="deletarPrato(prato._id)" style="background: #ffebee; border: 1px solid #ffcdd2; color: #f44336; cursor: pointer; font-size: 1rem; border-radius: 5px; padding: 4px 8px;" title="Remover Prato">✖</button>
        </div>
      </div>

      <!-- Linha Inferior: Botão Eu Levo ou Responsável + Desistir -->
      <div style="display: flex; align-items: center; justify-content: flex-end; width: 100%; border-top: 1px dashed #eee; padding-top: 10px;">
        <div v-if="prato.responsavel" style="display: flex; align-items: center; justify-content: space-between; width: 100%; gap: 10px; flex-wrap: wrap;">
          <span style="background: #e8f5e9; color: #2e7d32; padding: 6px 12px; border-radius: 20px; font-size: 0.9rem; font-weight: bold; border: 1px solid #c8e6c9;">
            ✅ {{ prato.responsavel }} vai levar
          </span>
          <button @click="desistirPrato(prato._id, prato.nomePrato)" style="background: transparent; border: 1px solid #f44336; color: #f44336; padding: 6px 12px; border-radius: 5px; font-size: 0.85rem; font-weight: bold; cursor: pointer; white-space: nowrap;">
            Desistir
          </button>
        </div>
        <button v-else @click="assumirPrato(prato._id, prato.nomePrato)" style="background: #ff9800; color: white; border: none; padding: 10px 15px; border-radius: 5px; font-weight: bold; cursor: pointer; width: 100%;">
          🙋 Eu levo!
        </button>
      </div>
    </div>

    <div v-if="pratos.length === 0" style="color: #999; margin: 20px 0;">O cardápio ainda está vazio.</div>

    <button @click="adicionarPrato" style="margin-top: 15px; background: #8bc34a; color: white; border: none; padding: 12px 20px; border-radius: 5px; font-weight: bold; font-size: 16px; cursor: pointer; width: 100%;">
      ➕ Adicionar item ao cardápio
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import Swal from 'sweetalert2';

const props = defineProps({
  participants: {
    type: Array,
    default: () => []
  }
});

const pratos = ref([]);
const apiUrl = 'https://natal-bl3x.onrender.com/api/ceia';

const nomesFamilia = computed(() => props.participants.map(p => p.name));

const fetchPratos = async () => {
  try {
    const res = await fetch(apiUrl);
    pratos.value = await res.json();
  } catch (error) { console.error("Erro:", error); }
};
onMounted(() => fetchPratos());

// FORMULÁRIO PARA ASSUMIR UM PRATO (Com a lista predefinida)
const assumirPrato = async (id, nomePrato) => {
  const { value: nome } = await Swal.fire({
    title: 'Confirmar Responsabilidade',
    html: `
      <div style="text-align: left;">
        <p style="margin-bottom: 10px; color: #333;">Quem vai levar: <b>${nomePrato}</b>?</p>
        <select id="swal-responsavel" class="swal2-select" style="width: 100%; max-width: 100%; margin: 0;">
          <option value="" disabled selected>Selecione seu nome...</option>
          ${nomesFamilia.value.map(n => `<option value="${n}">${n}</option>`).join('')}
        </select>
      </div>
    `,
    focusConfirm: false,
    showCancelButton: true,
    confirmButtonColor: '#ff9800',
    cancelButtonColor: '#d33',
    confirmButtonText: 'Confirmar!',
    preConfirm: () => {
      const val = document.getElementById('swal-responsavel').value;
      if (!val) Swal.showValidationMessage('Você precisa selecionar o seu nome!');
      return val;
    }
  });

  if (nome) {
    try {
      await fetch(`${apiUrl}/${id}/assumir`, { method: 'PUT', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ responsavel: nome }) });
      Swal.fire('Aí sim!', `${nomePrato} está garantido por ${nome}!`, 'success');
      fetchPratos();
    } catch (error) { Swal.fire('Erro', 'Não foi possível salvar.', 'error'); }
  }
};

// FORMULÁRIO ARRUMADO PARA ADICIONAR COMIDA (Caixas não cortam)
const adicionarPrato = async () => {
  const { value: formValues } = await Swal.fire({
    title: 'Adicionar à Ceia',
    html: `
      <div style="text-align: left;">
        <label style="font-weight: bold; font-size: 14px; color: #333;">O que é?</label>
        <input id="swal-input1" class="swal2-input" style="width: 100%; max-width: 100%; margin: 5px 0 15px 0; box-sizing: border-box;" placeholder="Ex: Salpicão">
        
        <label style="font-weight: bold; font-size: 14px; color: #333;">Categoria do prato:</label>
        <select id="swal-input2" class="swal2-select" style="width: 100%; max-width: 100%; margin: 5px 0 10px 0;">
          <option value="Principal">Prato Principal</option>
          <option value="Acompanhamento">Acompanhamento</option>
          <option value="Sobremesa">Sobremesa</option>
          <option value="Bebida">Bebida</option>
        </select>
      </div>
    `,
    focusConfirm: false,
    showCancelButton: true,
    confirmButtonColor: '#8bc34a',
    confirmButtonText: 'Adicionar',
    preConfirm: () => {
      const prato = document.getElementById('swal-input1').value;
      const categoria = document.getElementById('swal-input2').value;
      if (!prato) Swal.showValidationMessage('Você precisa digitar o nome do prato!');
      return [prato, categoria];
    }
  });

  if (formValues) {
    try {
      await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ nomePrato: formValues[0], categoria: formValues[1] }) });
      Swal.fire('Adicionado!', 'O item entrou para o cardápio.', 'success');
      fetchPratos();
    } catch (error) { Swal.fire('Erro', 'Falha ao adicionar item.', 'error'); }
  }
};

const editarPrato = async (prato) => {
  const { value: formValues } = await Swal.fire({
    title: 'Editar Prato',
    html: `
      <div style="text-align: left;">
        <label style="font-weight: bold; font-size: 14px; color: #333;">Nome do Prato:</label>
        <input id="swal-edit1" class="swal2-input" style="width: 100%; max-width: 100%; margin: 5px 0 15px 0; box-sizing: border-box;" value="${prato.nomePrato}">
        
        <label style="font-weight: bold; font-size: 14px; color: #333;">Categoria do prato:</label>
        <select id="swal-edit2" class="swal2-select" style="width: 100%; max-width: 100%; margin: 5px 0 10px 0;">
          <option value="Principal" ${prato.categoria === 'Principal' ? 'selected' : ''}>Prato Principal</option>
          <option value="Acompanhamento" ${prato.categoria === 'Acompanhamento' ? 'selected' : ''}>Acompanhamento</option>
          <option value="Sobremesa" ${prato.categoria === 'Sobremesa' ? 'selected' : ''}>Sobremesa</option>
          <option value="Bebida" ${prato.categoria === 'Bebida' ? 'selected' : ''}>Bebida</option>
        </select>
      </div>
    `,
    focusConfirm: false,
    showCancelButton: true,
    confirmButtonColor: '#8bc34a',
    confirmButtonText: 'Salvar',
    preConfirm: () => {
      const nome = document.getElementById('swal-edit1').value;
      const cat = document.getElementById('swal-edit2').value;
      if (!nome) Swal.showValidationMessage('Digite o nome do prato!');
      return { nomePrato: nome, categoria: cat };
    }
  });

  if (formValues) {
    try {
      await fetch(`${apiUrl}/${prato._id}`, { method: 'PUT', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(formValues) });
      Swal.fire('Atualizado!', 'Prato foi modificado.', 'success');
      fetchPratos();
    } catch (e) { Swal.fire('Erro', 'Não foi possível editar.', 'error'); }
  }
};

const deletarPrato = async (id) => {
  const confirm = await Swal.fire({ title: 'Apagar prato?', text: "Remover este item do cardápio?", icon: 'warning', showCancelButton: true, confirmButtonColor: '#d33', confirmButtonText: 'Sim, apagar!' });
  if (confirm.isConfirmed) {
    try { await fetch(`${apiUrl}/${id}`, { method: 'DELETE' }); fetchPratos(); } 
    catch (error) { Swal.fire('Erro', 'Não foi possível apagar.', 'error'); }
  }
};

const desistirPrato = async (id, nomePrato) => {
  const confirm = await Swal.fire({ title: 'Desistir?', text: `Deixar de levar o ${nomePrato}?`, icon: 'warning', showCancelButton: true, confirmButtonColor: '#ff9800', confirmButtonText: 'Sim, desistir' });
  if (confirm.isConfirmed) {
    try {
      await fetch(`${apiUrl}/${id}/assumir`, { method: 'PUT', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ responsavel: '' }) });
      Swal.fire('Pronto', `O ${nomePrato} está livre novamente.`, 'success');
      fetchPratos();
    } catch (e) { Swal.fire('Erro', 'Não foi possível atualizar.', 'error'); }
  }
};
</script>