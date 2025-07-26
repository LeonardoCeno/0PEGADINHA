<template>
    <div class="fixador" >
    <!-- Nova barrinha no topo -->
    <div class="top-bar">
        <div class="top-bar-links">
            <router-link to="/" class="top-link">Home</router-link>
            <router-link to="/favoritos" class="top-link">Favoritos</router-link>
            <router-link to="/pedidos" class="top-link">Meus Pedidos</router-link>
        </div>
    </div>
    
    <header>
        <a class="logo desktop" href="/">
        <img src="../components/img/agrsimtabao-Photoroom.png" alt="" />
        </a>
        <div class="input desktop" style="position:relative;">
            <input type="text" placeholder="Livros, Mangás, novos olhares..." v-model="busca" @input="onInputBusca" @focus="onFocusBusca" @blur="onBlurBusca" @keyup.enter="pesquisarEnter" />
            <img src="../components/img/LupaFinal.png" alt="" />
            <div v-if="mostrarSugestoes && sugestoes.length > 0 && busca.length > 0" class="autocomplete-sugestoes" @mousedown.prevent>
                <div class="autocomplete-titulo">
                    Resultados para "{{ busca }}"
                </div>
                <div v-for="produto in sugestoes.slice(0, 3)" :key="produto.id" class="sugestao-item" @mousedown.prevent="irParaProduto(produto.id)">
                    <img v-if="produto.image_path" :src="produto.image_path.startsWith('http') ? produto.image_path : apiBase + produto.image_path" alt="imagem" />
                    <div class="sugestao-info">
                        <span class="disponivel">
                            <img :src="produto.stock >= 1 ? DISPONIVELREAL : INDISPONIVELREAL" :alt="produto.stock >= 1 ? 'Disponível' : 'Indisponível'" />
                        </span>
                        <span class="sugestao-nome">{{ produto.name }}</span>
                        <span class="sugestao-preco">R$ {{ produto.price }}</span>
                    </div>
                </div>
            </div>
        </div>
        <div class="botoes desktop">
            <button>
            <p>Carrinho</p>
            <img src="../components/img/carrinhofinal.png" alt="" />
            </button>
            <button>
                <p>Pedidos</p>
                <img src="../components/img/listafinal.png" alt="" />
            </button>
            <router-link v-if="!isLoggedIn" to="/login">   
            <button>
                <p>Entrar</p>
                <img src="../components/img/usuariofinal.png" alt="" />
            </button>
            </router-link>
            <div v-else class="conta-dropdown-wrapper" @mouseenter="showDropdown = true" @mouseleave="showDropdown = false" style="position: relative; display: inline-block;">
                <button>
                    <p>Conta</p>
                    <img src="../components/img/usuariofinal.png" alt="" />
                </button>
                <div v-if="showDropdown" class="conta-dropdown-menu">
                    <button @click="goToPainel">Dados</button>
                    <button @click="logout">Sair</button>
                </div>
            </div>
        </div>
        <div class="topo-mobile mobile">
            <a class="logo" href="/">
            <img src="../components/img/agrsimtabao-Photoroom.png" alt="" />
            </a>
            <div class="botoes">
                <router-link v-if="!isLoggedIn" to="/login">   
            <button>
                <p>Entrar</p>
                <img src="../components/img/usuariofinal.png" alt="" />
            </button>
            </router-link>
            <div v-else class="conta-dropdown-wrapper" @mouseenter="showDropdown = true" @mouseleave="showDropdown = false" style="position: relative; display: inline-block;">
                <button>
                    <p>Conta</p>
                    <img src="../components/img/usuariofinal.png" alt="" />
                </button>
                <div v-if="showDropdown" class="conta-dropdown-menu">
                    <button @click="goToPainel">Dados</button>
                    <button @click="logout">Sair</button>
                </div>
            </div>
                <button>
                <p>Pedidos</p>
                <img src="../components/img/listafinal.png" alt="" />
                </button>
                <button>
                <p>Carrinho</p>
                <img src="../components/img/carrinhofinal.png" alt="" />
                </button>
            </div>
        </div>
        <div class="input mobile" style="position:relative;">
            <input type="text" placeholder="Pesquisar" v-model="busca" @input="onInputBusca" @focus="onFocusBusca" @blur="onBlurBusca" @keyup.enter="pesquisarEnter" />
            <img src="../components/img/LupaFinal.png" alt="" />
            <div v-if="mostrarSugestoes && sugestoes.length > 0" class="autocomplete-sugestoes" @mousedown.prevent>
                <div class="autocomplete-titulo">
                    Resultados para "{{ busca }}"
                </div>
                <div v-for="produto in sugestoes.slice(0, 3)" :key="produto.id" class="sugestao-item" @mousedown.prevent="irParaProduto(produto.id)">
                    <img v-if="produto.image_path" :src="produto.image_path.startsWith('http') ? produto.image_path : apiBase + produto.image_path" alt="imagem" />
                    <div class="sugestao-info">
                        <span class="sugestao-nome">{{ produto.name }}</span>
                        <span class="sugestao-preco">R$ {{ produto.price }}</span>
                    </div>
                </div>
            </div>
        </div>
    </header>

    <div v-if="!esconderCategorias" class="Categorias">
        <div class="categorias-dropdown-wrapper" @mouseenter="showCategoriasDropdown = true" @mouseleave="showCategoriasDropdown = false" style="position: relative; display: inline-block;">
            <button href="#">
                <img src="../components/img/listafinal.png" alt=""> <p>Categorias</p>
            </button>
            <div v-if="showCategoriasDropdown" class="categorias-dropdown-menu">
                <div v-for="cat in categorias" :key="cat.id">
                    <button @click="irParaCategoria(cat.id)"> {{ cat.name }} </button>
                </div>
                <button @click="irParaPesquisas">Tudo</button>
            </div>
        </div>
        <button @click="irParaLancamentos"> <img src="../components/img/Lancamentofinal-Photoroom.png" alt=""> <p>Lançamentos</p></button>
        <button @click="irParaLivros"> <img src="../components/img//Livrofinalverdadeiro-Photoroom.png" alt=""> <p>Livros</p></button>
        <button @click="irParaMangás"> <img src="../components/img/mangáfinal.png" alt=""> <p>Mangás</p></button>
        <button @click="irParaArtbooks" > <img src="../components/img/pincel.png" alt=""> <p>Artbooks</p></button>
        <button @click="irParaOfertas"> <img src="../components/img/ofertasfinal.png" alt=""> <p>Ofertas</p></button>
        <button @click="irParaPesquisas"> <img src="../components/img/Tudofinal-Photoroom.png" alt=""> <p>Tudo</p></button>
    </div>
