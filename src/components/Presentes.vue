<template>
  <div class="card-natalino" style="padding: 20px; margin-top: 20px; text-align: center;">
    <h2 style="color: #e8f5e9; margin-top: 0;">🛍️ Vitrine de Presentes</h2>
    <p style="color: #e8f5e9; margin-bottom: 20px; font-size: 1.1rem;">Dicas do que a família quer ganhar!</p>

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
            <div v-for="kit in getPresentes(nome)" :key="kit._id" :style="kit.isKit !== false ? 'background: #fff; padding: 15px; border-radius: 8px; border: 1px solid #e5c09e; position: relative; box-shadow: 0 2px 5px rgba(0,0,0,0.05);' : 'background: #fff9f0; padding: 15px; border-radius: 8px; border: 1px solid #ccc; position: relative; box-shadow: 0 2px 5px rgba(0,0,0,0.05);'">
              
              <div v-if="usuarioAtual === nome" style="position: absolute; top: 10px; right: 10px; display: flex; gap: 8px; z-index: 2;">
                <button v-if="kit.isKit !== false" @click="editarKit(kit)" style="background: transparent; border: none; color: #2e7d32; cursor: pointer; font-size: 1rem;" title="Editar meta e nome do pedido">✏️</button>
                <button v-if="kit.isKit === false" @click="editarIndividual(kit)" style="background: transparent; border: none; color: #2e7d32; cursor: pointer; font-size: 1rem;" title="Editar item">✏️</button>
                <button @click="deletarPresente(kit._id)" style="background: transparent; border: none; color: #c62828; cursor: pointer; font-size: 1.1rem;" title="Remover pedido completo">✖</button>
              </div>

              <h4 v-if="kit.isKit !== false" style="margin: 0 0 10px 0; color: #333333; font-size: 1.2rem; padding-right: 50px;">{{ kit.nomeKit }}</h4>
              
              <!-- Total -->
              <div v-if="kit.isKit !== false" style="margin-bottom: 15px; border-bottom: 1px solid #ccc; padding-bottom: 10px;">
                <div style="font-size: 1.1rem; color: #333333; font-weight: bold;">
                  <span>Valor Total do Pedido:</span>
                  <span style="color: #2e7d32; margin-left: 10px;">R$ {{ calcularSomaKit(kit).toFixed(2).replace('.', ',') }}</span>
                </div>
              </div>

              <!-- Itens -->
              <div style="display: flex; flex-direction: column; gap: 10px;">
                <div v-for="(item, idx) in kit.itens" :key="idx" :style="kit.isKit !== false ? 'background: #fff9f0; padding: 10px; border-radius: 5px; border: 1px solid #ccc; position: relative;' : 'position: relative; padding-right: 30px;'">
                   <button v-if="usuarioAtual === nome && kit.isKit !== false" @click="deletarSubItem(kit, idx)" style="position: absolute; top: 5px; right: 5px; background: transparent; border: none; color: #c62828; cursor: pointer; font-size: 0.9rem;" title="Remover item">✖</button>
                   <p style="margin: 0; font-weight: bold; color: #333333;">{{ item.item }}</p>
                   <p v-if="item.valor" style="margin: 5px 0 0 0; font-size: 0.95rem; color: #2e7d32; font-weight: bold;">💰 R$ {{ item.valor.toFixed(2).replace('.', ',') }}</p>
                   <p v-if="item.tamanhoEspecificacao" style="margin: 5px 0 0 0; font-size: 0.85rem; color: #333333;">📝 {{ item.tamanhoEspecificacao }}</p>
                   <div v-if="item.linkLoja" style="margin-top: 8px; display: flex; flex-direction: column; gap: 5px;">
                     <a v-for="(link, i) in item.linkLoja.split('\\n')" :key="i" :href="link" target="_blank" style="display: inline-block; color: #8b0000; font-size: 0.85rem; text-decoration: none; font-weight: bold; border-bottom: 1px solid #8b0000; align-self: flex-start;">🛒 Ver na Loja {{ item.linkLoja.split('\\n').length > 1 ? i + 1 : '' }}</a>
                   </div>
                </div>
              </div>

              <button v-if="usuarioAtual === nome && kit.isKit !== false" @click="adicionarSubItem(kit)" style="margin-top: 15px; background: transparent; color: #2e7d32; border: 2px dashed #2e7d32; padding: 8px; border-radius: 5px; cursor: pointer; width: 100%; font-weight: bold;">➕ Adicionar item ao pacote</button>

            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- BOTOES FLUTUANTES (FAB) -->
  <teleport to="body">
    <div class="fab-container" style="display: flex; flex-direction: column; gap: 12px; align-items: flex-end;">
      <button @click="adicionarPresente(false)" class="fab-btn" style="background: #2196f3; color: white; box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);">
        <span style="font-size: 1.2rem;">🎁</span>
        <span class="fab-text">Pedido Individual</span>
      </button>
      <button @click="adicionarPresente(true)" class="fab-btn" style="background: #2e7d32; color: white; box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);">
        <span style="font-size: 1.2rem;">🛍️</span>
        <span class="fab-text">Criar um Kit</span>
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
const BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:3000';
const apiUrl = `${BASE_URL}/api/presentes`;

