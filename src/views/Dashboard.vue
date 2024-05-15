<script setup>
import { onMounted, ref, watch } from 'vue';
import { ProductService } from '@/service/ProductService';
import { useLayout } from '@/layout/composables/layout';
import axios from 'axios';
const { isDarkTheme } = useLayout();

let qtdeSales = ref(0);
let qtdeSuppliers = ref(0);
let qtdeProducts = ref(0);
let qtdeCategories = ref(0);
let productMinimum = ref([]);
const QtdeProductMinimum = () => {
    const token = localStorage.getItem('authorization');
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;

    return new Promise((resolve, reject) => {
        axios
            .get('http://localhost:8000/notificacao/products')
            .then((response) => {
                resolve(response.data); // Resolvendo a promessa com o valor obtido
            })
            .catch((error) => {
                console.error('Erro ao buscar a produtos com estoque minimos:', error);
                reject(error); // Rejeitando a promessa em caso de erro
            });
    });
};
const QtdeSales = () => {
    const token = localStorage.getItem('authorization');
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;

    return new Promise((resolve, reject) => {
        axios
            .get('http://localhost:8000/quantidade/sales')
            .then((response) => {
                resolve(response.data[0][0]); // Resolvendo a promessa com o valor obtido
            })
            .catch((error) => {
                console.error('Erro ao buscar a quantidade de vendas:', error);
                reject(error); // Rejeitando a promessa em caso de erro
            });
    });
};
const QtdeSuppliers = () => {
    const token = localStorage.getItem('authorization');
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;

    return new Promise((resolve, reject) => {
        axios
            .get('http://localhost:8000/quantidade/suppliers')
            .then((response) => {
                resolve(response.data[0][0]); // Resolvendo a promessa com o valor obtido
            })
            .catch((error) => {
                console.error('Erro ao buscar a quantidade de fornecedores:', error);
                reject(error); // Rejeitando a promessa em caso de erro
            });
    });
};
const QtdeProducts = () => {
    const token = localStorage.getItem('authorization');
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;

    return new Promise((resolve, reject) => {
        axios
            .get('http://localhost:8000/quantidade/products')
            .then((response) => {
                resolve(response.data[0][0]); // Resolvendo a promessa com o valor obtido
            })
            .catch((error) => {
                console.error('Erro ao buscar a quantidade de produtos:', error);
                reject(error); // Rejeitando a promessa em caso de erro
            });
    });
};
const QtdeCategories = () => {
    const token = localStorage.getItem('authorization');
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;

    return new Promise((resolve, reject) => {
        axios
            .get('http://localhost:8000/quantidade/categories')
            .then((response) => {
                resolve(response.data[0][0]); // Resolvendo a promessa com o valor obtido
            })
            .catch((error) => {
                console.error('Erro ao buscar a quantidade de categorias:', error);
                reject(error); // Rejeitando a promessa em caso de erro
            });
    });
};

const products = ref(null);

const items = ref([
    { label: 'Add New', icon: 'pi pi-fw pi-plus' },
    { label: 'Remove', icon: 'pi pi-fw pi-minus' }
]);
const lineOptions = ref(null);
const productService = new ProductService();

onMounted(async () => {
    productService.getProductsSmall().then((data) => (products.value = data));
    qtdeSales.value = await QtdeSales();
    qtdeCategories.value = await QtdeCategories();
    qtdeSuppliers.value = await QtdeSuppliers();
    qtdeProducts.value = await QtdeProducts();
    productMinimum.value = await QtdeProductMinimum();
});

const applyLightTheme = () => {
    lineOptions.value = {
        plugins: {
            legend: {
                labels: {
                    color: '#495057'
                }
            }
        },
        scales: {
            x: {
                ticks: {
                    color: '#495057'
                },
                grid: {
                    color: '#ebedef'
                }
            },
            y: {
                ticks: {
                    color: '#495057'
                },
                grid: {
                    color: '#ebedef'
                }
            }
        }
    };
};

const applyDarkTheme = () => {
    lineOptions.value = {
        plugins: {
            legend: {
                labels: {
                    color: '#ebedef'
                }
            }
        },
        scales: {
            x: {
                ticks: {
                    color: '#ebedef'
                },
                grid: {
                    color: 'rgba(160, 167, 181, .3)'
                }
            },
            y: {
                ticks: {
                    color: '#ebedef'
                },
                grid: {
                    color: 'rgba(160, 167, 181, .3)'
                }
            }
        }
    };
};

