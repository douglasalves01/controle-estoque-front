<script setup>
import { ref } from 'vue';

import AppMenuItem from './AppMenuItem.vue';

const model = ref([
    {
        label: 'Home',
        items: [{ label: 'Dashboard', icon: 'pi pi-fw pi-home', to: '/dashboard' }]
    },
    {
        label: 'Cadastro',
        items: [
            { label: 'Categoria', to: '/create/category' },
            { label: 'Produto', to: '/create/product' },
            { label: 'Fornecedor', to: '/create/supplier' }
        ]
    },
    {
        label: 'Visualizar',
        items: [
            { label: 'Categoria', to: '/visualize/category' },
            { label: 'Produto', to: '/visualize/product' },
            { label: 'Fornecedor', to: '/visualize/supplier' },
            { label: 'Vendas', to: '/visualize/sale' },
            { label: 'Estoque', to: '/visualize/stock' },
            { label: 'Movimentações', to: '/visualize/moviment' }
        ]
    },
    {
        label: 'Financeiro',
        items: [{ label: 'Lucro por venda', to: '/finance/sales' }]
    },
    {
        label: 'Vendas',
        items: [{ label: 'Realizar venda', to: '/sales' }]
    },
    {
        label: 'Administrativo',
        items: [{ label: 'Configurações', to: '/config' }]
    }
]);
</script>

<template>
    <ul class="layout-menu">
        <template v-for="(item, i) in model" :key="item">
            <app-menu-item v-if="!item.separator" :item="item" :index="i"></app-menu-item>
            <li v-if="item.separator" class="menu-separator"></li>
        </template>
    </ul>
</template>

<style lang="scss" scoped></style>
