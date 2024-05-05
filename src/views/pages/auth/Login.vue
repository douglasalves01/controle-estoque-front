<script setup>
import { useLayout } from '@/layout/composables/layout';
import { ref, computed } from 'vue';
import AppConfig from '@/layout/AppConfig.vue';
const { layoutConfig } = useLayout();

const logoUrl = computed(() => {
    return `/layout/images/${layoutConfig.darkTheme.value ? 'logo-white' : 'logo-dark'}.svg`;
});
const handleSubmitForm = (dados) => {
    axios
        .post('http://localhost:8000/login', dados)
        .then((response) => {
            if (response.status === 200) {
                const token = response.data;
                localStorage.setItem('authorization', token);
                router.push('/dashboard');
            }
        })
        .catch((error) => {
            if (error.response.data) {
                addMessage('error', error.response.data.detail);
            }
        });
};
</script>
<script>
import router from '../../../router/index.js';
import axios from 'axios';

const message = ref([]);
const count = ref(0);

const addMessage = (type, error) => {
    let newMessage;
    message.value = [];
    if (type === 'success') {
        newMessage = { severity: 'success', detail: 'Success Message', content: 'Message sent', id: count.value++ };
    } else if (type === 'info') {
        newMessage = { severity: 'info', detail: 'Info Message', content: 'PrimeVue rocks', id: count.value++ };
    } else if (type === 'warn') {
        newMessage = { severity: 'warn', detail: 'Warn Message', content: 'There are unsaved changes', id: count.value++ };
    } else if (type === 'error') {
        newMessage = { severity: 'error', detail: 'Error Message', content: error, id: count.value++ };
    }
    message.value.push(newMessage);
};
export default {
    data() {
        return {
            dados: {
                name: '',
                password: ''
            }
        };
    },
    methods: {
        handleSubmitForm() {
            handleSubmitForm(this.dados);
        }
    }
};
</script>
<template>
    <div class="surface-ground flex align-items-center justify-content-center min-h-screen min-w-screen overflow-hidden">
        <div class="flex flex-column align-items-center justify-content-center">
            <img :src="logoUrl" alt="Sakai logo" class="mb-5 w-6rem flex-shrink-0" />
            <div style="border-radius: 56px; padding: 0.3rem; background: linear-gradient(180deg, var(--primary-color) 10%, rgba(33, 150, 243, 0) 30%)">
                <div class="w-full surface-card py-8 px-5 sm:px-8" style="border-radius: 53px">
                    <div class="text-center mb-5">
                        <span class="text-600 font-medium">Faça login para continuar</span>
                    </div>

                    <div>
                        <form v-on:submit.prevent="handleSubmitForm(dados)" action="" method="post">
                            <label for="name" class="block text-900 text-xl font-medium mb-2">Nome</label>
                            <InputText id="name" type="text" placeholder="Nome" class="w-full md:w-30rem mb-5" style="padding: 1rem" v-model="dados.name" />

                            <label for="password" class="block text-900 font-medium text-xl mb-2">Senha</label>
                            <Password id="password" v-model="dados.password" placeholder="Senha" :toggleMask="true" class="w-full mb-3" inputClass="w-full" :inputStyle="{ padding: '1rem' }"></Password>

                            <Button label="Login" class="w-full p-3 text-xl" type="submit"></Button>
                        </form>
                        <transition-group name="p-message" tag="div">
                            <Message v-for="msg of message" :severity="msg.severity" :key="msg.content">{{ msg.content }}</Message>
                        </transition-group>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <AppConfig simple />
</template>

<style scoped>
.pi-eye {
    transform: scale(1.6);
    margin-right: 1rem;
}

.pi-eye-slash {
    transform: scale(1.6);
    margin-right: 1rem;
}
</style>
