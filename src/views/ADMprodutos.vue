<template>
  <div class="tudo">
    <div v-if="mostraFormulario" class="criacao-form-wrapper">
      <div class="criacao-form">
        <h2>{{ editando ? 'Editar Produto' : 'Criar Produto' }}</h2>
        <form @submit.prevent="editando ? atualizarProduto() : criarProduto()">
          <div>
            <label>Nome:</label>
            <input v-model="nomeForm" required />
          </div>
          <div>
            <label>Descrição:</label>
            <textarea v-model="descricaoForm" required></textarea>
          </div>
          <div class="linha-dupla">
            <div class="campo-metade">
              <label>Preço:</label>
              <input type="number" v-model.number="precoForm" min="0" step="0.01" required />
            </div>
            <div class="campo-metade">
              <label>Estoque:</label>
              <input type="number" v-model.number="estoqueForm" min="0" required />
            </div>
          </div>
          <div>
            <label>Categoria:</label>
            <select v-model="categoriaIdForm" required>
              <option value="" disabled>Selecione</option>
              <option v-for="cat in categorias" :key="cat.id" :value="cat.id">{{ cat.name }}</option>
            </select>
          </div>
          <div>
            <label>Imagem:</label>
            <input type="file" @change="onFileChange" accept="image/*" />
          </div>
          <button type="submit">{{ editando ? 'Salvar' : 'Criar Produto' }}</button>
          <button type="button" @click="editando ? cancelarEdicao() : fecharFormulario()">Cancelar</button>
        </form>
        <p v-if="editando ? mensagemEdicao : mensagem">{{ editando ? mensagemEdicao : mensagem }}</p>
      </div>
    </div>
    <div class="produtos" >
    <div class="produtos-header">
      <h3>Produtos</h3>
      <div class="filtro-categorias">
        <div class="filtro-estoque">
          <label for="filtroEstoque" style="margin-right: 6px; font-size: 1rem;">Estoque:</label>
          <select id="filtroEstoque" v-model="estoqueSelecionado">
            <option value="">Indefinido</option>
            <option value="0">0</option>
            <option value="10-30">10-30</option>
            <option value="30-50">30-50</option>
            <option value="50-100">50-100</option>
            <option value="100+">100 ou mais</option>
          </select>
        </div>
        <div class="filtro-categoria">
          <label for="filtroCategoria" style="margin-right: 6px; font-size: 1rem;">Filtrar por categoria:</label>
          <select id="filtroCategoria" v-model="categoriaSelecionada">
            <option value="">Todas</option>
            <option v-for="cat in categorias" :key="cat.id" :value="cat.id">{{ cat.name }}</option>
          </select>
        </div>
        <button class="novo-produto-btn" @click="abrirCriacao">Novo produto</button>
      </div>
    </div>
    <div v-if="carregandoProdutos">Carregando produtos...</div>
    <div v-else-if="erroProdutos">{{ erroProdutos }}</div>
    <div v-else>
      <div v-if="produtos.length === 0">Nenhum produto cadastrado ainda.</div>
      <ul v-else class="lista">
        <li v-for="produto in produtosFiltrados" :key="produto.id" class="produto">
          <div class="nome-preco-imagem">
            <img v-if="produto.image_path" :src="produto.image_path" alt="Imagem do produto" class="produto-imagem" />
            <h4>{{ produto.name }}</h4>
            <p>R$ {{ produto.price }}</p>
          </div>
          <div class="BTli">
            <button @click="editarProduto(produto)">Editar</button>
            <button class="excluir-btn" @click="abrirModalExclusao(produto.id)">Excluir</button>
          </div>
          <span style="font-size:12px;color:#555;">Estoque: {{ produto.stock }}</span>
        </li>
      </ul>
    </div>
    </div>
  </div>
  
  <!-- Modal de Confirmação -->
  <div v-if="mostrarModalConfirmacao" class="modal-overlay">
      <div class="modal-confirmacao">
          <h3>Confirmar Exclusão</h3>
          <p>Tem certeza que deseja excluir este produto?</p>
          <div class="modal-botoes">
              <button @click="confirmarExclusao" class="btn-confirmar">Confirmar</button>
              <button @click="fecharModalConfirmacao" class="btn-cancelar">Cancelar</button>
          </div>
      </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'
import { useToast } from 'vue-toastification'
import api from '../services/api'

