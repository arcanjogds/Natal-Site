<template>
  <div class="card-natalino" style="padding: 20px; margin-top: 20px; text-align: center;">
    <h2 style="color: #333333; margin-top: 0;">🍽️ Cardápio da Ceia</h2>
    <p style="color: #333333; margin-bottom: 20px; font-size: 1.1rem;">Escolha o que você vai levar para a nossa ceia!</p>

    <!-- ÁREA DE FILTROS -->
    <div style="display: flex; gap: 10px; margin-bottom: 20px; flex-wrap: wrap;">
      <select v-model="filtroCategoria" style="flex: 1; min-width: 150px; padding: 10px; border-radius: 8px; font-size: 1rem;">
        <option value="Todas">Todas as Categorias</option>
        <option value="Principal">Principal</option>
        <option value="Acompanhamento">Acompanhamento</option>
        <option value="Sobremesa">Sobremesa</option>
        <option value="Bebida">Bebida</option>
      </select>

      <select v-model="filtroResponsavel" style="flex: 1; min-width: 150px; padding: 10px; border-radius: 8px; font-size: 1rem;">
        <option value="Todos">Todas as Pessoas</option>
        <option value="Sem Ninguém">❌ Sem ninguém (Livres)</option>
        <option v-for="nome in nomesFamilia" :key="nome" :value="nome">🙋 {{ nome }}</option>
      </select>
    </div>

    <div v-for="prato in pratosFiltrados" :key="prato._id" style="background: #fff9f0; padding: 15px; margin-bottom: 12px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); display: flex; flex-direction: column; gap: 12px;">
      
      <!-- Linha Superior: Nome, Categoria e Controles -->
      <div style="display: flex; justify-content: space-between; align-items: flex-start; width: 100%;">
        <div style="text-align: left;">
          <strong style="font-size: 1.2rem; color: #2e7d32;">{{ prato.nomePrato }} <span v-if="calcularTotalPrato(prato) > 0">({{ calcularTotalPrato(prato) }}x)</span></strong>
          <span style="display: block; font-size: 0.85rem; color: #757575; text-transform: uppercase; margin-top: 3px;">{{ prato.categoria }}</span>
        </div>
        <div style="display: flex; gap: 8px;">
          <button @click="editarPrato(prato)" style="background: #e8f5e9; border: 1px solid #c8e6c9; color: #2e7d32; cursor: pointer; font-size: 1rem; border-radius: 5px; padding: 4px 8px;" title="Editar Prato">✏️</button>
          <button @click="deletarPrato(prato._id)" style="background: #ffebee; border: 1px solid #ffcdd2; color: #c62828; cursor: pointer; font-size: 1rem; border-radius: 5px; padding: 4px 8px;" title="Remover Prato">✖</button>
        </div>
      </div>

      <!-- Linha Inferior: Botão Eu Levo ou Responsável + Desistir -->
      <div style="width: 100%; border-top: 1px dashed #eee; padding-top: 10px;">
        <div v-for="resp in prato.responsaveis" :key="resp.nome" style="display: flex; align-items: center; justify-content: space-between; width: 100%; gap: 10px; flex-wrap: wrap; margin-bottom: 8px;">
          <span style="background: #e8f5e9; color: #2e7d32; padding: 6px 12px; border-radius: 20px; font-size: 0.9rem; font-weight: bold; border: 1px solid #c8e6c9;">
            ✅ {{ resp.nome }} vai levar {{ resp.quantidade }}x
          </span>
          <button v-if="usuarioAtual === resp.nome" @click="desistirPrato(prato, resp.nome)" style="background: transparent; border: 1px solid #f44336; color: #f44336; padding: 6px 12px; border-radius: 5px; font-size: 0.85rem; font-weight: bold; cursor: pointer; white-space: nowrap;">
            Desistir
          </button>
        </div>
        <button @click="assumirPrato(prato)" style="background: #2e7d32; color: white; border: none; padding: 10px 15px; border-radius: 5px; font-weight: bold; cursor: pointer; width: 100%; margin-top: 5px;">
          🙋 Eu levo!
        </button>
      </div>
    </div>

    <div v-if="pratosFiltrados.length === 0" style="color: #999; margin: 20px 0;">O cardápio ainda está vazio ou nenhum prato corresponde ao filtro.</div>

  </div>

  <!-- BOTAO FLUTUANTE (FAB) -->
  <teleport to="body">
    <div class="fab-container">
      <button @click="adicionarPrato" class="fab-btn" style="background: #2e7d32; color: white; box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);">
        <span style="font-size: 1.2rem;">➕</span>
        <span class="fab-text">Adicionar item</span>
      </button>
    </div>
  </teleport>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import Swal from 'sweetalert2';

const props = defineProps({
  participants: {
    type: Array,
    default: () => []
  },
  usuarioAtual: {
    type: String,
    default: ''
  }
});

const pratos = ref([]);
const BASE_URL = window.location.hostname === 'localhost' || window.location.hostname === '127.0.0.1' ? 'http://localhost:3000' : 'https://natal-bl3x.onrender.com';
const apiUrl = `${BASE_URL}/api/ceia`;

