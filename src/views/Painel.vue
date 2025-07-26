<template>
<Header></Header>
<div class="divisoria" ></div>
<div class="Tudo">
    <div class="menuesquerdo" >
        <h2> <span class="h2fake" > Olá, {{ usuario.name }}</span> 👋 </h2>
        <router-link id="primeiro" to="/dados"> <button>Meus dados</button></router-link>
        <router-link to="/carrinho"> <button>Carrinho</button></router-link>
        <router-link to="/favoritos"> <button>Favoritos</button></router-link>
        <router-link to="/pedidos"> <button>Pedidos</button></router-link>
        <router-link to="/enderecos"> <button>Endereços</button></router-link>
        <router-link to="/cupons"> <button>Cupons</button></router-link>
        <div class="admin" v-if="userRole === 'ADMIN' || userRole === 'MODERATOR'">
        <h3>ADMIN</h3>
        <router-link to="/ADMmoderadores"> <button>Gerenciar Moderadores</button></router-link>
        <router-link to="/ADMcategorias"> <button>Gerenciar categorias</button></router-link>
        <router-link to="/ADMprodutos"> <button>Gerenciar produtos</button></router-link>
        <router-link to="/ADMpedidos"> <button>Gerenciar Pedidos</button></router-link>
        <router-link to="/ADMcupons"> <button>Gerenciar Cupons</button></router-link>
        </div>
    </div>
    <div class="menudireito" >
        <router-view></router-view>
    </div>
</div>
<Footer></Footer>
</template>

<script setup>
import Header from '../components/Headercomponent.vue'
import Footer from '../components/Footercomponent.vue'
import { ref, onMounted } from 'vue'
import api from '../services/api'

const usuario = ref({})
const userRole = ref(null)

onMounted(async () => {
    try {
        const token = localStorage.getItem('token')
        if (token) {
            api.defaults.headers.common['Authorization'] = `Bearer ${token}`
            const { data } = await api.get('/users/me')
            usuario.value = data
            userRole.value = data.role
        }
    } catch (e) {
        userRole.value = null
    }
})
</script>

<style scoped>

.divisoria{
    width: 100%;
    height: 15px;
    background-color: #06080afa;
}

.Tudo{
    display: flex;
    justify-content: center;
    width: 100%;
    height: 100vh;
    padding: 20px;
}

.menuesquerdo{
    max-width: 300px;
    width: 16%;
    height: 100%;
    background-color: #06080afa;
    border: px solid white;
    min-width: 266px;
    z-index: 10;
    border-radius: 12px 0 0 12px;
}

.menuesquerdo h2 {
    color: white;
    text-align: center;
    padding-top: 20px;
    font-size: 1.4rem;
}

.h2fake {
    text-decoration: underline;
}

.menuesquerdo h3{
    color: #ffffff;
    font-size: 1.5rem;
    font-weight: bold;
    text-align: center;
    margin-top: 20px;
    text-decoration: underline;
}

.menuesquerdo button{
    width: 100%;
    height: 60px;
    color: #ffffff;
    font-weight: bold;
    font-size: 1rem;
}

#primeiro button {
    border-radius: 12px 0 0 0;
}

.menuesquerdo button:hover{
    transition: 0.1s;
    background-color: #eeeeee;
    color: #000000;
}


.menudireito{
    width: 77%;
    height: 100%;
    background-color: #ffffff;
    border: 1px solid black;
}

.admin button{
    color: #00b7ff;
}

</style>