</div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import api, { buscarProdutosAdmin228 } from '../services/api'
import { useRouter, useRoute } from 'vue-router'
import DISPONIVELREAL from './img/DISPONIVELREAL.png'
import INDISPONIVELREAL from './img/INDISPONIVELREAL.png'
import { getCategoriasPorUsuario228 } from '../services/api'

const apiBase = 'http://35.196.79.227:8000'
// Função para checar se o usuário ta logado (token em memória)
const isLoggedIn = computed(() => !!api.defaults.headers.common['Authorization'])
const showDropdown = ref(false)
const showCategoriasDropdown = ref(false)
const router = useRouter()
const route = useRoute()
const categorias = ref([])

// isso carrega as categorias criadas para que atualize o "categorias" do header automaticamente ao o admin criar um nova
async function carregarCategorias() {
    try {
        categorias.value = await getCategoriasPorUsuario228()
    } catch (error) {
        console.error('Erro ao carregar categorias:', error)
    }
}

onMounted(() => {
    carregarCategorias()
})

// isso aqui é pra que se o usuario estiver no painel a parte das categorias abaixo do header não aparecer (sim, é uma soluçao pra nao ter que separar ele do header pq fiz os 2 junto num outro componente 👍)
const painelChildrenRoutes = [
    'Dados', 'Cupons', 'Pedidos', 'Favoritos', 'Enderecos', 'Carrinho',
    'ADMcategorias', 'ADMprodutos', 'ADMpedidos', 'ADMcupons', 'ADMmoderadores'
]
const esconderCategorias = computed(() => {
    return painelChildrenRoutes.includes(route.name)
})

