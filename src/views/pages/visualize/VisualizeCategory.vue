<script setup>
import { FilterMatchMode } from 'primevue/api';
import { ref, onMounted, onBeforeMount } from 'vue';
// import { ProductService } from '@/service/ProductService';
import { useToast } from 'primevue/usetoast';
import axios from 'axios';
const toast = useToast();

const products = ref([]);
const productDialog = ref(false);
const deleteProductDialog = ref(false);
const deleteProductsDialog = ref(false);
const product = ref({});
const selectedProducts = ref(null);
const dt = ref(null);
const filters = ref({});
const submitted = ref(false);
const statuses = ref([
    { label: 'ATIVO', value: 'ativo' },
    { label: 'INATIVO', value: 'inativo' }
]);
let idCategory = '';
// const productService = new ProductService();

const getBadgeSeverity = (inventoryStatus) => {
    switch (inventoryStatus.toLowerCase()) {
        case 'ativo':
            return 'success';
        case 'inativo':
            return 'danger';
    }
};

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    axios
        .get('http://localhost:8000/categorias')
        .then((response) => {
            if (response.status === 200) {
                products.value = response.data;
            }
        })
        .catch((error) => {
            const message = error.response.data.message;
            console.log(message);
        });
});

const openNew = () => {
    product.value = {};
    submitted.value = false;
    productDialog.value = true;
};

const hideDialog = () => {
    productDialog.value = false;
    submitted.value = false;
};

const saveProduct = (category, status) => {
    submitted.value = true;
    console.log(submitted, 'dsadas');
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .put(`http://localhost:8000/editar/categoria/${idCategory}`, { category, status })
        .then((response) => {
            if (response.status === 200) {
                toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Updated', life: 3000 });
            }
        })
        .catch((error) => {
            const message = error.response.data.message;
            toast.add({ severity: 'error', summary: 'error', detail: message, life: 3000 });
        });

    productDialog.value = false;
    product.value = {};
};

const editProduct = (editProduct) => {
    idCategory = editProduct;
    product.value = { ...editProduct };
    productDialog.value = true;
};

const confirmDeleteProduct = (editProduct) => {
    product.value = editProduct;
    idCategory = editProduct;
    deleteProductDialog.value = true;
};

const deleteProduct = () => {
    /////aqui após confirmar
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .delete(`http://localhost:8000/excluir/categoria/${idCategory}`)
        .then((response) => {
            if (response.status === 200) {
                deleteProductDialog.value = false;
                toast.add({ severity: 'success', summary: 'Categoria deletada', detail: 'Categoria deletado', life: 3000 });
            }
        })
        .catch((error) => {
            const message = error.response.data.message;
            toast.add({ severity: 'error', summary: 'Erro ao deletar categoria', detail: message, life: 3000 });
        });
};

// const findIndexById = (id) => {
//     let index = -1;
//     for (let i = 0; i < products.value.length; i++) {
//         if (products.value[i].id === id) {
//             index = i;
//             break;
//         }
//     }
//     return index;
// };

// const createId = () => {
//     let id = '';
//     const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
//     for (let i = 0; i < 5; i++) {
//         id += chars.charAt(Math.floor(Math.random() * chars.length));
//     }
//     return id;
// };

const exportCSV = () => {
    dt.value.exportCSV();
};

const confirmDeleteSelected = () => {
    deleteProductsDialog.value = true;
};
const deleteSelectedProducts = () => {
    products.value = products.value.filter((val) => !selectedProducts.value.includes(val));
    deleteProductsDialog.value = false;
    selectedProducts.value = null;
    toast.add({ severity: 'success', summary: 'Successful', detail: 'Products Deleted', life: 3000 });
};

const initFilters = () => {
    filters.value = {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS }
    };
};
</script>