const nomesFamilia = computed(() => props.participants.map(p => p.name));

const filtroCategoria = ref('Todas');
const filtroResponsavel = ref('Todos');

const pratosFiltrados = computed(() => {
  return pratos.value.filter(prato => {
    if (filtroCategoria.value !== 'Todas' && prato.categoria !== filtroCategoria.value) {
      return false;
    }
    if (filtroResponsavel.value === 'Sem Ninguém') {
      if (prato.responsaveis && prato.responsaveis.length > 0) return false;
    } else if (filtroResponsavel.value !== 'Todos') {
      if (!prato.responsaveis || !prato.responsaveis.some(r => r.nome === filtroResponsavel.value)) return false;
    }
    return true;
  });
});

const calcularTotalPrato = (prato) => {
  if (!prato.responsaveis) return 0;
  return prato.responsaveis.reduce((soma, pessoa) => soma + (pessoa.quantidade || 1), 0);
};

const fetchPratos = async () => {
  try {
    const res = await fetch(apiUrl);
    pratos.value = await res.json();
  } catch (error) { console.error("Erro:", error); }
};
onMounted(() => fetchPratos());

// FORMULÁRIO PARA ASSUMIR UM PRATO
const assumirPrato = async (prato) => {
  if (!props.usuarioAtual) {
    Swal.fire('Identifique-se!', 'Por favor, selecione quem é você no canto superior direito da página antes de assumir um prato.', 'warning');
    return;
  }
  
  if (prato.responsaveis && prato.responsaveis.some(r => r.nome === props.usuarioAtual)) {
    Swal.fire('Opa!', 'Você já assumiu este prato. Se quiser mudar a quantidade, desista e assuma novamente.', 'info');
    return;
  }

  const { value: formValues } = await Swal.fire({
    title: 'Confirmar Responsabilidade',
    html: `
      <p style="font-size: 1.1rem; color: #333;">Você (${props.usuarioAtual}) vai levar: <strong>${prato.nomePrato}</strong></p>
      <label style="font-weight: bold; font-size: 14px; color: #333; margin-top: 15px; display: block; text-align: left;">Quantidade (ex: 2 refrigerantes, 1 travessa):</label>
      <input id="swal-qtd" type="number" min="1" class="swal2-input" style="width: 100%; max-width: 100%; margin: 5px 0; box-sizing: border-box;" value="1">
    `,
    icon: 'question',
    showCancelButton: true,
    confirmButtonColor: '#ff9800',
    cancelButtonColor: '#d33',
    confirmButtonText: 'Sim, eu levo!',
    cancelButtonText: 'Cancelar',
    preConfirm: () => {
      const qtd = document.getElementById('swal-qtd').value;
      if (!qtd || qtd < 1) {
        Swal.showValidationMessage('A quantidade deve ser pelo menos 1!');
      }
      return qtd;
    }
  });

  if (formValues) {
    try {
      const novosResponsaveis = prato.responsaveis ? [...prato.responsaveis] : [];
      novosResponsaveis.push({ nome: props.usuarioAtual, quantidade: parseInt(formValues) });
      
      await fetch(`${apiUrl}/${prato._id}/assumir`, { 
          method: 'PUT', 
          headers: { 'Content-Type': 'application/json' }, 
          body: JSON.stringify({ responsaveis: novosResponsaveis }) 
      });
      Swal.fire('Aí sim!', `${prato.nomePrato} está garantido por ${props.usuarioAtual}!`, 'success');
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

const desistirPrato = async (prato, nomeDesistente) => {
  const confirm = await Swal.fire({ title: 'Desistir?', text: `Deixar de levar o ${prato.nomePrato}?`, icon: 'warning', showCancelButton: true, confirmButtonColor: '#ff9800', confirmButtonText: 'Sim, desistir' });
  if (confirm.isConfirmed) {
    try {
      const novosResponsaveis = prato.responsaveis.filter(r => r.nome !== nomeDesistente);
      
      await fetch(`${apiUrl}/${prato._id}/assumir`, { 
          method: 'PUT', 
          headers: { 'Content-Type': 'application/json' }, 
          body: JSON.stringify({ responsaveis: novosResponsaveis }) 
      });
      Swal.fire('Pronto', `Você desistiu de ${prato.nomePrato}.`, 'success');
      fetchPratos();
    } catch (e) { Swal.fire('Erro', 'Não foi possível atualizar.', 'error'); }
  }
};
</script>

<style scoped>
.fab-container {
  position: fixed;
  bottom: 24px;
  right: 24px;
  z-index: 9999;
}
.fab-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  border: none;
  border-radius: 50px;
  padding: 15px 25px;
  font-weight: bold;
  font-size: 16px;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(0,0,0,0.3);
  transition: all 0.3s ease;
}
.fab-text {
  display: inline;
}

@media (max-width: 600px) {
  .fab-btn {
    width: 60px;
    height: 60px;
    padding: 0;
    justify-content: center;
    border-radius: 50%;
  }
  .fab-text {
    display: none;
  }
}
</style>