// aqui cuida da parte das sugestoes que aparece ao você digitar algo no input do header, ele só te sugere se no input tiver 1 letra ou mais (e funciona bem 👍)
const busca = ref('')
const sugestoes = ref([])
const mostrarSugestoes = ref(false)
let todosProdutosAdmin = []
let timeoutBusca = null


// simples, carrega os itens do ADM (eu)
async function carregarProdutosAdmin() {
    if (todosProdutosAdmin.length === 0) {
        todosProdutosAdmin = await buscarProdutosAdmin228()
    }
}

// função async que faz o input funcionar, buscando os produtos que mais batem com o que você pesquisou
async function onInputBusca() {
    clearTimeout(timeoutBusca)
    await carregarProdutosAdmin()
    timeoutBusca = setTimeout(() => {
        const termo = busca.value.toLowerCase()
        sugestoes.value = todosProdutosAdmin
            .filter(p => p.name && p.name.toLowerCase().includes(termo))
            .map(p => ({
                ...p,
                matchIndex: p.name.toLowerCase().indexOf(termo)
            }))
            .sort((a, b) => {
                // Prioriza nomes que tem o termo mais "próximo" do início, mas não só no início
                if (a.matchIndex !== b.matchIndex) return a.matchIndex - b.matchIndex
                // Se o índice for igual, prioriza nomes mais curtos
                return a.name.length - b.name.length
            })
            .slice(0, 5)
        mostrarSugestoes.value = sugestoes.value.length > 0
    }, 100)
}

// aqui que faz aparecer só se tem uma ou mais letras👍
function onFocusBusca() {
    if (sugestoes.value.length > 0) {
        mostrarSugestoes.value = true
    }
}

// limpa
function onBlurBusca() {
    setTimeout(() => {
        mostrarSugestoes.value = false
        busca.value = ''
    }, 120)
}


//to fazendo ainda, mas aqui vai te legar pra pagina do produto q se clico
function irParaProduto(id) {
    router.push(`/produto/${id}`)
    mostrarSugestoes.value = false
    busca.value = ''
}

// função que faz o usuario deslogar e perde a permissao de certas areás q só pode se tiver logado
function logout() {
    localStorage.removeItem('token')
    delete api.defaults.headers.common['Authorization']
    showDropdown.value = false
    window.location.reload()
}

//daqui pra baixo é funçoes que levam as coisas q tao escrita abaixo do header
function goToPainel() {
    showDropdown.value = false
    router.push('/dados')
}
function irParaPesquisas() {
    router.push('/pesquisas')
}
function irParaLancamentos() {
    router.push({ path: '/pesquisas', query: { lancamentos: 1 } })
}
function irParaLivros() {
    router.push({ path: '/pesquisas', query: { categoriaId: 316 } })
}
function irParaArtbooks() {
    router.push({ path: '/pesquisas', query: { categoriaId: 320 } })
}
function irParaOfertas() {
    //vou por ainda
}
function irParaMangás() {
    router.push({ path: '/pesquisas', query: { categoriaId: 318 } })
}
// aqui criei um "genérico" pras categorias dentro da div que abre quando voce passa o mouse em cima da "Categorias" do header
// nao repliquei em todas pra que o Categorias (falo da div toda) fique fixo, e pra funções futuras
function irParaCategoria(id) {
    router.push({ path: '/pesquisas', query: { categoriaId: id } })
}
// isso aqui é o que te leva pro pesquisas.vue com base no que voce pesquisou
function pesquisarEnter() {
    if (busca.value.trim().length > 0) {
        router.push({ path: '/pesquisas', query: { termo: busca.value.trim() } })
        busca.value = ''
        mostrarSugestoes.value = false
    }
}

</script>

<style scoped>

.top-bar {
    background: #ffffff;
    padding: 8px 0;
    border-bottom: 1px solid #02060af5;
}

.top-bar-links {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 40px;
    max-width: 1200px;
    margin: 0 auto;
}

.top-link {
    color: #02060af5;
    text-decoration: none;
    font-size: 14px;
    font-weight: 500;
    transition: color 0.2s ease;
    font-family: helvetica;
}

