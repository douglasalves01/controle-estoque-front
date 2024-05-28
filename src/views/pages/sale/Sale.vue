<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import { useToast } from 'primevue/usetoast';
const toast = useToast();
const productsTeste = ref([]);
const layout = ref('grid');
const carrinho = ref([]);

onMounted(() => {
    const token = localStorage.getItem('authorization');
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .get('http://localhost:8000/produto-active')
        .then((response) => {
            if (response.status === 200) {
                productsTeste.value = response.data;
            }
        })
        .catch((error) => {
            console.log(error.response.data.message);
        });
});

function adicionarAoCarrinho(id, nome, valor) {
    const produtoNoCarrinho = carrinho.value.find((item) => item.id === id);
    if (produtoNoCarrinho) {
        produtoNoCarrinho.quantidade++;
    } else {
        carrinho.value.push({ id, nome, valor, quantidade: 1 });
    }
}

function removerDoCarrinho(id) {
    const index = carrinho.value.findIndex((item) => item.id === id);
    if (index !== -1) {
        if (carrinho.value[index].quantidade > 1) {
            carrinho.value[index].quantidade--;
        } else {
            carrinho.value.splice(index, 1);
        }
    }
}

const totalCarrinho = computed(() => {
    return carrinho.value.reduce((total, item) => total + item.valor * item.quantidade, 0);
});

function realizarVenda() {
    const id_produto = carrinho.value.map((item) => item.id.toString());
    const quantidade = carrinho.value.map((item) => item.quantidade.toString());
    const token = localStorage.getItem('authorization');

    if (!id_produto.length) {
        toast.add({ severity: 'error', summary: 'Erro', detail: 'Insira o produto no carrinho!', life: 3000 });
        return;
    }

    if (!quantidade.length) {
        toast.add({ severity: 'error', summary: 'Erro', detail: 'Insira a quantidade do produto!', life: 3000 });
        return;
    }
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .post('http://127.0.0.1:8000/venda', { id_produto, quantidade })
        .then((response) => {
            toast.add({ severity: 'success', summary: 'Successful', detail: 'Venda realizada com sucesso!', life: 3000 });
            carrinho.value = []; // Clear the cart after a successful sale
            console.log(response.data);
        })
        .catch((error) => {
            const message = error.response.data.message;
            toast.add({ severity: 'error', summary: 'Error', detail: message, life: 3000 });
        });
}
</script>

<template>
    <div class="grid">
        <!-- DataView ocupando 66.67% da largura -->
        <div class="col-8">
            <div class="card">
                <DataView :value="productsTeste" :layout="layout">
                    <template #grid="slotProps">
                        <div class="grid">
                            <div v-for="(item, index) in slotProps.items" :key="index" class="col-12 md:col-4 card-product" @click="adicionarAoCarrinho(item[0], item[1], item[2])">
                                <div class="flex flex-column sm:flex-row sm:align-items-center p-3 gap-2">
                                    <div class="flex flex-column md:flex-row justify-content-between md:align-items-center flex-1 gap-4">
                                        <div class="flex flex-row md:flex-column justify-content-between align-items-start gap-2">
                                            <div>
                                                {{ item[0] }}
                                                <span class="text-xl font-semibold text-900">{{ item[1] }}</span>
                                            </div>
                                        </div>
                                        <div class="flex flex-column md:align-items-end gap-5">
                                            <span class="text-xl font-semibold text-900">R$ {{ item[2].toFixed(2) }}</span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </template>
                </DataView>
            </div>
        </div>
        <!-- Div ao lado do DataView ocupando 33.33% da largura -->
        <div class="col-4">
            <div class="card">
                <h3>Carrinho</h3>
                <ul>
                    <li v-for="(item, index) in carrinho" :key="index">
                        {{ item.nome }} - R$ {{ (item.valor * item.quantidade).toFixed(2) }} (Quantidade: {{ item.quantidade }})
                        <Button icon="pi pi-trash" class="p-button-text p-button-danger" @click="removerDoCarrinho(item.id)"></Button>
                    </li>
                </ul>
                <div class="venda">
                    <div>Total: R$ {{ totalCarrinho.toFixed(2) }}</div>
                    <Button icon="pi pi-shopping-cart" label="Realizar venda" @click="realizarVenda"></Button>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
* {
    list-style: none;
}
.venda {
    display: flex;
    justify-content: space-between;
}
.card {
}
.card-product {
    height: 150px;
    padding: 2rem;
    background-color: rgb(100, 234, 174);
    cursor: pointer;
    margin-right: 0.5rem;
    margin-bottom: 0.5rem;
    box-sizing: border-box;
    border-radius: 5px;
}
.card-product:active {
    background-color: rgb(157, 243, 204);
}
@media (min-width: 768px) {
    .card-product {
        flex: 1 1 calc(33.33% - 1rem); /* Adjust the width to account for the margin */
        max-width: calc(33.33% - 1rem); /* Ensure the max-width is correct */
    }
}
</style>
