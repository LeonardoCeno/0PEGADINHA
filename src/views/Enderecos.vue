<template>
    <div class="tudo">
        <div class="container-central">
            <div class="titulo-bar">
                <div class="titulo">
                    <h1>Endereços</h1>
                </div>
                <button class="btn-novo-endereco" @click="mostrarModal = true">Novo Endereço</button>
            </div>

            <!-- Modal de novo endereço -->
            <div v-if="mostrarModal" class="modal-overlay" @click.self="mostrarModal = false">
                <div class="modal-form">
                    <h2>Novo Endereço</h2>
                    <form class="form-endereco" @submit.prevent="adicionarEndereco">
                        <div class="campos-form">
                            <input v-model="novoEndereco.street" type="text" placeholder="Rua" required />
                            <input v-model="novoEndereco.number" type="number" placeholder="Número" required />
                            <input v-model="novoEndereco.zip" type="text" placeholder="CEP" required />
                            <input v-model="novoEndereco.city" type="text" placeholder="Cidade" required />
                            <input v-model="novoEndereco.state" type="text" placeholder="Estado" required />
                            <input v-model="novoEndereco.country" type="text" placeholder="País" required />
                        </div>
                        <div class="botoes-modal">
                            <button type="submit">Salvar</button>
                            <button type="button" class="btn-cancelar" @click="mostrarModal = false">Cancelar</button>
                        </div>
                    </form>
                </div>
            </div>

            <!-- Modal de edição de endereço -->
            <div v-if="modalEditar" class="modal-overlay" @click.self="fecharModalEditar">
                <div class="modal-form">
                    <h2>Editar Endereço</h2>
                    <form class="form-endereco" @submit.prevent="salvarEdicaoEndereco">
                        <div class="campos-form">
                            <input v-model="enderecoEditando.street" type="text" placeholder="Rua" required />
                            <input v-model="enderecoEditando.number" type="number" placeholder="Número" required />
                            <input v-model="enderecoEditando.zip" type="text" placeholder="CEP" required />
                            <input v-model="enderecoEditando.city" type="text" placeholder="Cidade" required />
                            <input v-model="enderecoEditando.state" type="text" placeholder="Estado" required />
                            <input v-model="enderecoEditando.country" type="text" placeholder="País" required />
                        </div>
                        <div class="botoes-modal">
                            <button type="submit">Salvar</button>
                            <button type="button" class="btn-cancelar" @click="fecharModalEditar">Cancelar</button>
                        </div>
                    </form>
                </div>
            </div>

            <div class="enderecos-lista">
                <div v-if="carregando" class="carregando">Carregando endereços...</div>
                <div v-else-if="erro" class="erro">{{ erro }}</div>
                <div v-else>
                    <div v-if="enderecos.length === 0" class="nenhum-endereco">Nenhum endereço cadastrado.</div>
                    <div v-else class="enderecos-grid alinhado-esquerda">
                        <div v-for="endereco in enderecos" :key="endereco.id" class="endereco-card">
                            <div class="linha"><strong>Rua:</strong> {{ endereco.street }}</div>
                            <div class="linha"><strong>Número:</strong> {{ endereco.number }}</div>
                            <div class="linha"><strong>CEP:</strong> {{ endereco.zip }}</div>
                            <div class="linha"><strong>Cidade:</strong> {{ endereco.city }}</div>
                            <div class="linha"><strong>Estado:</strong> {{ endereco.state }}</div>
                            <div class="linha"><strong>País:</strong> {{ endereco.country }}</div>
                            <div class="acoes-endereco">
                                <button class="btn-acao editar" @click="abrirModalEditar(endereco)">Editar</button>
                                <button class="btn-acao excluir" @click="excluirEndereco(endereco.id)">Excluir</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { getEnderecos, criarEndereco, atualizarEndereco, deletarEndereco } from '../services/api'

const enderecos = ref([])
const carregando = ref(true)
const erro = ref(null)
const mostrarModal = ref(false)
const modalEditar = ref(false)
const enderecoEditando = ref({})

const novoEndereco = ref({
    street: '',
    number: '',
    zip: '',
    city: '',
    state: '',
    country: ''
})

async function carregarEnderecos() {
    carregando.value = true
    erro.value = null
    try {
        enderecos.value = await getEnderecos()
    } catch (e) {
        erro.value = 'Erro ao carregar endereços.'
    } finally {
        carregando.value = false
    }
}

async function adicionarEndereco() {
    try {
        await criarEndereco(novoEndereco.value)
        await carregarEnderecos()
        novoEndereco.value = { street: '', number: '', zip: '', city: '', state: '', country: '' }
        mostrarModal.value = false
    } catch (e) {
        erro.value = 'Erro ao adicionar endereço.'
    }
}

function abrirModalEditar(endereco) {
    enderecoEditando.value = { ...endereco }
    modalEditar.value = true
}

function fecharModalEditar() {
    modalEditar.value = false
    enderecoEditando.value = {}
}