watch(
    isDarkTheme,
    (val) => {
        if (val) {
            applyDarkTheme();
        } else {
            applyLightTheme();
        }
    },
    { immediate: true }
);
</script>

<template>
    <div class="grid">
        <div class="col-12 lg:col-6 xl:col-3" v-if="qtdeCategories !== null">
            <div class="card mb-0">
                <div class="flex justify-content-between mb-3">
                    <div>
                        <span class="block text-500 font-medium mb-3">Categorias</span>
                        <div class="text-900 font-medium text-xl">{{ qtdeCategories }}</div>
                    </div>
                    <div class="flex align-items-center justify-content-center bg-blue-100 border-round" style="width: 2.5rem; height: 2.5rem">
                        <i class="pi pi-shopping-cart text-blue-500 text-xl"></i>
                    </div>
                </div>
            </div>
        </div>
        <div class="col-12 lg:col-6 xl:col-3" v-if="qtdeProducts !== null">
            <div class="card mb-0">
                <div class="flex justify-content-between mb-3">
                    <div>
                        <span class="block text-500 font-medium mb-3">Produtos</span>
                        <div class="text-900 font-medium text-xl">{{ qtdeProducts }}</div>
                    </div>
                    <div class="flex align-items-center justify-content-center bg-blue-100 border-round" style="width: 2.5rem; height: 2.5rem">
                        <i class="pi pi-shopping-cart text-blue-500 text-xl"></i>
                    </div>
                </div>
            </div>
        </div>
        <div class="col-12 lg:col-6 xl:col-3" v-if="qtdeSuppliers !== null">
            <div class="card mb-0">
                <div class="flex justify-content-between mb-3">
                    <div>
                        <span class="block text-500 font-medium mb-3">Forneceores</span>
                        <div class="text-900 font-medium text-xl">{{ qtdeSuppliers }}</div>
                    </div>
                    <div class="flex align-items-center justify-content-center bg-blue-100 border-round" style="width: 2.5rem; height: 2.5rem">
                        <i class="pi pi-shopping-cart text-blue-500 text-xl"></i>
                    </div>
                </div>
            </div>
        </div>
        <div class="col-12 lg:col-6 xl:col-3" v-if="qtdeSales !== null">
            <div class="card mb-0">
                <div class="flex justify-content-between mb-3">
                    <div>
                        <span class="block text-500 font-medium mb-3">Vendas</span>
                        <div class="text-900 font-medium text-xl">{{ qtdeSales }}</div>
                    </div>
                    <div class="flex align-items-center justify-content-center bg-blue-100 border-round" style="width: 2.5rem; height: 2.5rem">
                        <i class="pi pi-shopping-cart text-blue-500 text-xl"></i>
                    </div>
                </div>
            </div>
        </div>

        <div class="col-50 xl:col-12">
            <div class="card">
                <div class="flex align-items-center justify-content-between mb-4">
                    <h5>Notifications</h5>
                    <div>
                        <Button icon="pi pi-ellipsis-v" class="p-button-text p-button-plain p-button-rounded" @click="$refs.menu1.toggle($event)"></Button>
                        <Menu ref="menu1" :popup="true" :model="items"></Menu>
                    </div>
                </div>
                <span class="block text-600 font-medium mb-3">TODAY</span>
                <ul class="p-0 mx-0 mt-0 mb-4 list-none" v-if="productMinimum.lenght !== 0">
                    <li v-for="(product, index) in productMinimum" :key="index" class="flex align-items-center py-2">
                        <div class="w-3rem h-3rem flex align-items-center justify-content-center bg-orange-100 border-circle mr-3 flex-shrink-0">
                            <i class="pi pi-exclamation-triangle text-xl text-orange-500"></i>
                        </div>
                        <div class="flex-grow-1">
                            <span class="text-gray-500 font-medium">O produto </span>
                            <span class="text-blue-500 font-medium">{{ product[1] }}</span> -
                            <span class="text-gray-500">{{ product[2] }} atingiu o estoque <span class="text-red-500 font-medium">mínimo!</span></span>
                        </div>
                    </li>
                </ul>
            </div>
        </div>
    </div>
</template>
