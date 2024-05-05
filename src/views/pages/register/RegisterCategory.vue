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
                category: ''
            }
        };
    },
    methods: {
        handleSubmitForm() {
            const token = localStorage.getItem('authorization');
            // Configurar o token no cabeçalho 'Authorization'
            axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
            axios
                .post('http://localhost:8000/cadastrar/categoria', this.dados)
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
        }
    }
};
</script>

<template>
    <div class="grid">
        <div class="col-12 md:col-6">
            <div class="card">
                <form v-on:submit.prevent="handleSubmitForm()" action="" method="post">
                    <h5>Cadastro de Categoria</h5>
                    <div class="formgroup-inline">
                        <div class="field">
                            <label for="category" class="p-sr-only">Nome da Categoria</label>
                            <InputText id="category" type="text" placeholder="Categoria" v-model="dados.category" />
                        </div>
                        <Button label="cadastrar" type="Submit"></Button>
                    </div>
                </form>
                <transition-group name="p-message" tag="div">
                    <Message v-for="msg of message" :severity="msg.severity" :key="msg.content">{{ msg.content }}</Message>
                </transition-group>
            </div>
        </div>
    </div>
</template>