async function salvarEdicaoEndereco() {
    try {
        await atualizarEndereco(enderecoEditando.value.id, enderecoEditando.value)
        await carregarEnderecos()
        fecharModalEditar()
    } catch (e) {
        erro.value = 'Erro ao editar endereço.'
    }
}

async function excluirEndereco(id) {
    if (confirm('Tem certeza que deseja excluir este endereço?')) {
        try {
            await deletarEndereco(id)
            await carregarEnderecos()
        } catch (e) {
            erro.value = 'Erro ao excluir endereço.'
        }
    }
}

onMounted(carregarEnderecos)
</script>

<style scoped>
.tudo {
    width: 100%;
    min-height: 100vh;
    background-color: #ffffff;
    padding-bottom: 40px;
}

.container-central {
    max-width: 1100px;
    margin: 0 auto;
    padding: 0 32px;
    box-sizing: border-box;
    width: 100%;
}

.titulo-bar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    background-color: #ffffff;
    height: 17vh;
    width: 100%;
    box-sizing: border-box;
}
.titulo {
    display: flex;
    align-items: center;
    justify-content: flex-start;
    font-size: 1.5rem;
    height: 100%;
}

.btn-novo-endereco {
    background: #3a9c73;
    color: #fff;
    border: none;
    border-radius: 6px;
    padding: 10px 22px;
    font-size: 1.1rem;
    font-weight: bold;
    cursor: pointer;
    transition: background 0.2s;
    box-shadow: 0 2px 8px rgba(0,0,0,0.07);
}
.btn-novo-endereco:hover {
    background: #287a56;
}

.modal-overlay {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    background: rgba(0,0,0,0.25);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
}
.modal-form {
    background: #fff;
    border-radius: 16px;
    box-shadow: 0 6px 32px rgba(0,0,0,0.15);
    padding: 32px 28px 24px 28px;
    min-width: 320px;
    max-width: 370px;
    width: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    animation: modalShow 0.2s;
}
@keyframes modalShow {
    from { opacity: 0; transform: translateY(-30px); }
    to { opacity: 1; transform: translateY(0); }
}
.modal-form h2 {
    margin-bottom: 18px;
    color: #3a9c73;
    font-size: 1.35rem;
    font-weight: 700;
}
.form-endereco {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
    gap: 0;
}
.campos-form {
    display: flex;
    flex-direction: column;
    gap: 13px;
    width: 100%;
}
.form-endereco input {
    padding: 10px 12px;
    border: 1.5px solid #e0e0e0;
    border-radius: 7px;
    font-size: 1rem;
    background: #fafbfc;
    transition: border 0.2s, box-shadow 0.2s;
    outline: none;
    box-shadow: 0 1px 2px rgba(0,0,0,0.03);
}
.form-endereco input:focus {
    border: 1.5px solid #3a9c73;
    background: #fff;
    box-shadow: 0 2px 8px rgba(58,156,115,0.08);
}
.botoes-modal {
    display: flex;
    gap: 10px;
    margin-top: 18px;
    width: 100%;
    justify-content: flex-end;
}
.form-endereco button {
    padding: 9px 22px;
    background: #3a9c73;
    color: #fff;
    border: none;
    border-radius: 6px;
    font-size: 1rem;
    cursor: pointer;
    font-weight: bold;
    transition: background 0.2s, box-shadow 0.2s;
    box-shadow: 0 2px 8px rgba(0,0,0,0.07);
}
.form-endereco button:hover {
    background: #287a56;
    box-shadow: 0 4px 16px rgba(58,156,115,0.13);
}
.btn-cancelar {
    background: #c0392b !important;
}
.btn-cancelar:hover {
    background: #a93226 !important;
}

.enderecos-lista {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    width: 100%;
    box-sizing: border-box;
}
.enderecos-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
    gap: 20px;
    width: 100%;
    max-width: 900px;
    margin-top: 20px;
}
.alinhado-esquerda {
    justify-items: start;
}
.endereco-card {
    background: #f7f7f7;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.07);
    padding: 18px 22px;
    display: flex;
    flex-direction: column;
    gap: 6px;
    font-size: 1.08rem;
    border-left: 5px solid #3a9c73;
    position: relative;
}
.acoes-endereco {
    display: flex;
    gap: 10px;
    margin-top: 10px;
}
.btn-acao {
    padding: 6px 16px;
    border: none;
    border-radius: 5px;
    font-size: 0.98rem;
    font-weight: 500;
    cursor: pointer;
    transition: background 0.2s, color 0.2s;
}
.btn-acao.editar {
    background: #68655a;
    color: #fff;
}
.btn-acao.editar:hover {
    background: #504e42;
}
.btn-acao.excluir {
    background: #e74c3c;
    color: #fff;
}
.btn-acao.excluir:hover {
    background: #922b21;
}
.linha {
    margin-bottom: 2px;
}
.nenhum-endereco {
    color: #888;
    margin-top: 20px;
    font-size: 1.1rem;
}
.carregando {
    color: #3a9c73;
    margin-top: 20px;
    font-size: 1.1rem;
}
.erro {
    color: #c0392b;
    margin-top: 20px;
    font-size: 1.1rem;
}
</style>