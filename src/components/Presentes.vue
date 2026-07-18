<template>
  <div class="card-natalino" style="padding: 20px; margin-top: 20px; text-align: center;">
    <h2 style="color: #0d47a1; margin-top: 0;">🛍️ Vitrine de Presentes</h2>
    <p style="color: #1565c0; margin-bottom: 20px; font-size: 1.1rem;">Dicas do que a família quer ganhar!</p>

    <!-- FILTRO POR NOME -->
    <div style="margin-bottom: 20px;">
      <select v-model="filtroFamiliar" style="width: 100%; max-width: 300px; padding: 10px; font-size: 1rem; border-radius: 8px; font-weight: bold;">
        <option value="">🎁 Ver lista de todos</option>
        <option v-for="nome in nomesFamilia" :key="nome" :value="nome">Apenas de {{ nome }}</option>
      </select>
    </div>

    <!-- LISTA DE CARDS POR PESSOA -->
    <div style="display: flex; flex-direction: column; gap: 20px; text-align: left;">
      <div v-for="nome in nomesFiltrados" :key="nome" style="background: #fff9f0; border-radius: 12px; overflow: hidden; box-shadow: 0 4px 10px rgba(0,0,0,0.1); border: 1px solid var(--bg-natal); display: flex; flex-direction: column; width: 100%;">
        
        <!-- HEADER DO CARD (NOME) -->
        <div style="background: var(--bg-natal); padding: 15px; color: #f3e5ab; display: flex; justify-content: center; align-items: center;">
          <h3 style="margin: 0; font-size: 1.4rem;">{{ nome }}</h3>
        </div>

        <!-- LISTA DE PRESENTES DESSA PESSOA -->
        <div style="padding: 15px; flex-grow: 1;">
          <div v-if="getPresentes(nome).length === 0" style="color: #999; font-size: 0.9rem; font-style: italic; text-align: center; margin-top: 10px;">
            Ainda não pediu nada.
          </div>
          <div style="display: flex; flex-direction: column; gap: 15px;">
            <div v-for="kit in getPresentes(nome)" :key="kit._id" style="background: #fff; padding: 15px; border-radius: 8px; border: 1px solid #e5c09e; position: relative; box-shadow: 0 2px 5px rgba(0,0,0,0.05);">
              
              <div v-if="usuarioAtual === nome" style="position: absolute; top: 10px; right: 10px; display: flex; gap: 8px;">
                <button @click="editarKit(kit)" style="background: transparent; border: none; color: #2e7d32; cursor: pointer; font-size: 1rem;" title="Editar meta e nome do pedido">✏️</button>
                <button @click="deletarPresente(kit._id)" style="background: transparent; border: none; color: #c62828; cursor: pointer; font-size: 1.1rem;" title="Remover pedido completo">✖</button>
              </div>

              <h4 style="margin: 0 0 10px 0; color: #8b0000; font-size: 1.2rem; padding-right: 50px;">{{ kit.nomeKit }}</h4>
              
              <!-- Barra de Progresso -->
              <div style="margin-bottom: 15px;">
                <div style="display: flex; justify-content: space-between; font-size: 0.9rem; color: #333; margin-bottom: 5px; font-weight: bold;">
                  <span>Total: R$ {{ calcularSomaKit(kit).toFixed(2).replace('.', ',') }}</span>
                  <span>Meta: R$ {{ (kit.meta || 150).toFixed(2).replace('.', ',') }}</span>
                </div>
                <div style="width: 100%; background-color: #e0e0e0; border-radius: 10px; overflow: hidden; height: 12px;">
                  <div :style="{ width: Math.min((calcularSomaKit(kit) / (kit.meta || 150)) * 100, 100) + '%', backgroundColor: calcularSomaKit(kit) >= (kit.meta || 150) ? '#4CAF50' : '#ff9800', height: '100%', transition: 'width 0.3s' }"></div>
                </div>
              </div>

              <!-- Itens -->
              <div style="display: flex; flex-direction: column; gap: 10px;">
                <div v-for="(item, idx) in kit.itens" :key="idx" style="background: #fff9f0; padding: 10px; border-radius: 5px; border: 1px solid #ccc; position: relative;">
                   <button v-if="usuarioAtual === nome" @click="deletarSubItem(kit, idx)" style="position: absolute; top: 5px; right: 5px; background: transparent; border: none; color: #c62828; cursor: pointer; font-size: 0.9rem;" title="Remover item">✖</button>
                   <p style="margin: 0; font-weight: bold; color: #333; padding-right: 20px;">{{ item.item }}</p>
                   <p v-if="item.valor" style="margin: 5px 0 0 0; font-size: 0.95rem; color: #2e7d32; font-weight: bold;">💰 R$ {{ item.valor.toFixed(2).replace('.', ',') }}</p>
                   <p v-if="item.tamanhoEspecificacao" style="margin: 5px 0 0 0; font-size: 0.85rem; color: #333;">📝 {{ item.tamanhoEspecificacao }}</p>
                   <a v-if="item.linkLoja" :href="item.linkLoja" target="_blank" style="display: inline-block; margin-top: 8px; color: #8b0000; font-size: 0.85rem; text-decoration: none; font-weight: bold; border-bottom: 1px solid #8b0000;">🛒 Ver na Loja</a>
                </div>
              </div>

              <button v-if="usuarioAtual === nome" @click="adicionarSubItem(kit)" style="margin-top: 15px; background: transparent; color: #2e7d32; border: 2px dashed #2e7d32; padding: 8px; border-radius: 5px; cursor: pointer; width: 100%; font-weight: bold;">➕ Adicionar item ao pacote</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- BOTAO FLUTUANTE (FAB) -->
  <teleport to="body">
    <div class="fab-container">
      <button @click="adicionarPresente" class="fab-btn" style="background: #2e7d32; color: white; box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);">
        <span style="font-size: 1.2rem;">➕</span>
        <span class="fab-text">Criar Novo Pedido</span>
      </button>
    </div>
  </teleport>
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
const BASE_URL = window.location.hostname === 'localhost' || window.location.hostname === '127.0.0.1' ? 'http://localhost:3000' : 'https://natal-bl3x.onrender.com';
const apiUrl = `${BASE_URL}/api/presentes`;

