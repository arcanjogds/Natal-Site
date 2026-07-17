<template>
  <div style="background: #f1f8e9; padding: 20px; border-radius: 8px; border: 2px solid #8bc34a; margin-top: 20px; text-align: center;">
    <h2 style="color: #33691e; margin-top: 0;">🍽️ Cardápio da Ceia</h2>
    <p style="color: #558b2f; margin-bottom: 20px; font-size: 1.1rem;">Escolha o que você vai levar para a nossa ceia!</p>

    <!-- Lista de Pratos -->
    <div v-for="prato in pratos" :key="prato._id" style="background: white; padding: 15px; margin-bottom: 12px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 10px;">
      <div style="text-align: left;">
        <strong style="font-size: 1.2rem; color: #2e7d32;">{{ prato.nomePrato }}</strong>
        <span style="display: block; font-size: 0.85rem; color: #757575; text-transform: uppercase; margin-top: 3px;">{{ prato.categoria }}</span>
      </div>

      <!-- Exibe quem assumiu ou o botão para assumir -->
      <div v-if="prato.responsavel">
        <span style="background: #e8f5e9; color: #2e7d32; padding: 8px 12px; border-radius: 20px; font-size: 0.9rem; font-weight: bold; border: 1px solid #c8e6c9;">
          ✅ {{ prato.responsavel }} vai levar
        </span>
      </div>
      <button v-else @click="assumirPrato(prato._id, prato.nomePrato)" style="background: #ff9800; color: white; border: none; padding: 10px 15px; border-radius: 5px; font-weight: bold; cursor: pointer; transition: 0.2s;">
        🙋 Eu levo!
      </button>
    </div>

    <div v-if="pratos.length === 0" style="color: #999; margin: 20px 0;">
      O cardápio ainda está vazio.
    </div>

    <!-- Botão para adicionar novo prato -->
    <button @click="adicionarPrato" style="margin-top: 15px; background: #8bc34a; color: white; border: none; padding: 12px 20px; border-radius: 5px; font-weight: bold; font-size: 16px; cursor: pointer; width: 100%;">
      ➕ Adicionar item ao cardápio
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import Swal from 'sweetalert2';

// Estado reativo da lista
const pratos = ref([]);
// A URL do seu backend já no ar
const apiUrl = 'https://natal-bl3x.onrender.com/api/ceia';

// Função para ler o banco de dados
const fetchPratos = async () => {
  try {
    const res = await fetch(apiUrl);
    pratos.value = await res.json();
  } catch (error) {
    console.error("Erro ao buscar pratos:", error);
  }
};

// Busca os pratos assim que o componente carrega na tela
onMounted(() => {
  fetchPratos();
});

// Função para cadastrar que a pessoa vai levar a comida
const assumirPrato = async (id, nomePrato) => {
  const { value: nome } = await Swal.fire({
    title: 'Confirmar Responsabilidade',
    text: `Escreva seu nome para confirmar que você levará: ${nomePrato}`,
    input: 'text',
    inputPlaceholder: 'Seu nome...',
    showCancelButton: true,
    confirmButtonColor: '#ff9800',
    cancelButtonColor: '#d33',
    confirmButtonText: 'Confirmar!',
    cancelButtonText: 'Cancelar'
  });

  if (nome && nome.trim() !== '') {
    try {
      await fetch(`${apiUrl}/${id}/assumir`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ responsavel: nome.trim() })
      });
      Swal.fire('Aí sim!', `${nomePrato} está garantido por ${nome}!`, 'success');
      fetchPratos(); // Atualiza a lista na tela automaticamente
    } catch (error) {
      Swal.fire('Erro', 'Não foi possível salvar.', 'error');
    }
  }
};

// Função para adicionar uma nova comida/bebida na lista
const adicionarPrato = async () => {
  const { value: formValues } = await Swal.fire({
    title: 'Adicionar à Ceia',
    html:
      '<input id="swal-input1" class="swal2-input" placeholder="O que é? (Ex: Salpicão)">' +
      '<select id="swal-input2" class="swal2-input" style="width: 73%;">' +
      '<option value="Principal">Prato Principal</option>' +
      '<option value="Acompanhamento">Acompanhamento</option>' +
      '<option value="Sobremesa">Sobremesa</option>' +
      '<option value="Bebida">Bebida</option>' +
      '</select>',
    focusConfirm: false,
    showCancelButton: true,
    confirmButtonColor: '#8bc34a',
    confirmButtonText: 'Adicionar',
    preConfirm: () => {
      const prato = document.getElementById('swal-input1').value;
      const categoria = document.getElementById('swal-input2').value;
      if (!prato) {
        Swal.showValidationMessage('Você precisa digitar o nome do prato!');
      }
      return [prato, categoria];
    }
  });

  if (formValues) {
    try {
      await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ nomePrato: formValues[0], categoria: formValues[1] })
      });
      Swal.fire('Adicionado!', 'O item entrou para o cardápio da família.', 'success');
      fetchPratos();
    } catch (error) {
      Swal.fire('Erro', 'Falha ao adicionar item.', 'error');
    }
  }
};
</script>