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
                margem_lucro: '',
                comissao_venda: '',
                custo_fixo: ''
            }
        };
    },
    methods: {
        handleSubmitForm() {
            const token = localStorage.getItem('authorization');
            // Configurar o token no cabeçalho 'Authorization'
            axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
            axios
                .put('http://localhost:8000/config/1', this.dados)
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
        fetchConfigData() {
            const token = localStorage.getItem('authorization');
            // Configurar o token no cabeçalho 'Authorization'
            axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
            axios
                .get('http://localhost:8000/configs')
                .then((response) => {
                    this.dados.margem_lucro = response.data[0][1].toString();
                    this.dados.comissao_venda = response.data[0][2].toString();
                    this.dados.custo_fixo = response.data[0][3].toString();
                })
                .catch((error) => {
                    console.error('Erro ao buscar configurações:', error);
                });
        }
    },
    mounted() {
        this.fetchConfigData();
    }
};
</script>

<template>
    <div class="grid">
        <div class="col-12 md:col-6">
            <div class="card">
                <form v-on:submit.prevent="handleSubmitForm()" action="" method="post">
                    <h5>Configurações</h5>
                    <div class="col-12">
                        <div class="p-fluid formgrid grid">
                            <div class="field col-12 md:col-6">
                                <label for="margem_lucro">Margem de lucro</label>
                                <InputText id="margem_lucro" type="text" v-model="dados.margem_lucro" />
                            </div>
                            <div class="field col-12 md:col-6">
                                <label for="comissao_venda">Comissão de venda</label>
                                <InputText id="comissao_venda" type="text" v-model="dados.comissao_venda" />
                            </div>
                            <div class="field col-12 md:col-6">
                                <label for="custo_fixo">Custo fixo</label>
                                <InputText id="custo_fixo" type="text" v-model="dados.custo_fixo" />
                            </div>
                        </div>
                        <Button label="Salvar" type="Submit"></Button>
                    </div>
                </form>
                <transition-group name="p-message" tag="div">
                    <Message v-for="msg of message" :severity="msg.severity" :key="msg.content">{{ msg.content }}</Message>
                </transition-group>
            </div>
        </div>
    </div>
</template>