const nomesFamilia = computed(() => props.participants.map(p => p.name));

const filtroFamiliar = ref(props.filtroInicial);

const nomesFiltrados = computed(() => {
  if (filtroFamiliar.value) return [filtroFamiliar.value];
  return nomesFamilia.value;
});

const calcularSomaKit = (kit) => {
  if (!kit || !kit.itens) return 0;
  return kit.itens.reduce((acc, curr) => acc + (curr.valor || 0), 0);
};

const getPresentes = (nome) => {
  const pedidos = presentes.value.filter(p => p.nomeFamiliar === nome);
  // Ordena pelo valor total dos itens (crescente)
  return pedidos.sort((a, b) => calcularSomaKit(a) - calcularSomaKit(b));
};

const fetchPresentes = async () => {
  try {
    const res = await fetch(apiUrl);
    presentes.value = await res.json();
  } catch (error) { console.error(error); }
};
onMounted(() => fetchPresentes());

const adicionarPresente = async () => {
  if (!props.usuarioAtual) {
    Swal.fire('Identifique-se!', 'Por favor, selecione quem é você no canto superior direito da página antes de criar um pedido.', 'warning');
    return;
  }

  const { value: formValues } = await Swal.fire({
    title: 'Criar Novo Pedido / Kit',
    html: `
      <div style="text-align: left;">
        <label style="font-weight: bold; font-size: 14px; color: #333;">Nome do Pedido (Ex: Kit Verão):</label>
        <input id="swal-nome-kit" class="swal2-input" style="width: 100%; box-sizing: border-box; margin-bottom: 10px;" placeholder="Opcional">
        
        <label style="font-weight: bold; font-size: 14px; color: #333;">Meta de Valor Total (R$):</label>
        <input id="swal-meta" type="number" class="swal2-input" style="width: 100%; box-sizing: border-box; margin-bottom: 10px;" value="150">

        <hr style="margin: 15px 0;">
        <h4 style="margin: 0 0 10px 0; color: #333;">Primeiro Item do Pedido</h4>

        <label style="font-weight: bold; font-size: 14px; color: #333;">Item:</label>
        <input id="swal-item" class="swal2-input" style="width: 100%; box-sizing: border-box; margin-bottom: 10px;" placeholder="Ex: Perfume">

        <label style="font-weight: bold; font-size: 14px; color: #333;">Valor do Item (R$):</label>
        <input id="swal-valor" type="number" class="swal2-input" style="width: 100%; box-sizing: border-box; margin-bottom: 10px;" placeholder="Ex: 50">

        <label style="font-weight: bold; font-size: 14px; color: #333;">Descrição:</label>
        <input id="swal-espec" class="swal2-input" style="width: 100%; box-sizing: border-box; margin-bottom: 10px;">

        <label style="font-weight: bold; font-size: 14px; color: #333;">Link da loja (Opcional):</label>
        <input id="swal-link" class="swal2-input" style="width: 100%; box-sizing: border-box;">
      </div>
    `,
    showCancelButton: true,
    confirmButtonText: 'Criar Pedido',
    preConfirm: () => {
      const nomeKit = document.getElementById('swal-nome-kit').value || 'Pedido de Presente';
      const meta = parseFloat(document.getElementById('swal-meta').value) || 150;
      
      const item = document.getElementById('swal-item').value;
      const valor = parseFloat(document.getElementById('swal-valor').value) || 0;
      const tamanhoEspecificacao = document.getElementById('swal-espec').value;
      let linkLoja = document.getElementById('swal-link').value;
      
      if (!item) {
        Swal.showValidationMessage('Adicione pelo menos 1 item para criar o pedido!');
        return false;
      }
      if (linkLoja && !linkLoja.startsWith('http')) linkLoja = 'https://' + linkLoja;
      
      return { 
        nomeFamiliar: props.usuarioAtual, 
        nomeKit, 
        meta,
        itens: [{ item, valor, tamanhoEspecificacao, linkLoja }] 
      };
    }
  });

  if (formValues) {
    try {
      await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(formValues) });
      Swal.fire('Pronto!', 'Pedido criado!', 'success');
      fetchPresentes();
    } catch (error) { Swal.fire('Erro', 'Falha ao conectar.', 'error'); }
  }
};

