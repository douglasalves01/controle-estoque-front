<script setup>
import { useLayout } from '@/layout/composables/layout';

useLayout();
</script>
<script>
import axios from 'axios';
import { ref } from 'vue';
const message = ref([]);
const count = ref(0);

const addMessage = (type, info) => {
    let newMessage;
    message.value = [];
    if (type === 'success') {
        newMessage = { severity: 'success', detail: 'Success Message', content: info, id: count.value++ };
    } else if (type === 'info') {
        newMessage = { severity: 'info', detail: 'Info Message', content: info, id: count.value++ };
    } else if (type === 'warn') {
        newMessage = { severity: 'warn', detail: 'Warn Message', content: info, id: count.value++ };
    } else if (type === 'error') {
        newMessage = { severity: 'error', detail: 'Error Message', content: info, id: count.value++ };
    }
    message.value.push(newMessage);
};
export default {
    data() {
        return {
            dados: {
                product: '',
                price: '',
                unit_measure: '',
                currentStock: '',
                minimumStock: '',
                unitCost: '',
                location: '',
                id_supplier: '',
                id_category: '',
                description: '',
                icms: ''
            },
            statuses: [],
            categories: [],
            configs: {
                margem_lucro: '',
                comissao_venda: '',
                custo_fixo: '',
                imposto_venda: ''
            }
        };
    },
    methods: {
        handleSubmitForm() {
            const token = localStorage.getItem('authorization');
            // Configurar o token no cabeçalho 'Authorization'
            axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;

            axios
                .post('http://localhost:8000/cadastrar/produto', {
                    product: this.dados.product,
                    price: this.dados.price,
                    unit_measure: this.dados.unit_measure,
                    currentStock: this.dados.currentStock,
                    minimumStock: this.dados.minimumStock,
                    unitCost: this.dados.unitCost,
                    location: this.dados.location,
                    id_supplier: this.dados.id_supplier.value.toString(),
                    id_category: this.dados.id_category.value.toString(),
                    description: this.dados.description.toUpperCase(),
                    icms: this.dados.icms
                })
                .then((response) => {
                    if (response.status === 200) {
                        addMessage('success', response.data.message);
                    }
                })
                .catch((error) => {
                    if (error.response.data) {
                        addMessage('error', error.response.data.detail);
                    }
                });
        },
        fetchSupplier() {
            const token = localStorage.getItem('authorization');
            // Configurar o token no cabeçalho 'Authorization'
            axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
            axios
                .get('http://localhost:8000/fornecedores-active')
                .then((response) => {
                    // Mapeie os dados dos fornecedores para o formato necessário pelo Dropdown
                    this.statuses = response.data.map((supplier) => ({
                        label: supplier[2],
                        value: supplier[0]
                    }));
                })
                .catch((error) => {
                    console.error('Erro ao buscar fornecedores:', error);
                });
        },
        fetchCategory() {
            const token = localStorage.getItem('authorization');
            // Configurar o token no cabeçalho 'Authorization'
            axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
            axios
                .get('http://localhost:8000/categorias-active')
                .then((response) => {
                    // Mapeie os dados dos fornecedores para o formato necessário pelo Dropdown
                    this.categories = response.data.map((category) => ({
                        label: category[1],
                        value: category[0]
                    }));
                })
                .catch((error) => {
                    console.error('Erro ao buscar categorias:', error);
                });
        },
        fetchConfigs() {
            const token = localStorage.getItem('authorization');
            // Configurar o token no cabeçalho 'Authorization'
            axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
            axios
                .get('http://localhost:8000/configs')
                .then((response) => {
                    this.configs.margem_lucro = response.data[0][1];
                    this.configs.comissao_venda = response.data[0][2];
                    this.configs.custo_fixo = response.data[0][3];
                })
                .catch((error) => {
                    console.error('Erro ao buscar categorias:', error);
                });
        },
        calculaPrecoVenda(custo_produto, icms) {
            let fatores = (this.configs.custo_fixo + this.configs.comissao_venda + icms + this.configs.margem_lucro) / 100;
            let preco_sugerido = (custo_produto / (1 - fatores)).toFixed(2);

            addMessage('info', `Preço de venda sugerido de R$ ${preco_sugerido}`);
        }
    },
    watch: {
        'dados.unitCost': function (custo_produto) {
            let icms = parseFloat(this.dados.icms);
            this.calculaPrecoVenda(custo_produto, icms);
        },
        'dados.icms': function (icms) {
            let custo_produto = parseFloat(this.dados.unitCost);
            if (!isNaN(custo_produto)) {
                this.calculaPrecoVenda(custo_produto, parseFloat(icms));
            }
        }
    },
    mounted() {
        this.fetchSupplier();
        this.fetchCategory();
        this.fetchConfigs();
    }
};
</script>
<template>
    <div class="grid">
        <div class="col-12 md:col-6">
            <div class="card">
                <form v-on:submit.prevent="handleSubmitForm()" action="" method="post">
                    <h5>Cadastro de produto</h5>
                    <div class="formgroup-inline">
                        <div class="field">
                            <label for="product" class="p-sr-only">Produto</label>
                            <InputText id="product" type="text" placeholder="Nome produto" v-model="dados.product" />
                        </div>
                        <div class="field">
                            <label for="price" class="p-sr-only">Valor</label>
                            <InputText id="price" type="number" step="0.01" placeholder="Valor" v-model="dados.price" />
                        </div>
                        <div class="field">
                            <label for="description" class="p-sr-only">Descrição</label>
                            <InputText id="description" type="text" placeholder="Descrição" v-model="dados.description" />
                        </div>
                        <div class="field">
                            <label for="unit_measure" class="p-sr-only">Unidade de medida</label>
                            <InputText id="unit_measure" type="text" placeholder="Unidade de medida" v-model="dados.unit_measure" />
                        </div>
                        <div class="field">
                            <label for="icms" class="p-sr-only">ICMS</label>
                            <InputText id="icms" type="number" step="0.01" placeholder="ICMS" v-model="dados.icms" />
                        </div>
                        <div class="field">
                            <Dropdown id="supplier" v-model="dados.id_supplier" :options="statuses" optionLabel="label" placeholder="Selecione um fornecedor">
                                <template #value="suppliers">
                                    <div class="width-dropdown">
                                        <!-- Adicione uma classe para o estilo específico do dropdown -->
                                        <div v-if="suppliers.value && suppliers.value.label">
                                            <span :class="'supplier-badge status-' + suppliers.value.value">{{ suppliers.value.label }}</span>
                                        </div>
                                        <div v-else-if="suppliers.value && !suppliers.value.value">
                                            <span :class="'supplier-badge status-' + suppliers.value.value.toLowerCase()">{{ suppliers.value }}</span>
                                        </div>
                                        <span v-else>
                                            {{ suppliers.placeholder }}
                                        </span>
                                    </div>
                                </template>
                            </Dropdown>
                        </div>
                        <div class="field">
                            <Dropdown id="category" v-model="dados.id_category" :options="categories" optionLabel="label" placeholder="Selecione uma categoria">
                                <template #value="categories">
                                    <div class="width-dropdown">
                                        <div v-if="categories.value && categories.value.label">
                                            <span :class="'supplier-badge status-' + categories.value.value">{{ categories.value.label }}</span>
                                        </div>
                                        <div v-else-if="categories.value && !categories.value.value">
                                            <span :class="'supplier-badge status-' + categories.value.value.toLowerCase()">{{ categories.value }}</span>
                                        </div>
                                        <span v-else>
                                            {{ categories.placeholder }}
                                        </span>
                                    </div>
                                </template>
                            </Dropdown>
                        </div>
                        <div class="field">
                            <label for="current_stock" class="p-sr-only">Estoque atual</label>
                            <InputText id="current_stock" type="text" placeholder="Estoque Atual" v-model="dados.currentStock" />
                        </div>
                        <div class="field">
                            <label for="minimun_stock" class="p-sr-only">Estoque mínimo</label>
                            <InputText id="minimun_stock" type="text" placeholder="Estoque mínimo" v-model="dados.minimumStock" />
                        </div>
                        <div class="field">
                            <label for="unit_cost" class="p-sr-only">Custo Unitário</label>
                            <InputText id="unit_cost" type="number" step="0.01" placeholder="Custo unitário" v-model="dados.unitCost" />
                        </div>
                        <div class="field">
                            <label for="location" class="p-sr-only">Localização estoque</label>
                            <InputText id="location" type="text" placeholder="Localização no estoque" v-model="dados.location" />
                        </div>
                    </div>
                    <Button label="cadastrar" type="Submit"></Button>
                </form>
                <transition-group name="p-message" tag="div">
                    <Message v-for="msg of message" :severity="msg.severity" :key="msg.content">{{ msg.content }}</Message>
                </transition-group>
            </div>
        </div>
    </div>
</template>
<style>
.width-dropdown {
    min-width: 165px; /* Defina a largura mínima desejada para o dropdown */
}
</style>
