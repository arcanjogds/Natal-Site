<template>
  <div style="background: #e3f2fd; padding: 20px; border-radius: 8px; border: 2px solid #2196f3; margin-top: 20px; text-align: center;">
    <h2 style="color: #0d47a1; margin-top: 0;">🛍️ Vitrine de Presentes</h2>
    <p style="color: #1565c0; margin-bottom: 20px; font-size: 1.1rem;">Dicas do que a família quer ganhar!</p>

    <!-- FILTRO POR NOME -->
    <div style="margin-bottom: 20px;">
      <select v-model="filtroFamiliar" style="width: 100%; max-width: 300px; padding: 10px; font-size: 1rem; border-radius: 8px; border: 1px solid #90caf9; color: #1565c0; font-weight: bold;">
        <option value="">🎁 Ver lista de todos</option>
        <option v-for="nome in nomesFamilia" :key="nome" :value="nome">Apenas de {{ nome }}</option>
      </select>
    </div>

    <!-- LISTA DE CARDS POR PESSOA -->
    <div style="display: flex; flex-direction: column; gap: 20px; text-align: left;">
      <div v-for="nome in nomesFiltrados" :key="nome" style="background: white; border-radius: 12px; overflow: hidden; box-shadow: 0 4px 10px rgba(0,0,0,0.1); border: 1px solid #bbdefb; display: flex; flex-direction: column; width: 100%;">
        
        <!-- HEADER DO CARD (NOME) -->
        <div style="background: #2196f3; padding: 15px; color: white; display: flex; justify-content: center; align-items: center;">
          <h3 style="margin: 0; font-size: 1.4rem;">{{ nome }}</h3>
        </div>

        <!-- LISTA DE PRESENTES DESSA PESSOA -->
        <div style="padding: 15px; flex-grow: 1; background: #fafafa;">
          <div v-if="getPresentes(nome).length === 0" style="color: #999; font-size: 0.9rem; font-style: italic; text-align: center; margin-top: 10px;">
            Ainda não pediu nada.
          </div>
          <div style="display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 15px;">
            <div v-for="presente in getPresentes(nome)" :key="presente._id" style="background: white; padding: 12px; border-radius: 8px; border: 1px solid #e0e0e0; position: relative; box-shadow: 0 2px 5px rgba(0,0,0,0.05);">
              <div style="position: absolute; top: 5px; right: 5px; display: flex; gap: 8px;">
                <button @click="editarPresente(presente)" style="background: transparent; border: none; color: #1976d2; cursor: pointer; font-size: 1rem;" title="Editar pedido">✏️</button>
                <button @click="deletarPresente(presente._id)" style="background: transparent; border: none; color: #f44336; cursor: pointer; font-size: 1.1rem;" title="Remover pedido">✖</button>
              </div>
              <p style="margin: 0; font-weight: bold; color: #333; padding-right: 45px;">{{ presente.item }}</p>
              <p v-if="presente.tamanhoEspecificacao" style="margin: 5px 0 0 0; font-size: 0.85rem; color: #666;">📝 {{ presente.tamanhoEspecificacao }}</p>
              <a v-if="presente.linkLoja" :href="presente.linkLoja" target="_blank" style="display: inline-block; margin-top: 8px; color: #1976d2; font-size: 0.85rem; text-decoration: none; font-weight: bold; border-bottom: 1px solid #1976d2;">🛒 Ver na Loja</a>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- BOTAO FLUTUANTE (FAB) -->
    <div class="fab-container">
      <button @click="adicionarPresente" class="fab-btn" style="background: #2196f3; color: white;">
        <span style="font-size: 1.2rem;">➕</span>
        <span class="fab-text">Adicionar Pedido</span>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import Swal from 'sweetalert2';

const props = defineProps({
  participants: {
    type: Array,
    default: () => []
  },
  usuarioAtual: {
    type: String,
    default: ''
  },
  filtroInicial: {
    type: String,
    default: ''
  }
});

const presentes = ref([]);
const apiUrl = 'https://natal-bl3x.onrender.com/api/presentes';

const nomesFamilia = computed(() => props.participants.map(p => p.name));

const filtroFamiliar = ref(props.filtroInicial);

const nomesFiltrados = computed(() => {
  if (filtroFamiliar.value) return [filtroFamiliar.value];
  return nomesFamilia.value;
});

const getPresentes = (nome) => {
  return presentes.value.filter(p => p.nomeFamiliar === nome);
};

const fetchPresentes = async () => {
  try {
    const res = await fetch(apiUrl);
    presentes.value = await res.json();
  } catch (error) { console.error(error); }
};
onMounted(() => fetchPresentes());