const editarKit = async (kit) => {
  const { value: formValues } = await Swal.fire({
    title: 'Editar Pedido Principal',
    html: `
      <div style="text-align: left;">
        <label style="font-weight: bold; font-size: 14px; color: #333;">Nome do Pedido (Ex: Kit Verão):</label>
        <input id="swal-edit-nome" class="swal2-input" style="width: 100%; box-sizing: border-box; margin-bottom: 10px;" value="${kit.nomeKit}">
        <label style="font-weight: bold; font-size: 14px; color: #333;">Meta de Valor (R$):</label>
        <input id="swal-edit-meta" type="number" class="swal2-input" style="width: 100%; box-sizing: border-box;" value="${kit.meta || 150}">
      </div>
    `,
    showCancelButton: true,
    confirmButtonText: 'Salvar',
    preConfirm: () => {
      const nomeKit = document.getElementById('swal-edit-nome').value || 'Pedido de Presente';
      const meta = parseFloat(document.getElementById('swal-edit-meta').value) || 150;
      return { nomeKit, meta };
    }
  });

  if (formValues) {
    try {
      await fetch(`${apiUrl}/${kit._id}`, { method: 'PUT', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(formValues) });
      Swal.fire('Atualizado!', 'Pedido modificado.', 'success');
      fetchPresentes();
    } catch (e) { Swal.fire('Erro', 'Falha ao conectar.', 'error'); }
  }
};

const adicionarSubItem = async (kit) => {
  const { value: formValues } = await Swal.fire({
    title: 'Adicionar Item',
    html: `
      <div style="text-align: left;">
        <label style="font-weight: bold; font-size: 14px; color: #333;">Item:</label>
        <input id="swal-sub-item" class="swal2-input" style="width: 100%; box-sizing: border-box; margin-bottom: 10px;">
        <label style="font-weight: bold; font-size: 14px; color: #333;">Valor (R$):</label>
        <input id="swal-sub-valor" type="number" class="swal2-input" style="width: 100%; box-sizing: border-box; margin-bottom: 10px;">
        <label style="font-weight: bold; font-size: 14px; color: #333;">Descrição (Opcional):</label>
        <input id="swal-sub-desc" class="swal2-input" style="width: 100%; box-sizing: border-box; margin-bottom: 10px;">
        <label style="font-weight: bold; font-size: 14px; color: #333;">Link (Opcional):</label>
        <input id="swal-sub-link" class="swal2-input" style="width: 100%; box-sizing: border-box;">
      </div>
    `,
    showCancelButton: true,
    confirmButtonText: 'Adicionar',
    preConfirm: () => {
      const item = document.getElementById('swal-sub-item').value;
      const valor = parseFloat(document.getElementById('swal-sub-valor').value) || 0;
      const tamanhoEspecificacao = document.getElementById('swal-sub-desc').value;
      let linkLoja = document.getElementById('swal-sub-link').value;
      
      if (!item) {
        Swal.showValidationMessage('O nome do item é obrigatório!');
        return false;
      }
      if (linkLoja && !linkLoja.startsWith('http')) linkLoja = 'https://' + linkLoja;
      return { item, valor, tamanhoEspecificacao, linkLoja };
    }
  });

  if (formValues) {
    try {
      const novosItens = [...(kit.itens || []), formValues];
      await fetch(`${apiUrl}/${kit._id}`, { method: 'PUT', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ itens: novosItens }) });
      fetchPresentes();
    } catch (e) { Swal.fire('Erro', 'Falha ao salvar.', 'error'); }
  }
};

const deletarSubItem = async (kit, idx) => {
  const confirm = await Swal.fire({ title: 'Apagar item?', icon: 'warning', showCancelButton: true, confirmButtonText: 'Sim' });
  if (confirm.isConfirmed) {
    try {
      const novosItens = kit.itens.filter((_, i) => i !== idx);
      await fetch(`${apiUrl}/${kit._id}`, { method: 'PUT', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ itens: novosItens }) });
      fetchPresentes();
    } catch (e) { Swal.fire('Erro', 'Falha ao apagar.', 'error'); }
  }
};

const deletarPresente = async (id) => {
  const confirm = await Swal.fire({ title: 'Apagar pedido inteiro?', text: "Isso removerá todos os itens deste kit.", icon: 'warning', showCancelButton: true, confirmButtonColor: '#d33', confirmButtonText: 'Sim, apagar!' });
  if (confirm.isConfirmed) {
    try { await fetch(`${apiUrl}/${id}`, { method: 'DELETE' }); fetchPresentes(); } 
    catch (error) { Swal.fire('Erro', 'Não foi possível apagar.', 'error'); }
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