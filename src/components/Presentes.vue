<template>
  <div style="background: #e3f2fd; padding: 20px; border-radius: 8px; border: 2px solid #2196f3; margin-top: 20px; text-align: center;">
    <img src="/img/presentes.jpg" alt="Presentes" style="width: 100%; border-radius: 10px; margin-bottom: 15px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); object-fit: cover; max-height: 200px;">
    
    <h2 style="color: #0d47a1; margin-top: 0;">🛍️ Vitrine de Presentes</h2>
    <p style="color: #1565c0; margin-bottom: 20px; font-size: 1.1rem;">Dicas do que a família quer ganhar!</p>

    <!-- GRID DE CARDS POR PESSOA -->
    <div style="display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 20px; text-align: left;">
      <div v-for="nome in nomesFamilia" :key="nome" style="background: white; border-radius: 12px; overflow: hidden; box-shadow: 0 4px 10px rgba(0,0,0,0.1); border: 1px solid #bbdefb; display: flex; flex-direction: column;">
        
        <!-- HEADER DO CARD (NOME) -->
        <div style="background: #2196f3; padding: 15px; color: white; display: flex; justify-content: center; align-items: center;">
          <h3 style="margin: 0; font-size: 1.4rem;">{{ nome }}</h3>
        </div>

        <!-- LISTA DE PRESENTES DESSA PESSOA -->
        <div style="padding: 15px; flex-grow: 1; background: #fafafa;">
          <div v-if="getPresentes(nome).length === 0" style="color: #999; font-size: 0.9rem; font-style: italic; text-align: center; margin-top: 10px;">
            Ainda não pediu nada.
          </div>
          <ul style="list-style: none; padding: 0; margin: 0;">
            <li v-for="presente in getPresentes(nome)" :key="presente._id" style="background: white; padding: 12px; border-radius: 8px; border: 1px solid #e0e0e0; margin-bottom: 10px; position: relative;">
              <div style="position: absolute; top: 5px; right: 5px; display: flex; gap: 8px;">
                <button @click="editarPresente(presente)" style="background: transparent; border: none; color: #1976d2; cursor: pointer; font-size: 1rem;" title="Editar pedido">✏️</button>
                <button @click="deletarPresente(presente._id)" style="background: transparent; border: none; color: #f44336; cursor: pointer; font-size: 1.1rem;" title="Remover pedido">✖</button>
              </div>
              <p style="margin: 0; font-weight: bold; color: #333; padding-right: 45px;">{{ presente.item }}</p>
              <p v-if="presente.tamanhoEspecificacao" style="margin: 5px 0 0 0; font-size: 0.85rem; color: #666;">📝 {{ presente.tamanhoEspecificacao }}</p>
              <a v-if="presente.linkLoja" :href="presente.linkLoja" target="_blank" style="display: inline-block; margin-top: 8px; color: #1976d2; font-size: 0.85rem; text-decoration: none; font-weight: bold; border-bottom: 1px solid #1976d2;">🛒 Ver na Loja</a>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <button @click="adicionarPresente" style="margin-top: 25px; background: #2196f3; color: white; border: none; padding: 15px 20px; border-radius: 8px; font-weight: bold; font-size: 16px; cursor: pointer; width: 100%; box-shadow: 0 4px 6px rgba(33, 150, 243, 0.3);">
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