// FORMULÁRIO ARRUMADO SEM PRECISAR ESCOLHER NOME
const adicionarPresente = async () => {
  if (!props.usuarioAtual) {
    Swal.fire('Identifique-se!', 'Por favor, selecione quem é você no canto superior direito da página antes de adicionar um pedido.', 'warning');
    return;
  }

  const { value: formValues } = await Swal.fire({
    title: 'Sua lista de desejos',
    html: `
      <div style="text-align: left;">
        <label style="font-weight: bold; font-size: 14px; color: #333;">O que você quer?</label>
        <input id="swal-item" class="swal2-input" style="width: 100%; max-width: 100%; margin: 5px 0 15px 0; box-sizing: border-box;" placeholder="Ex: Perfume Boticário">

        <label style="font-weight: bold; font-size: 14px; color: #333;">Tamanho ou Detalhe:</label>
        <input id="swal-espec" class="swal2-input" style="width: 100%; max-width: 100%; margin: 5px 0 15px 0; box-sizing: border-box;" placeholder="Ex: 220v, Tam M">

        <label style="font-weight: bold; font-size: 14px; color: #333;">Link da loja (Opcional):</label>
        <input id="swal-link" class="swal2-input" style="width: 100%; max-width: 100%; margin: 5px 0 10px 0; box-sizing: border-box;" placeholder="Cole o link aqui...">
      </div>
    `,
    focusConfirm: false,
    showCancelButton: true,
    confirmButtonColor: '#2196f3',
    confirmButtonText: 'Salvar Pedido',
    cancelButtonText: 'Cancelar',
    preConfirm: () => {
      const nomeFamiliar = props.usuarioAtual;
      const item = document.getElementById('swal-item').value;
      const tamanhoEspecificacao = document.getElementById('swal-espec').value;
      let linkLoja = document.getElementById('swal-link').value;
      
      if (!item) {
        Swal.showValidationMessage('O item é obrigatório!');
        return false;
      }
      if (linkLoja && !linkLoja.startsWith('http')) linkLoja = 'https://' + linkLoja;
      return { nomeFamiliar, item, tamanhoEspecificacao, linkLoja };
    }
  });

  if (formValues) {
    try {
      await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(formValues) });
      Swal.fire('Pronto!', 'Seu pedido já está na vitrine!', 'success');
      fetchPresentes();
    } catch (error) { Swal.fire('Erro', 'Falha ao conectar.', 'error'); }
  }
};

const deletarPresente = async (id) => {
  const confirm = await Swal.fire({ title: 'Apagar pedido?', text: "Remover este item?", icon: 'warning', showCancelButton: true, confirmButtonColor: '#d33', confirmButtonText: 'Sim, apagar!' });
  if (confirm.isConfirmed) {
    try { await fetch(`${apiUrl}/${id}`, { method: 'DELETE' }); fetchPresentes(); } 
    catch (error) { Swal.fire('Erro', 'Não foi possível apagar.', 'error'); }
  }
};

const editarPresente = async (presente) => {
  const { value: formValues } = await Swal.fire({
    title: 'Editar Pedido',
    html: `
      <div style="text-align: left;">
        <label style="font-weight: bold; font-size: 14px; color: #333;">O que você quer?</label>
        <input id="swal-edit-item" class="swal2-input" style="width: 100%; max-width: 100%; margin: 5px 0 15px 0; box-sizing: border-box;" value="${presente.item}">

        <label style="font-weight: bold; font-size: 14px; color: #333;">Tamanho ou Detalhe:</label>
        <input id="swal-edit-espec" class="swal2-input" style="width: 100%; max-width: 100%; margin: 5px 0 15px 0; box-sizing: border-box;" value="${presente.tamanhoEspecificacao || ''}">

        <label style="font-weight: bold; font-size: 14px; color: #333;">Link da loja (Opcional):</label>
        <input id="swal-edit-link" class="swal2-input" style="width: 100%; max-width: 100%; margin: 5px 0 10px 0; box-sizing: border-box;" value="${presente.linkLoja || ''}">
      </div>
    `,
    focusConfirm: false,
    showCancelButton: true,
    confirmButtonColor: '#2196f3',
    confirmButtonText: 'Salvar',
    preConfirm: () => {
      const item = document.getElementById('swal-edit-item').value;
      const tamanhoEspecificacao = document.getElementById('swal-edit-espec').value;
      let linkLoja = document.getElementById('swal-edit-link').value;
      
      if (!item) {
        Swal.showValidationMessage('O item é obrigatório!');
        return false;
      }
      if (linkLoja && !linkLoja.startsWith('http')) linkLoja = 'https://' + linkLoja;
      return { item, tamanhoEspecificacao, linkLoja };
    }
  });

  if (formValues) {
    try {
      await fetch(`${apiUrl}/${presente._id}`, { method: 'PUT', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(formValues) });
      Swal.fire('Atualizado!', 'Pedido modificado com sucesso.', 'success');
      fetchPresentes();
    } catch (error) { Swal.fire('Erro', 'Falha ao conectar.', 'error'); }
  }
};
</script>

<style scoped>
.fab-container {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 1000;
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