const imagem = ref(null)
const categorias = ref([])
const mensagem = ref('')
const toast = useToast()
const mostraFormulario = ref(false)
const mostrarModalConfirmacao = ref(false)
const produtoParaExcluir = ref(null)

// Edição
const editando = ref(false)
const editId = ref(null)
const editNome = ref('')
const editDescricao = ref('')
const editPreco = ref(0)
const editEstoque = ref(0)
const editCategoriaId = ref('')
const editImagem = ref(null)
const mensagemEdicao = ref('')

const nomeForm = ref('')
const descricaoForm = ref('')
const precoForm = ref(0)
const estoqueForm = ref(0)
const categoriaIdForm = ref('')
const imagemForm = ref(null)

const categoriaSelecionada = ref('')
const estoqueSelecionado = ref('')

const produtosFiltrados = computed(() => {
  let produtosFiltrados = produtos.value

  // Filtro por categoria
  if (categoriaSelecionada.value) {
    produtosFiltrados = produtosFiltrados.filter(produto => 
      String(produto.category_id) === String(categoriaSelecionada.value)
    )
  }

  // Filtro por estoque
  if (estoqueSelecionado.value) {
    produtosFiltrados = produtosFiltrados.filter(produto => {
      const estoque = produto.stock
      switch (estoqueSelecionado.value) {
        case '0':
          return estoque === 0
        case '10-30':
          return estoque >= 10 && estoque <= 30
        case '30-50':
          return estoque >= 30 && estoque <= 50
        case '50-100':
          return estoque >= 50 && estoque <= 100
        case '100+':
          return estoque >= 100
        default:
          return true
      }
    })
  }

  return produtosFiltrados
})

const produtos = ref([])
const carregandoProdutos = ref(true)
const erroProdutos = ref('')

watch(editando, (novo) => {
  if (novo) {
    nomeForm.value = editNome.value
    descricaoForm.value = editDescricao.value
    precoForm.value = editPreco.value
    estoqueForm.value = editEstoque.value
    categoriaIdForm.value = editCategoriaId.value
    imagemForm.value = null
  } else {
    nomeForm.value = ''
    descricaoForm.value = ''
    precoForm.value = 0
    estoqueForm.value = 0
    categoriaIdForm.value = ''
    imagemForm.value = null
  }
})

onMounted(async () => {
  try {
    const { data } = await api.get('/categories/user/228')
    categorias.value = data
  } catch (e) {
    mensagem.value = 'Erro ao carregar categorias.'
  }
  await carregarProdutos()
})

function onFileChange(e) {
  imagem.value = e.target.files[0]
  imagemForm.value = e.target.files[0]
}

function abrirCriacao() {
  editando.value = false
  mostraFormulario.value = true
  nomeForm.value = ''
  descricaoForm.value = ''
  precoForm.value = 0
  estoqueForm.value = 0
  categoriaIdForm.value = ''
  imagemForm.value = null
  mensagem.value = ''
}

function fecharFormulario() {
  mostraFormulario.value = false
  editando.value = false
  nomeForm.value = ''
  descricaoForm.value = ''
  precoForm.value = 0
  estoqueForm.value = 0
  categoriaIdForm.value = ''
  imagemForm.value = null
  mensagem.value = ''
  mensagemEdicao.value = ''
}

async function criarProduto() {
  mensagem.value = ''
  try {
    const formData = new FormData()
    formData.append('name', nomeForm.value)
    formData.append('description', descricaoForm.value)
    formData.append('price', precoForm.value)
    formData.append('stock', estoqueForm.value)
    formData.append('category_id', categoriaIdForm.value)
    if (imagemForm.value) formData.append('image', imagemForm.value)
    await api.post('/products/', formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    })
    toast.success('Produto criado com sucesso!')
    await carregarProdutos()
    fecharFormulario()
  } catch (e) {
    toast.error('Erro ao criar produto.')
  }
}

function editarProduto(produto) {
  editando.value = true
  mostraFormulario.value = false
  editId.value = produto.id
  editNome.value = produto.name
  editDescricao.value = produto.description
  editPreco.value = produto.price
  editEstoque.value = produto.stock
  editCategoriaId.value = produto.category_id
  editImagem.value = null
  mensagemEdicao.value = ''
  nomeForm.value = produto.name
  descricaoForm.value = produto.description
  precoForm.value = produto.price
  estoqueForm.value = produto.stock
  categoriaIdForm.value = produto.category_id
  imagemForm.value = null
  mostraFormulario.value = true
}
function cancelarEdicao() {
  editando.value = false
  editId.value = null
  editNome.value = ''
  editDescricao.value = ''
  editPreco.value = 0
  editEstoque.value = 0
  editCategoriaId.value = ''
  editImagem.value = null
  mensagemEdicao.value = ''
  mostraFormulario.value = false
  nomeForm.value = ''
  descricaoForm.value = ''
  precoForm.value = 0
  estoqueForm.value = 0
  categoriaIdForm.value = ''
  imagemForm.value = null
}