.top-link:hover {
    color: #079ac7;
}

.top-link.router-link-active {
    color: #079ac7;
    font-weight: 600;
}

header {
    display: flex;
    align-items: center;
    justify-content: center;
    background: linear-gradient(135deg, #ffffff 0%, #f8f9fa 100%);
    min-height: 12vh;
    gap: 6vw;
    flex-wrap: wrap;
    padding: 15px 20px;
}

.logo {
    position: relative;
    justify-content: center;
    left: 20px;
    width: 12%;
    display: flex;
}

.logo img {
    width: auto;
    height: 95px;
    filter: brightness(15%);
    transform: rotate(-9deg);
}

.logo img:hover{
    transition: 0.4s ease-in-out;
    filter: brightness(65%);
}

.input {
    display: flex;
    align-items: center;
    justify-content: space-between;
    background-color: #ffffff;
    border-radius: 15px;
    padding: 0 20px;
    width: 41vw;
    max-width: 720px;
    height: 50px;
    flex-shrink: 1;
    border: 1px solid #6d6d6d;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
}

.input:focus-within {
    border: 1.8px solid #03040cf5;
}

.input img {
    width: auto;
    height: 4vh;
    filter: brightness(20%);
}

input {
    width: 32vw;
    border: none;
    outline: none;
    background-color: transparent;
    min-width: 100px;
    font-size: 16px;
    color: #2c3e50;
    font-weight: 500;
}

input::placeholder {
    color: #95a5a6;
    font-style: italic;
    font-weight: 400;
}

.botoes {
    display: flex;
    gap: 10px;
    font-size: 15px;
    flex-wrap: wrap;
    justify-content: center;
    position: relative;
    right: 15px;
}

.botoes p {
    font-family: helvetica;
    font-weight: bold;
    font-size: 16px;
}

button {
    display: flex;
    align-items: center;
    gap: 6px;
    color: #000000;
    white-space: nowrap;
    background: transparent;
    border: none;
    padding: 6px 12px;
    cursor: pointer;
    transition: color 0.1s;
    font-weight: 500;
    font-size: 15px;
    height: 55px;
}

.botoes button:hover {
    color: #079ac7;
}

button img {
    width: auto;
    height: 3vh;
    transition: 0.1s;
}

button:hover img {
    filter: invert(45%) sepia(65%) saturate(1000%) hue-rotate(160deg) brightness(105%) contrast(100%);
}

.botoes img {
    width: 24px;
    height: 24px;
}

.mobile {
    display: none;
}

@media (max-width: 768px) {
    .desktop {
        display: none;
    }
    .mobile {
        display: flex;
    }
    header {
        flex-direction: column;
        align-items: center;
        gap: 15px;
    }

    .topo-mobile {
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        width: 90vw;
    }

    .topo-mobile .logo {
        justify-content: flex-start;
        width: auto;
    }

    .topo-mobile .logo img {
        height: 60px;
        filter: brightness(20%);
    }

    .topo-mobile .logo img:hover {
        transition: 0.4s ease-in-out;
        filter: brightness(65%);
    }

    .input {
        width: 90vw;
    }

    input {
        width: 75vw;
    }

    .botoes {
        gap: 20px;
        flex-wrap: nowrap;
        justify-content: flex-end;
    }

    button p {
        font-size: 14px;
    }

    button img {
        height: 4vh;
    }

    .logo {
        position: static;
    }
}

/* Responsividade para tablets pequenos (600px - 767px) */
@media (min-width: 600px) and (max-width: 767px) {
    .topo-mobile .logo img {
        height: 50px;
    }
    
    .botoes {
        gap: 15px;
    }
    
    button p {
        font-size: 13px;
    }
    
    .input {
        width: 85vw;
    }
}

/* Responsividade para celulares grandes (480px - 599px) */
@media (min-width: 480px) and (max-width: 599px) {
    .topo-mobile .logo {
        display: none; /* Oculta a logo em telas pequenas */
    }
    
    .topo-mobile {
        justify-content: center;
    }
    
    .botoes {
        gap: 12px;
    }
    
    button p {
        font-size: 12px;
    }
    
    button img {
        height: 3.5vh;
    }
    
    .input {
        width: 88vw;
    }
    
    input {
        width: 70vw;
    }
}

/* Responsividade para celulares pequenos (320px - 479px) */
@media (min-width: 320px) and (max-width: 479px) {
    .topo-mobile .logo {
        display: none; /* Oculta a logo em telas pequenas */
    }
    
    .topo-mobile {
        justify-content: center;
    }
    
    .botoes {
        gap: 8px;
    }
    
    button p {
        font-size: 11px;
    }
    
    button img {
        height: 3vh;
    }
    
    .input {
        width: 92vw;
    }
    
    input {
        width: 65vw;
    }
    
    header {
        gap: 10px;
        padding: 8px;
    }
}

/* Responsividade para celulares muito pequenos (< 320px) */
@media (max-width: 319px) {
    .topo-mobile .logo {
        display: none; /* Oculta a logo em telas pequenas */
    }
    
    .topo-mobile {
        justify-content: center;
    }
    
    .botoes {
        gap: 5px;
    }
    
    button p {
        font-size: 10px;
    }
    
    button img {
        height: 2.5vh;
    }
    
    .input {
        width: 95vw;
    }
    
    input {
        width: 60vw;
    }
    
    header {
        gap: 8px;
        padding: 5px;
    }
}

.Categorias {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 6vh;
    background: #02060af5;
    font-size: 14px;
    gap: 70px;
}

.Categorias img {
    width: 1.1vw;
    min-width: 12px;
    height: auto;
    filter: brightness(0) invert(1);
    transition: all 0.3s ease;
}

.Categorias button {
    display: flex;
    align-items: center;
    gap: 7px;
    padding: 5px 10px;
    border-radius: 10px;
    transition: color 0.2s ease;
    color: white;
    height: 55px;
    font-weight: bold;
    background: transparent;
}

.Categorias button:hover {
    color: #63b3ed;
    background: transparent;
}

.Categorias button:hover img {
    filter: brightness(0) invert(45%) sepia(65%) saturate(1050%) hue-rotate(160deg) brightness(115%) contrast(100%);
}

.conta-dropdown-menu {
    position: absolute;
    top: 51px;
    background: #fff;
    border: 1px solid #e5e7eb;
    box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    min-width: 100px;
    z-index: 100;
    display: flex;
    flex-direction: column;
    padding: 8px 0;
    margin-top: 4px;
    border: 1px, solid #000000;
}
.conta-dropdown-menu button {
    color: #000000;
    padding: 10px 18px;
    text-align: left;
    width: 160px;
    font-size: 15px;
    cursor: pointer;
    transition: background 0.2s;
}
.conta-dropdown-menu button:hover {
    color: #079ac7;
}

.categorias-dropdown-menu {
    position: absolute;
    top: 51px;
    background: #fff;
    border: 1px solid #e5e7eb;
    box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    min-width: 150px;
    z-index: 100;
    display: flex;
    flex-direction: column;
    padding: 8px 0;
    margin-top: 4px;
    border: 1px, solid #000000;
}
.categorias-dropdown-menu button {
    color: #000000;
    padding: 10px 18px;
    text-align: left;
    width: 200px;
    font-size: 15px;
    cursor: pointer;
    border-radius: 0px;
}
.categorias-dropdown-menu button:hover {
    color: #079ac7;
    transition: 0s;
    border-top: 0.5px solid grey;
    border-bottom: 0.5px solid grey;
}

.autocomplete-sugestoes {
    position: absolute;
    top: 45px;
    left: 0;
    width: 100%;
    background: #fff;
    border: 1.5px solid #979797;
    border-radius: 0 0 12px 12px;
    z-index: 100;
    padding: 4px 0;
}

.sugestao-item {
    display: flex;
    align-items: center;
    gap: 18px;
    padding: 10px 18px;
    cursor: pointer;
    border-bottom: 1px solid #e9e9e9;
}

.sugestao-item img {
    width: 80px;
    height: 108px;
    filter: none;
}

.sugestao-info {
    display: flex;
    flex-direction: column;
    justify-content: center;
}

.sugestao-info .disponivel img{
    width: 90px;
    height: auto;
    border-radius: 7px;
}

.sugestao-nome {
    font-size: 1.08rem;
    font-weight: 600;
    color: #222;
    text-overflow: ellipsis;
    margin-bottom: 2px;
}

.sugestao-preco {
    font-weight: Bold;
    font-size: 1.2rem;
    color: #414141;
}

.sugestao-item:hover {
    background: #dfdfdf;
}

.autocomplete-titulo {
    color: #888;
    font-size: 0.98rem;
    font-weight: 500;
    padding: 10px 18px 4px 18px;
    margin-bottom: 2px;
    margin-top: 2px;
    text-align: left;
    letter-spacing: 0.01em;
}

@media (max-width: 950px) {
    .Categorias button:nth-child(3),
    .Categorias button:nth-child(5),
    .Categorias button:nth-child(4) {
        display: none;
    }
    .Categorias {
        gap: 55px;
    }
}

@media (max-width: 768px) {
    .Categorias {
        gap: 40px;
        padding: 0 10px;
    }
    
    .Categorias button {
        padding: 4px 8px;
        font-size: 13px;
    }
    
    .Categorias img {
        width: 10px;
        min-width: 10px;
    }
}

@media (max-width: 550px) {
    .Categorias button:nth-child(6) {
        display: none;
    }
    .Categorias {
        gap: 30px;
        padding: 0 5px;
    }
    
    .Categorias button {
        padding: 3px 6px;
        font-size: 12px;
    }
    
    .Categorias img {
        width: 8px;
        min-width: 8px;
    }
}

@media (max-width: 480px) {
    .Categorias {
        gap: 20px;
        padding: 0 3px;
    }
    
    .Categorias button {
        padding: 2px 4px;
        font-size: 11px;
    }
    
    .Categorias img {
        width: 6px;
        min-width: 6px;
    }
}

@media (max-width: 320px) {
    .Categorias {
        gap: 15px;
        padding: 0 2px;
    }
    
    .Categorias button {
        padding: 1px 3px;
        font-size: 10px;
    }
    
    .Categorias img {
        width: 5px;
        min-width: 5px;
    }
}

@media (max-width: 550px) {
    .top-bar-links {
        gap: 20px;
        padding: 0 15px;
    }
    
    .top-link {
        font-size: 12px;
    }
}

@media (max-width: 768px) {
    .top-bar-links {
        gap: 25px;
    }
    
    .top-link {
        font-size: 13px;
    }
}

@media (max-width: 480px) {
    .top-bar-links {
        gap: 15px;
        padding: 0 10px;
    }
    
    .top-link {
        font-size: 12px;
    }
}

@media (max-width: 320px) {
    .top-bar-links {
        gap: 10px;
        padding: 0 5px;
    }
    
    .top-link {
        font-size: 11px;
    }
}

/* Responsividade para TVs grandes (1200px+) */
@media (min-width: 1200px) {
    header {
        gap: 8vw;
        padding: 15px 30px;
    }
    
    .input {
        width: 35vw;
        max-width: 800px;
    }
    
    .logo img {
        height: 110px;
    }
    
    .botoes {
        gap: 15px;
    }
    
    .botoes p {
        font-size: 18px;
    }
    
    .botoes img {
        width: 28px;
        height: 28px;
    }
}

/* Responsividade para monitores médios (992px - 1199px) */
@media (min-width: 992px) and (max-width: 1199px) {
    header {
        gap: 7vw;
        padding: 12px 25px;
    }
    
    .input {
        width: 38vw;
        max-width: 750px;
    }
    
    .logo img {
        height: 100px;
    }
    
    .botoes {
        gap: 12px;
    }
    
    .botoes p {
        font-size: 17px;
    }
}

/* Responsividade para tablets grandes (768px - 991px) */
@media (min-width: 768px) and (max-width: 991px) {
    header {
        gap: 6vw;
        padding: 10px 20px;
    }
    
    .input {
        width: 42vw;
        max-width: 650px;
    }
    
    .logo img {
        height: 85px;
    }
    
    .botoes {
        gap: 10px;
    }
    
    .botoes p {
        font-size: 16px;
    }
    
    .botoes img {
        width: 22px;
        height: 22px;
    }
}

</style>