const getAuthHeaders = () => {
  const user = JSON.parse(localStorage.getItem('loggedInUser') || '{}');
  return { 'Content-Type': 'application/json', 'Authorization': `Bearer ${user.token || ''}` };
};

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

// Expor globalmente para o Swal poder acessar no oninput
window.formatarMoeda = function(input) {
  let valor = input.value.replace(/\D/g, "");
  if (!valor) { input.value = ""; return; }
  valor = (parseInt(valor, 10) / 100).toFixed(2) + "";
  valor = valor.replace(".", ",");
  valor = valor.replace(/(\d)(?=(\d{3})+(?!\d))/g, "$1.");
  input.value = "R$ " + valor;
};

const renderLinksInput = (containerId, initialLinks) => {
  const container = document.getElementById(containerId);
  if (!container) return;
  
  let linksArray = initialLinks ? initialLinks.split('\\n') : [''];
  if (linksArray.length === 0) linksArray = [''];
  
  const render = (arr) => {
    container.innerHTML = '';
    arr.forEach((link, index) => {
      const div = document.createElement('div');
      div.style.display = 'flex';
      div.style.alignItems = 'center';
      div.style.gap = '8px';
      div.style.marginBottom = '10px';
      
      const input = document.createElement('input');
      input.className = 'swal2-input swal-link-input';
      input.style.margin = '0';
      input.style.flex = '1';
      input.style.height = '42px';
      input.style.fontSize = '14px';
      input.placeholder = `Link ${index + 1}`;
      input.value = link;
      
      div.appendChild(input);
      
      if (index === arr.length - 1) {
        const btn = document.createElement('button');
        btn.type = 'button';
        btn.innerHTML = '+';
        btn.style.background = '#2e7d32';
        btn.style.color = 'white';
        btn.style.border = 'none';
        btn.style.borderRadius = '5px';
        btn.style.width = '42px';
        btn.style.height = '42px';
        btn.style.fontSize = '24px';
        btn.style.lineHeight = '1';
        btn.style.cursor = 'pointer';
        btn.style.display = 'flex';
        btn.style.alignItems = 'center';
        btn.style.justifyContent = 'center';
        btn.onclick = () => {
          const currentInputs = Array.from(container.querySelectorAll('.swal-link-input')).map(inp => inp.value);
          currentInputs.push('');
          render(currentInputs);
        };
        div.appendChild(btn);
      } else {
        const spacer = document.createElement('div');
        spacer.style.width = '42px';
        div.appendChild(spacer);
      }
      
      container.appendChild(div);
    });
  };
  render(linksArray);
};

const getLinksValues = (containerId) => {
  const container = document.getElementById(containerId);
  if (!container) return '';
  const inputs = Array.from(container.querySelectorAll('.swal-link-input'));
  return inputs.map(inp => inp.value.trim()).filter(l => l).map(l => l.startsWith('http') ? l : 'https://' + l).join('\\n');
};

