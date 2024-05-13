<script setup>
import { useLayout } from '@/layout/composables/layout';

useLayout();
</script>
<script>
import axios from 'axios';
import { ref } from 'vue';
const message = ref([]);
const count = ref(0);
const formatarCNPJ = (cnpj) => {
    return cnpj.replace(/[^\d]+/g, '');
};
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
            telefones: [''],
            dados: {
                cnpj: '',
                razao_social: '',
                nome_fantasia: '',
                endereco: '',
                telefone: []
            }
        };
    },
    methods: {
        addPhoneNumber() {
            this.telefones.push('');
        },
        formatarTelefone(telefone) {
            // Remova todos os caracteres não numéricos do telefone
            const numeros = telefone.replace(/\D/g, '');
            // Formatar o número como (XX) XXXXX-XXXX
            return `(${numeros.slice(0, 2)}) ${numeros.slice(2, 7)}-${numeros.slice(7)}`;
        },
        handleSubmitForm() {
            const token = localStorage.getItem('authorization');
            // Configurar o token no cabeçalho 'Authorization'
            axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
            this.dados.cnpj = formatarCNPJ(this.dados.cnpj);
            this.dados.telefone = this.telefones.map((telefone) => this.formatarTelefone(telefone));

            axios
                .post('http://localhost:8000/cadastrar/fornecedor', this.dados)
                .then((response) => {
                    console.log(response);
                    if (response.status === 200) {
                        addMessage('success', response.data.message);
                    }
                })
                .catch((error) => {
                    if (error.response.data) {
                        console.log(error.response);
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
                    <h5>Cadastro de produto</h5>
                    <div class="formgroup-inline">
                        <div class="field">
                            <label for="product" class="p-sr-only">Produto</label>
                            <InputText id="product" type="text" placeholder="Produto" v-model="dados.produto" />
                        </div>
                        <div class="field">
                            <label for="price" class="p-sr-only">Valor</label>
                            <InputText id="price" type="text" placeholder="Valor" v-model="dados.razao_social" />
                        </div>
                        <div class="field">
                            <label for="cnpj" class="p-sr-only">CNPJ</label>
                            <InputMask id="cnpj" v-model="dados.cnpj" mask="99.999.999/9999-99" placeholder="CNPJ" />
                        </div>
                        <div class="field">
                            <label for="endereco" class="p-sr-only">Endereço</label>
                            <InputText id="endereco" type="text" placeholder="Endereço" v-model="dados.endereco" />
                        </div>
                        <div class="field" v-for="(telefone, index) in telefones" :key="index">
                            <label class="p-sr-only" :for="'phone' + index">Telefone {{ index + 1 }}</label>
                            <div class="input-with-button">
                                <InputMask v-model="telefones[index]" :id="'phone' + index" mask="(99) 99999-9999" placeholder="Telefone"></InputMask>
                            </div>
                        </div>
                        <Button @click="addPhoneNumber">Add</Button>
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