async function atualizarProduto() {
  mensagemEdicao.value = ''
  try {
    await api.put(`/products/${editId.value}`, {
      name: nomeForm.value,
      description: descricaoForm.value,
      price: precoForm.value,
      category_id: categoriaIdForm.value
    })
    await api.put(`/products/${editId.value}/stock`, {
      stock: Number(estoqueForm.value)
    })
    if (imagemForm.value) {
      const formData = new FormData()
      formData.append('image', imagemForm.value)
      await api.put(`/products/${editId.value}/image`, formData, {
        headers: { 'Content-Type': 'multipart/form-data' }
      })
    }
    toast.success('Produto atualizado com sucesso!')
    await carregarProdutos()
    cancelarEdicao()
  } catch (e) {
    toast.error('Erro ao atualizar produto.')
  }
}

function abrirModalExclusao(id) {
  produtoParaExcluir.value = id
  mostrarModalConfirmacao.value = true
}

async function confirmarExclusao() {
  try {
    await api.delete(`/products/${produtoParaExcluir.value}`)
    toast.success('Produto excluído com sucesso!')
    await carregarProdutos()
    fecharModalConfirmacao()
  } catch (e) {
    toast.error('Erro ao excluir produto.')
    fecharModalConfirmacao()
  }
}

function fecharModalConfirmacao() {
  mostrarModalConfirmacao.value = false
  produtoParaExcluir.value = null
}

async function carregarProdutos() {
  carregandoProdutos.value = true
  erroProdutos.value = ''
  try {
    const { data } = await api.get('/products/user/228')
    produtos.value = data.map(produto => ({
      ...produto,
      image_path: produto.image_path 
        ? `http://35.196.79.227:8000${produto.image_path}` 
        : '/placeholder-image.jpg'
    }))
  } catch (e) {
    erroProdutos.value = 'Erro ao carregar produtos.'
  } finally {
    carregandoProdutos.value = false
  }
}
</script>

<style scoped>

.tudo {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
    background-color: #ffffff;
    padding: 50px 0px 0px 70px;
}

@media (max-width: 768px) {
    .tudo {
        padding: 20px 15px 0px 15px;
    }
}

@media (max-width: 480px) {
    .tudo {
        padding: 15px 10px 0px 10px;
    }
}

.produtos {
  width: 95%;
  height: 30%;
}

@media (max-width: 768px) {
  .produtos {
    width: 100%;
    height: auto;
  }
}

.novo-produto-btn {
  padding: 10px 20px;
  font-size: 1.2rem;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  align-self: flex-start;
}

@media (max-width: 768px) {
  .novo-produto-btn {
    padding: 8px 16px;
    font-size: 1rem;
  }
}

@media (max-width: 480px) {
  .novo-produto-btn {
    padding: 6px 12px;
    font-size: 0.9rem;
  }
}

.novo-produto-btn:hover {
  background-color: #45a049;
}

.criacao-form-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1000;
  overflow: auto;
}