const adicionarPresente = async (isKit = false) => {
  if (!props.usuarioAtual) {
    Swal.fire('Identifique-se!', 'Por favor, selecione quem é você no canto superior direito da página antes de criar um pedido.', 'warning');
    return;
  }

  let itensDoPedido = [];
  let nomeKitCache = '';
  let finalResult = null;
  
  while (true) {
    let itensHtml = '';
    if (itensDoPedido.length > 0) {
      itensHtml = '<div style="background: #eee; padding: 10px; border-radius: 5px; margin-bottom: 15px; font-size: 13px; text-align: left;">';
      itensHtml += '<strong>Itens já adicionados:</strong><ul style="margin: 5px 0 0 15px; padding: 0;">';
      itensDoPedido.forEach((it) => {
         itensHtml += `<li style="color: #333333;">${it.item} - R$ ${it.valor.toFixed(2).replace('.', ',')}</li>`;
      });
      itensHtml += '</ul></div>';
    }

    let titulo = isKit ? 'Criar Kit de Pedidos' : 'Adicionar Pedido Individual';
    
    let htmlForm = '<div style="text-align: left;">';
    
    if (isKit && itensDoPedido.length === 0) {
      htmlForm += `
        <label style="font-weight: bold; font-size: 14px; color: #333333;">Nome do Kit (Ex: Kit Praia):</label>
        <input id="swal-nome-kit" class="swal2-input" style="width: 100%; box-sizing: border-box; margin: 0 0 10px 0; max-width: 100%;" placeholder="Opcional" value="${nomeKitCache}">
        <hr style="margin: 15px 0;">
      `;
    } else if (isKit) {
       htmlForm += `<input type="hidden" id="swal-nome-kit" value="${nomeKitCache}">`;
    }
    
    htmlForm += itensHtml;

    if (isKit && itensDoPedido.length > 0) {
        htmlForm += `<h4 style="margin: 0 0 10px 0; color: #333333;">Adicionar mais um item ao kit</h4>`;
    }

    htmlForm += `
      <label style="font-weight: bold; font-size: 14px; color: #333333;">Item:</label>
      <input id="swal-item" class="swal2-input" style="width: 100%; box-sizing: border-box; margin: 0 0 10px 0; max-width: 100%;" placeholder="Ex: Perfume">

      <label style="font-weight: bold; font-size: 14px; color: #333333;">Valor do Item (R$):</label>
      <input id="swal-valor" type="text" oninput="window.formatarMoeda(this)" class="swal2-input" style="width: 100%; box-sizing: border-box; margin: 0 0 10px 0; max-width: 100%;" placeholder="R$ 0,00">

      <label style="font-weight: bold; font-size: 14px; color: #333333;">Descrição:</label>
      <input id="swal-espec" class="swal2-input" style="width: 100%; box-sizing: border-box; margin: 0 0 10px 0; max-width: 100%;">

      <label style="font-weight: bold; font-size: 14px; color: #333333;">Links da loja:</label>
      <div id="swal-links-container" style="margin-bottom: 15px;"></div>
    `;
    
    if (isKit) {
      htmlForm += `
        <button id="btn-add-mais" type="button" style="background: transparent; border: 2px dashed #2e7d32; color: #2e7d32; padding: 10px; width: 100%; border-radius: 5px; cursor: pointer; font-weight: bold;">➕ Adicionar mais um item a este kit</button>
      `;
    }
    
    htmlForm += '</div>';

    const res = await Swal.fire({
      title: titulo,
      html: htmlForm,
      showCancelButton: true,
      confirmButtonText: 'Salvar Pedido',
      cancelButtonText: 'Cancelar',
      didOpen: () => {
        renderLinksInput('swal-links-container', '');
        if (isKit) {
          const btnAdd = document.getElementById('btn-add-mais');
          if (btnAdd) {
            btnAdd.addEventListener('click', () => {
              window._addMaisClicked = true;
              Swal.clickConfirm();
            });
          }
        }
      },
      preConfirm: () => {
        const isAddMais = window._addMaisClicked;
        window._addMaisClicked = false;

        const item = document.getElementById('swal-item').value;
        const nomeKitEl = document.getElementById('swal-nome-kit');
        let nomeKit = nomeKitEl ? nomeKitEl.value : '';
        if (!nomeKit) {
          nomeKit = nomeKitCache || (itensDoPedido.length > 0 ? itensDoPedido[0].item : (item || 'Pedido de Presente'));
        }
        const valorInput = document.getElementById('swal-valor').value;
        const valor = parseFloat(valorInput.replace('R$ ', '').replace(/\./g, '').replace(',', '.')) || 0;
        const tamanhoEspecificacao = document.getElementById('swal-espec').value;
        let linkLoja = getLinksValues('swal-links-container');

        if (isKit && isAddMais) {
          if (!item) {
             Swal.showValidationMessage('Preencha o nome do item antes de adicionar!');
             return false;
          }
          itensDoPedido.push({ item, valor, tamanhoEspecificacao, linkLoja });
          return { addMais: true, nomeKitAtual: nomeKit };
        } else {
          let finalItens = [...itensDoPedido];
          if (item) {
            finalItens.push({ item, valor, tamanhoEspecificacao, linkLoja });
          }
          if (finalItens.length === 0) {
            Swal.showValidationMessage('Adicione pelo menos 1 item para criar o pedido!');
            return false;
          }
          return { addMais: false, nomeKit, finalItens };
        }
      }
    });

    if (res.isDismissed) return;
    if (res.isConfirmed) {
      if (res.value.addMais) {
        nomeKitCache = res.value.nomeKitAtual;
        continue;
      } else {
        finalResult = res.value;
        break;
      }
    }
  }

  if (finalResult) {
    try {
      const payload = {
        nomeFamiliar: props.usuarioAtual,
        nomeKit: finalResult.nomeKit,
        isKit: isKit,
        itens: finalResult.finalItens
      };
      await fetch(apiUrl, { method: 'POST', headers: getAuthHeaders(), body: JSON.stringify(payload) });
      Swal.fire('Pronto!', 'Pedido criado!', 'success');
      fetchPresentes();
    } catch (error) { Swal.fire('Erro', 'Falha ao conectar.', 'error'); }
  }
};

