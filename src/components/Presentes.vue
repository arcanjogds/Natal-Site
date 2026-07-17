<template>
  <div style="background: #e3f2fd; padding: 20px; border-radius: 8px; border: 2px solid #2196f3; margin-top: 20px; text-align: center;">
    <h2 style="color: #0d47a1; margin-top: 0;">🛍️ Vitrine de Presentes</h2>
    <p style="color: #1565c0; margin-bottom: 20px; font-size: 1.1rem;">Dicas do que a família quer ganhar!</p>

    <!-- Grid de Presentes -->
    <div style="display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 15px; text-align: left;">
      
      <div v-for="presente in presentes" :key="presente._id" style="background: white; padding: 15px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); position: relative; border-left: 5px solid #2196f3;">
        
        <!-- Botão de deletar (pequeno no canto) -->
        <button @click="deletarPresente(presente._id)" style="position: absolute; top: 5px; right: 5px; background: transparent; border: none; color: #f44336; cursor: pointer; font-size: 1.2rem;" title="Remover pedido">
          ✖
        </button>
        
        <h3 style="color: #d32f2f; margin-top: 0; margin-bottom: 5px;">{{ presente.nomeFamiliar }}</h3>
        <p style="margin: 0; font-weight: bold; color: #333; font-size: 1.1rem;">{{ presente.item }}</p>
        <p style="margin: 5px 0; font-size: 0.9rem; color: #666;">📝 {{ presente.tamanhoEspecificacao || 'Sem especificações' }}</p>
        
        <a v-if="presente.linkLoja" :href="presente.linkLoja" target="_blank" style="display: inline-block; margin-top: 10px; background: #2196f3; color: white; padding: 8px 12px; text-decoration: none; border-radius: 5px; font-size: 0.9rem; font-weight: bold;">
          🛒 Ver na Loja
        </a>
      </div>
      
    </div>

    <!-- Mensagem se estiver vazio -->
    <div v-if="presentes.length === 0" style="color: #999; margin: 20px 0;">
      Ninguém adicionou dicas de presentes ainda. Seja o primeiro!
    </div>

    <!-- Botão para adicionar -->
    <button @click="adicionarPresente" style="margin-top: 20px; background: #2196f3; color: white; border: none; padding: 12px 20px; border-radius: 5px; font-weight: bold; font-size: 16px; cursor: pointer; width: 100%;">
      ➕ Adicionar meu pedido
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import Swal from 'sweetalert2';

// Estado reativo e URL da API
const presentes = ref([]);
const apiUrl = 'https://natal-bl3x.onrender.com/api/presentes';

// Busca os dados no backend
const fetchPresentes = async () => {
  try {
    const res = await fetch(apiUrl);
    presentes.value = await res.json();
  } catch (error) {
    console.error("Erro ao buscar presentes:", error);
  }
};

onMounted(() => {
  fetchPresentes();
});

// Formulário SweetAlert para cadastrar presente
const adicionarPresente = async () => {
  const { value: formValues } = await Swal.fire({
    title: 'Sua lista de desejos',
    html:
      '<input id="swal-nome" class="swal2-input" placeholder="Seu Nome">' +
      '<input id="swal-item" class="swal2-input" placeholder="O que você quer? (Ex: Jogo na Steam, Perfume)">' +
      '<input id="swal-espec" class="swal2-input" placeholder="Tamanho/Detalhe (Ex: 220v, Tam 40)">' +
      '<input id="swal-link" class="swal2-input" placeholder="Link da loja (Opcional mas ajuda!)">',
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
      
      // Validação simples
      if (!nomeFamiliar || !item) {
        Swal.showValidationMessage('Seu nome e o item são obrigatórios!');
        return false;
      }

      // Garante que o link tem http:// ou https:// para funcionar direito
      if (linkLoja && !linkLoja.startsWith('http')) {
        linkLoja = 'https://' + linkLoja;
      }

      return { nomeFamiliar, item, tamanhoEspecificacao, linkLoja };
    }
  });

  if (formValues) {
    try {
      await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formValues)
      });
      Swal.fire('Pronto!', 'Seu pedido já está na vitrine!', 'success');
      fetchPresentes(); // Atualiza a tela
    } catch (error) {
      Swal.fire('Erro', 'Falha ao conectar com o banco de dados.', 'error');
    }
  }
};

// Função para apagar um item se a pessoa desistir
const deletarPresente = async (id) => {
  const confirm = await Swal.fire({
    title: 'Apagar pedido?',
    text: "Isso vai remover este item da vitrine.",
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#d33',
    cancelButtonColor: '#aaa',
    confirmButtonText: 'Sim, apagar!'
  });

  if (confirm.isConfirmed) {
    try {
      await fetch(`${apiUrl}/${id}`, { method: 'DELETE' });
      fetchPresentes(); // Atualiza a tela após deletar
    } catch (error) {
      Swal.fire('Erro', 'Não foi possível apagar o item.', 'error');
    }
  }
};
</script>