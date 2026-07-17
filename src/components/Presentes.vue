<template>
  <div style="background: #e3f2fd; padding: 20px; border-radius: 8px; border: 2px solid #2196f3; margin-top: 20px; text-align: center;">
    <h2 style="color: #0d47a1; margin-top: 0;">🛍️ Vitrine de Presentes</h2>
    <p style="color: #1565c0; margin-bottom: 20px; font-size: 1.1rem;">Dicas do que a família quer ganhar!</p>

    <!-- BARRA DE FILTRO COM OS NOMES DA FAMÍLIA -->
    <div style="display: flex; gap: 8px; overflow-x: auto; padding-bottom: 12px; margin-bottom: 20px; scrollbar-width: thin; border-bottom: 2px solid #bbdefb;">
      <button @click="filtroAtivo = 'Todos'" :style="filtroAtivo === 'Todos' ? btnAtivo : btnInativo">🌟 Todos</button>
      <button v-for="nome in nomesFamilia" :key="nome" @click="filtroAtivo = nome" :style="filtroAtivo === nome ? btnAtivo : btnInativo">
        {{ nome }}
      </button>
    </div>

    <!-- GRID DOS PRESENTES FILTRADOS -->
    <div style="display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 15px; text-align: left;">
      <div v-for="presente in presentesFiltrados" :key="presente._id" style="background: white; padding: 15px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); position: relative; border-left: 5px solid #2196f3;">
        <button @click="deletarPresente(presente._id)" style="position: absolute; top: 5px; right: 5px; background: transparent; border: none; color: #f44336; cursor: pointer; font-size: 1.2rem;" title="Remover pedido">✖</button>
        <h3 style="color: #d32f2f; margin-top: 0; margin-bottom: 5px;">{{ presente.nomeFamiliar }}</h3>
        <p style="margin: 0; font-weight: bold; color: #333; font-size: 1.1rem;">{{ presente.item }}</p>
        <p style="margin: 5px 0; font-size: 0.9rem; color: #666;">📝 {{ presente.tamanhoEspecificacao || 'Sem especificações' }}</p>
        <a v-if="presente.linkLoja" :href="presente.linkLoja" target="_blank" style="display: inline-block; margin-top: 10px; background: #2196f3; color: white; padding: 8px 12px; text-decoration: none; border-radius: 5px; font-size: 0.9rem; font-weight: bold;">🛒 Ver na Loja</a>
      </div>
    </div>

    <div v-if="presentesFiltrados.length === 0" style="color: #999; margin: 20px 0;">
      Ainda não tem pedidos para essa seleção.
    </div>

    <button @click="adicionarPresente" style="margin-top: 20px; background: #2196f3; color: white; border: none; padding: 12px 20px; border-radius: 5px; font-weight: bold; font-size: 16px; cursor: pointer; width: 100%;">
      ➕ Adicionar meu pedido
    </button>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import Swal from 'sweetalert2';

const presentes = ref([]);
const apiUrl = 'https://natal-bl3x.onrender.com/api/presentes';

// LISTA FIXA DA FAMÍLIA
const nomesFamilia = ['Danilo', 'Fernanda', 'Guilherme', 'Daniel', 'Ana Lúcia', 'Liviane', 'Patrícia', 'Maria', 'Simão'];

// Controle das Abas Individuais
const filtroAtivo = ref('Todos');
const btnAtivo = 'background: #1976d2; color: white; border: none; padding: 8px 16px; border-radius: 20px; font-weight: bold; cursor: pointer; white-space: nowrap;';
const btnInativo = 'background: transparent; color: #1976d2; border: 1px solid #1976d2; padding: 8px 16px; border-radius: 20px; font-weight: bold; cursor: pointer; white-space: nowrap;';

const presentesFiltrados = computed(() => {
  if (filtroAtivo.value === 'Todos') return presentes.value;
  return presentes.value.filter(p => p.nomeFamiliar === filtroAtivo.value);
});

const fetchPresentes = async () => {
  try {
    const res = await fetch(apiUrl);
    presentes.value = await res.json();
  } catch (error) { console.error(error); }
};
onMounted(() => fetchPresentes());

// FORMULÁRIO ARRUMADO COM LISTA DE NOMES E LABELS PARA NÃO CORTAR TEXTO
const adicionarPresente = async () => {
  const { value: formValues } = await Swal.fire({
    title: 'Sua lista de desejos',
    html: `
      <div style="text-align: left;">
        <label style="font-weight: bold; font-size: 14px; color: #333;">Seu Nome:</label>
        <select id="swal-nome" class="swal2-select" style="width: 100%; max-width: 100%; margin: 5px 0 15px 0;">
          <option value="" disabled selected>Escolha seu nome...</option>
          ${nomesFamilia.map(n => `<option value="${n}">${n}</option>`).join('')}
        </select>

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
      const nomeFamiliar = document.getElementById('swal-nome').value;
      const item = document.getElementById('swal-item').value;
      const tamanhoEspecificacao = document.getElementById('swal-espec').value;
      let linkLoja = document.getElementById('swal-link').value;
      
      if (!nomeFamiliar || !item) {
        Swal.showValidationMessage('Seu nome e o item são obrigatórios!');
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
      filtroAtivo.value = formValues.nomeFamiliar; // Já abre na aba da pessoa!
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
</script>