<template>
    <div class="grid">
        <div class="col-12">
            <div class="card">
                <Toolbar class="mb-4">
                    <template v-slot:start>
                        <FileUpload mode="basic" accept="image/*" :maxFileSize="1000000" label="Import" chooseLabel="Import" class="mr-2 inline-block" />
                        <Button label="Export" icon="pi pi-upload" severity="help" @click="exportCSV($event)" />
                    </template>
                </Toolbar>

                <DataTable
                    ref="dt"
                    :value="products"
                    v-model:selection="selectedProducts"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Mostrando {first} à {last} de {totalRecords} categorias"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Gerenciar Categorias</h5>
                            <IconField iconPosition="left" class="block mt-2 md:mt-0">
                                <InputIcon class="pi pi-search" />
                                <InputText class="w-full sm:w-auto" v-model="filters['global'].value" placeholder="Search..." />
                            </IconField>
                        </div>
                    </template>

                    <Column field="0" header="Código" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Código</span>
                            {{ slotProps.data[0] }}
                        </template>
                    </Column>
                    <Column field="1" header="Categoria" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Categoria</span>
                            {{ slotProps.data[1] }}
                        </template>
                    </Column>
                    <Column field="2" header="Status" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Status</span>
                            <Tag :severity="getBadgeSeverity(slotProps.data[2])">{{ slotProps.data[2] }}</Tag>
                        </template>
                    </Column>

                    <Column headerStyle="min-width:10rem;">
                        <template #body="slotProps">
                            <Button icon="pi pi-pencil" class="mr-2" severity="success" rounded @click="editProduct(slotProps.data[0])" />
                            <Button icon="pi pi-trash" class="mt-2" severity="warning" rounded @click="confirmDeleteProduct(slotProps.data[0])" />
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="productDialog" :style="{ width: '450px' }" header="Editar Produto " :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="category">Categoria</label>
                        <InputText id="category" v-model.trim="product.name" required="true" autofocus :invalid="submitted && !product.name" />
                        <small class="p-invalid" v-if="!product.name">Categoria é obrigatória</small>
                    </div>
                    <div class="field">
                        <label for="inventoryStatus" class="mb-3">Status</label>
                        <Dropdown id="inventoryStatus" v-model="product.inventoryStatus" :options="statuses" optionLabel="label" placeholder="Selecione o status">
                            <template #value="slotProps">
                                <div v-if="slotProps.value && slotProps.value.value">
                                    <span :class="'product-badge status-' + slotProps.value.value">{{ slotProps.value.label }}</span>
                                </div>
                                <div v-else-if="slotProps.value && !slotProps.value.value">
                                    <span :class="'product-badge status-' + slotProps.value.toLowerCase()">{{ slotProps.value }}</span>
                                </div>
                                <span v-else>
                                    {{ slotProps.placeholder }}
                                </span>
                            </template>
                        </Dropdown>
                        <small class="p-invalid" v-if="!product.inventoryStatus">Status é obrigatório</small>
                    </div>

                    <template #footer>
                        <Button label="Cancel" icon="pi pi-times" text="" @click="hideDialog" />
                        <Button label="Save" icon="pi pi-check" text="" @click="saveProduct(product.name, product.inventoryStatus.value)" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="deleteProductDialog" :style="{ width: '450px' }" header="Confirmação" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="product"
                            >Deseja desativar a categoria<b>{{ product.name }}</b
                            >?</span
                        >
                    </div>
                    <template #footer>
                        <Button label="Não" icon="pi pi-times" text @click="deleteProductDialog = false" />
                        <Button label="Sim" icon="pi pi-check" text @click="deleteProduct" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="deleteProductsDialog" :style="{ width: '450px' }" header="Confirmação" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="product">Quer mesmo excluir as categorias selecionadas?</span>
                    </div>
                    <template #footer>
                        <Button label="Não" icon="pi pi-times" text @click="deleteProductsDialog = false" />
                        <Button label="Sim" icon="pi pi-check" text @click="deleteSelectedProducts" />
                    </template>
                </Dialog>
            </div>
        </div>
    </div>
</template>