const editarKit = async (kit) => {
  const { value: formValues } = await Swal.fire({
    title: 'Editar Nome do Pedido',
    html: `
      <div style="text-align: left;">
        <label style="font-weight: bold; font-size: 14px; color: #333333;">Nome do Pedido (Ex: Kit Verão):</label>
        <input id="swal-edit-nome" class="swal2-input" style="width: 100%; box-sizing: border-box; margin: 0 0 10px 0; max-width: 100%;" value="${kit.nomeKit}">
      </div>
    `,
    showCancelButton: true,
    confirmButtonText: 'Salvar',
    preConfirm: () => {
      const nomeKit = document.getElementById('swal-edit-nome').value || 'Pedido de Presente';
      return { nomeKit };
    }
  });

  if (formValues) {
    try {
      await fetch(`${apiUrl}/${kit._id}`, { method: 'PUT', headers: getAuthHeaders(), body: JSON.stringify(formValues) });
      Swal.fire('Atualizado!', 'Pedido modificado.', 'success');
      fetchPresentes();
    } catch (e) { Swal.fire('Erro', 'Falha ao conectar.', 'error'); }
  }
};

const editarIndividual = async (kit) => {
  const item = kit.itens && kit.itens.length > 0 ? kit.itens[0] : {};
  const { value: formValues } = await Swal.fire({
    title: 'Editar Pedido Individual',
    html: `
      <div style="text-align: left;">
        <label style="font-weight: bold; font-size: 14px; color: #333333;">Item:</label>
        <input id="swal-edit-ind-item" class="swal2-input" style="width: 100%; box-sizing: border-box; margin: 0 0 10px 0; max-width: 100%;" value="${item.item || ''}">
        
        <label style="font-weight: bold; font-size: 14px; color: #333333;">Valor (R$):</label>
        <input id="swal-edit-ind-valor" type="text" oninput="window.formatarMoeda(this)" class="swal2-input" style="width: 100%; box-sizing: border-box; margin: 0 0 10px 0; max-width: 100%;" value="R$ ${(item.valor || 0).toFixed(2).replace('.', ',')}">
        
        <label style="font-weight: bold; font-size: 14px; color: #333333;">Descrição:</label>
        <input id="swal-edit-ind-desc" class="swal2-input" style="width: 100%; box-sizing: border-box; margin: 0 0 10px 0; max-width: 100%;" value="${item.tamanhoEspecificacao || ''}">
        
        <label style="font-weight: bold; font-size: 14px; color: #333333;">Links da loja:</label>
        <div id="swal-edit-ind-links-container" style="margin-bottom: 15px;"></div>
      </div>
    `,
    didOpen: () => {
      renderLinksInput('swal-edit-ind-links-container', item.linkLoja || '');
    },
    showCancelButton: true,
    confirmButtonText: 'Salvar',
    preConfirm: () => {
      const novoItem = document.getElementById('swal-edit-ind-item').value;
      const valorInput = document.getElementById('swal-edit-ind-valor').value;
      const valor = parseFloat(valorInput.replace('R$ ', '').replace(/\\./g, '').replace(',', '.')) || 0;
      const tamanhoEspecificacao = document.getElementById('swal-edit-ind-desc').value;
      let linkLoja = getLinksValues('swal-edit-ind-links-container');
      
      if (!novoItem) {
        Swal.showValidationMessage('O nome do item é obrigatório!');
        return false;
      }
      return { item: novoItem, valor, tamanhoEspecificacao, linkLoja };
    }
  });

  if (formValues) {
    try {
      await fetch(`${apiUrl}/${kit._id}`, { 
        method: 'PUT', 
        headers: getAuthHeaders(), 
        body: JSON.stringify({ itens: [formValues], nomeKit: formValues.item }) 
      });
      Swal.fire('Atualizado!', 'Item modificado.', 'success');
      fetchPresentes();
    } catch (e) { Swal.fire('Erro', 'Falha ao conectar.', 'error'); }
  }
};