.criacao-form {
  background-color: #fff;
  padding: 32px 24px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
  width: 98vw;
  max-width: 540px;
  max-height: 90vh;
  overflow-y: auto;
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

@media (max-width: 768px) {
  .criacao-form {
    width: 95%;
    padding: 20px;
    margin: 10px;
  }
}

@media (max-width: 480px) {
  .criacao-form {
    width: 98%;
    padding: 15px;
    margin: 5px;
  }
}

.criacao-form h2 {
  font-size: 2.5rem;
  font-family: helvetica;
  margin-bottom: 20px;
}

@media (max-width: 768px) {
  .criacao-form h2 {
    font-size: 2rem;
    margin-bottom: 15px;
  }
}

@media (max-width: 480px) {
  .criacao-form h2 {
    font-size: 1.5rem;
    margin-bottom: 10px;
  }
}

.criacao-form label {
  display: block;
  margin-bottom: 10px;
  font-size: 1.1rem;
  font-weight: bold;
}

@media (max-width: 768px) {
  .criacao-form label {
    font-size: 1rem;
    margin-bottom: 8px;
  }
}

@media (max-width: 480px) {
  .criacao-form label {
    font-size: 0.9rem;
    margin-bottom: 6px;
  }
}

.criacao-form input[type="text"],
.criacao-form textarea,
.criacao-form input[type="file"],
.criacao-form input[type="number"],
.criacao-form select {
  width: 100%;
  padding: 10px;
  margin-bottom: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 1rem;
}

@media (max-width: 768px) {
  .criacao-form input[type="text"],
  .criacao-form textarea,
  .criacao-form input[type="file"],
  .criacao-form input[type="number"],
  .criacao-form select {
    padding: 8px;
    margin-bottom: 15px;
    font-size: 0.9rem;
  }
}

@media (max-width: 480px) {
  .criacao-form input[type="text"],
  .criacao-form textarea,
  .criacao-form input[type="file"],
  .criacao-form input[type="number"],
  .criacao-form select {
    padding: 6px;
    margin-bottom: 10px;
    font-size: 0.8rem;
  }
}

.criacao-form button {
  background-color: #4CAF50;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1.1rem;
  margin-right: 10px;
}

@media (max-width: 768px) {
  .criacao-form button {
    padding: 8px 16px;
    font-size: 1rem;
  }
}

@media (max-width: 480px) {
  .criacao-form button {
    padding: 6px 12px;
    font-size: 0.9rem;
    margin-bottom: 5px;
  }
}

.criacao-form button:last-child {
  background-color: #f44336;
}

.criacao-form button:last-child:hover {
  background-color: #da190b;
}

.criacao-form form {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.criacao-form form button {
  margin-right: 10px;
  margin-bottom: 0;
}

.produtos-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 25px;
  border-bottom: 1px solid rgb(167, 167, 167);
}

@media (max-width: 768px) {
  .produtos-header {
    flex-direction: column;
    gap: 15px;
    align-items: flex-start;
  }
}

@media (max-width: 480px) {
  .produtos-header {
    gap: 10px;
  }
}

.produtos-header h3 {
  font-size: 2.5rem;
  font-family: helvetica;
}

@media (max-width: 768px) {
  .produtos-header h3 {
    font-size: 2rem;
  }
}

@media (max-width: 480px) {
  .produtos-header h3 {
    font-size: 1.5rem;
  }
}

ul {
  margin-top: 1rem;
  padding-left: 0;
  list-style: none;
  max-height: 70vh;
  overflow-y: auto;
  scrollbar-color: rgb(100, 100, 100) rgba(241, 241, 241, 0.527);
}

li {
  background-color: #f0f0f0;
  padding: 15px;
  margin-bottom: 10px;
  border-radius: 7px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 1.1rem;
  border: 1px solid #a9b5b6;
}

.BTli {
  display: flex;
  gap: 1vw;
}

.excluir-btn {
  background-color: #dc3545 !important;
  color: white !important;
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.9rem;
  margin-left: 10px;
}

.excluir-btn:hover {
  background-color: #b71c1c !important;
}

.BTli button {
  background-color: #6c757d;
  color: white;
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.9rem;
  margin-left: 10px;
}

.BTli button:hover {
  background-color: #5a6268;
}

.linha-dupla {
  display: flex;
  gap: 16px;
}

@media (max-width: 768px) {
  .linha-dupla {
    flex-direction: column;
    gap: 12px;
  }
}

@media (max-width: 480px) {
  .linha-dupla {
    gap: 8px;
  }
}

.campo-metade {
  flex: 1;
  display: flex;
  flex-direction: column;
}

@media (max-width: 768px) {
  .campo-metade {
    flex: none;
  }
}

.filtro-categorias {
  display: flex;
  align-items: center;
  gap: 12px;
}

.filtro-estoque,
.filtro-categoria {
  display: flex;
  align-items: center;
  gap: 6px;
}

@media (max-width: 768px) {
  .filtro-categorias {
    flex-direction: column;
    align-items: stretch;
    gap: 6px;
  }
  
  .filtro-estoque,
  .filtro-categoria {
    flex-direction: column;
    align-items: stretch;
    gap: 4px;
  }
}

@media (max-width: 480px) {
  .filtro-categorias {
    gap: 4px;
  }
  
  .filtro-estoque,
  .filtro-categoria {
    gap: 2px;
  }
}

.filtro-categorias select {
  padding: 6px 10px;
  border-radius: 5px;
  border: 1px solid #bdbdbd;
  font-size: 1rem;
}

@media (max-width: 768px) {
  .filtro-categorias select {
    padding: 4px 8px;
    font-size: 0.9rem;
  }
}

@media (max-width: 480px) {
  .filtro-categorias select {
    padding: 3px 6px;
    font-size: 0.8rem;
  }
}

.nome-preco-imagem {
  display: flex;
  flex-direction: column;
  align-items: center;
}

@media (max-width: 768px) {
  .nome-preco-imagem {
    gap: 8px;
  }
}

.nome-preco-imagem p {
  font-size: 22px;
  color: rgb(49, 49, 49);
  font-weight: bold;
}

@media (max-width: 768px) {
  .nome-preco-imagem p {
    font-size: 18px;
  }
}

@media (max-width: 480px) {
  .nome-preco-imagem p {
    font-size: 16px;
  }
}

.nome-preco-imagem img {
  margin-top: 10px;
  height: 225px;
  width: 160px;
  border: 0.1px solid rgb(212, 212, 212);
  filter: contrast(100%);
}

@media (max-width: 768px) {
  .nome-preco-imagem img {
    height: 180px;
    width: 130px;
  }
}

@media (max-width: 480px) {
  .nome-preco-imagem img {
    height: 150px;
    width: 110px;
  }
}

.produto {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  text-align: center;
  width: 230px;
  height: 92%;
  margin-top: 3vh;
  padding-left: 10px;
  padding-right: 10px;
}

@media (max-width: 768px) {
  .produto {
    width: 200px;
    height: auto;
    margin-top: 2vh;
    padding: 8px;
  }
}

@media (max-width: 480px) {
  .produto {
    width: 180px;
    margin-top: 1vh;
    padding: 6px;
  }
}

.produto h4 {
  font-family: 'Roboto', sans-serif;
  font-size: 15px;
  color: rgb(65, 65, 65);
  margin-top: 10px;
  height: 40px;
}

@media (max-width: 768px) {
  .produto h4 {
    font-size: 14px;
    height: 35px;
  }
}

@media (max-width: 480px) {
  .produto h4 {
    font-size: 13px;
    height: 30px;
  }
}

.lista {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  justify-items: center;
  padding: 4px;
  gap: 5px;
  position: relative;
}

@media (max-width: 800px) {
  .lista {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .filtro-categorias {
    flex-direction: column;
    align-items: stretch;
    gap: 6px;
  }
}

@media (max-width: 550px) {
  .lista {
    grid-template-columns: repeat(1, 1fr);
  }
}

@media (max-width: 768px) {
  .BTli {
    gap: 8px;
  }
}

@media (max-width: 480px) {
  .BTli {
    gap: 5px;
  }
}

@media (max-width: 768px) {
  .BTli button {
    padding: 4px 8px;
    font-size: 0.8rem;
    margin-left: 5px;
  }
}

@media (max-width: 480px) {
  .BTli button {
    padding: 3px 6px;
    font-size: 0.75rem;
    margin-left: 3px;
  }
}

/* Modal de Confirmação */
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 2000;
}

.modal-confirmacao {
    background: white;
    padding: 30px;
    border-radius: 10px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
    max-width: 400px;
    width: 90%;
    text-align: center;
}

.modal-confirmacao h3 {
    margin: 0 0 15px 0;
    color: #333;
    font-size: 1.3rem;
}

.modal-confirmacao p {
    margin: 0 0 25px 0;
    color: #666;
    font-size: 1rem;
}

.modal-botoes {
    display: flex;
    gap: 15px;
    justify-content: center;
}

.btn-confirmar {
    background-color: #dc3545;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1rem;
    transition: background-color 0.2s;
}

.btn-confirmar:hover {
    background-color: #b71c1c;
}

.btn-cancelar {
    background-color: #6c757d;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1rem;
    transition: background-color 0.2s;
}

.btn-cancelar:hover {
    background-color: #545b62;
}

@media (max-width: 480px) {
    .modal-confirmacao {
        padding: 20px;
        margin: 20px;
    }
    
    .modal-botoes {
        flex-direction: column;
        gap: 10px;
    }
    
    .btn-confirmar,
    .btn-cancelar {
        padding: 12px 20px;
    }
}

</style>