const adicionarSubItem = async (kit) => {
  const { value: formValues } = await Swal.fire({
    title: 'Adicionar Item',
    html: `
      <div style="text-align: left;">
        <label style="font-weight: bold; font-size: 14px; color: #333333;">Item:</label>
        <input id="swal-sub-item" class="swal2-input" style="width: 100%; box-sizing: border-box; margin: 0 0 10px 0; max-width: 100%;">
        <label style="font-weight: bold; font-size: 14px; color: #333333;">Valor (R$):</label>
        <input id="swal-sub-valor" type="text" oninput="window.formatarMoeda(this)" class="swal2-input" style="width: 100%; box-sizing: border-box; margin: 0 0 10px 0; max-width: 100%;" placeholder="R$ 0,00">
        <label style="font-weight: bold; font-size: 14px; color: #333333;">Descrição:</label>
        <input id="swal-sub-desc" class="swal2-input" style="width: 100%; box-sizing: border-box; margin: 0 0 10px 0; max-width: 100%;">
        <label style="font-weight: bold; font-size: 14px; color: #333333;">Links da loja:</label>
        <div id="swal-sub-links-container" style="margin-bottom: 15px;"></div>
      </div>
    `,
    didOpen: () => {
      renderLinksInput('swal-sub-links-container', '');
    },
    showCancelButton: true,
    confirmButtonText: 'Adicionar',
    preConfirm: () => {
      const item = document.getElementById('swal-sub-item').value;
      const valorInput = document.getElementById('swal-sub-valor').value;
      const valor = parseFloat(valorInput.replace('R$ ', '').replace(/\./g, '').replace(',', '.')) || 0;
      const tamanhoEspecificacao = document.getElementById('swal-sub-desc').value;
      let linkLoja = getLinksValues('swal-sub-links-container');
      
      if (!item) {
        Swal.showValidationMessage('O nome do item é obrigatório!');
        return false;
      }
      return { item, valor, tamanhoEspecificacao, linkLoja };
    }
  });

  if (formValues) {
    try {
      const novosItens = [...(kit.itens || []), formValues];
      await fetch(`${apiUrl}/${kit._id}`, { method: 'PUT', headers: getAuthHeaders(), body: JSON.stringify({ itens: novosItens }) });
      fetchPresentes();
    } catch (e) { Swal.fire('Erro', 'Falha ao salvar.', 'error'); }
  }
};

const deletarSubItem = async (kit, idx) => {
  const confirm = await Swal.fire({ title: 'Apagar item?', icon: 'warning', showCancelButton: true, confirmButtonText: 'Sim' });
  if (confirm.isConfirmed) {
    try {
      const novosItens = kit.itens.filter((_, i) => i !== idx);
      await fetch(`${apiUrl}/${kit._id}`, { method: 'PUT', headers: getAuthHeaders(), body: JSON.stringify({ itens: novosItens }) });
      fetchPresentes();
    } catch (e) { Swal.fire('Erro', 'Falha ao apagar.', 'error'); }
  }
};

const deletarPresente = async (id) => {
  const confirm = await Swal.fire({ title: 'Apagar pedido inteiro?', text: "Isso removerá todos os itens deste kit.", icon: 'warning', showCancelButton: true, confirmButtonColor: '#d33', confirmButtonText: 'Sim, apagar!' });
  if (confirm.isConfirmed) {
    try { await fetch(`${apiUrl}/${id}`, { method: 'DELETE', headers: getAuthHeaders() }); fetchPresentes(); } 
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