<script setup>
import { FilterMatchMode } from 'primevue/api';
import { ref, onMounted, onBeforeMount } from 'vue';
// import { ProductService } from '@/service/ProductService';
import { useToast } from 'primevue/usetoast';
import axios from 'axios';
const toast = useToast();

const stocks = ref([]);
const stockDialog = ref(false);
const deleteProductDialog = ref(false);
const deleteProductsDialog = ref(false);
const stock = ref({});
const selectedProducts = ref(null);
const dt = ref(null);
const filters = ref({});
const submitted = ref(false);
const statuses = ref([
    { label: 'ATIVO', value: 'ativo' },
    { label: 'INATIVO', value: 'inativo' }
]);
let idCategory = '';

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .get('http://localhost:8000/relatorio/estoques')
        .then((response) => {
            if (response.status === 200) {
                stocks.value = response.data;
                console.log(stocks.value);
            }
        })
        .catch((error) => {
            const message = error.response.data.message;
            console.log(message);
        });
});

const hideDialog = () => {
    stockDialog.value = false;
    submitted.value = false;
};

const saveProduct = (stock, status) => {
    submitted.value = true;
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .put(`http://localhost:8000/editar/categoria/${idCategory}`, { stock, status })
        .then((response) => {
            if (response.status === 200) {
                toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Updated', life: 3000 });
                window.location.reload();
            }
        })
        .catch((error) => {
            const message = error.response.data.message;
            toast.add({ severity: 'error', summary: 'error', detail: message, life: 3000 });
        });

    stockDialog.value = false;
    stock.value = {};
};

const editProduct = (editProduct) => {
    idCategory = editProduct;
    stock.value = { ...editProduct };
    stockDialog.value = true;
};

const confirmDeleteProduct = (editProduct) => {
    stock.value = editProduct;
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
                window.location.reload();
            }
        })
        .catch((error) => {
            const message = error.response.data.message;
            toast.add({ severity: 'error', summary: 'Erro ao deletar categoria', detail: message, life: 3000 });
        });
};

const exportCSV = () => {
    dt.value.exportCSV();
};

const deleteSelectedProducts = () => {
    stocks.value = stocks.value.filter((val) => !selectedProducts.value.includes(val));
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
                    :value="stocks"
                    v-model:selection="selectedProducts"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Mostrando {first} à {last} de {totalRecords} registros do estoque"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Gerenciar Estoque</h5>
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
                    <Column field="1" header="Estoque" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Estoque</span>
                            {{ slotProps.data[1] }}
                        </template>
                    </Column>
                    <Column field="1" header="Estoque mínimo" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Estoque mínimo</span>
                            {{ slotProps.data[2] }}
                        </template>
                    </Column>
                    <Column field="1" header="Localização" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Localização</span>
                            {{ slotProps.data[3] }}
                        </template>
                    </Column>
                    <Column field="1" header="Data entrada" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Data entrada</span>
                            {{ slotProps.data[4] }}
                        </template>
                    </Column>
                    <Column field="1" header="Produto" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Produto</span>
                            {{ slotProps.data[5] }}
                        </template>
                    </Column>
                    <Column field="1" header="Custo médio" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Custo médio</span>
                            {{ slotProps.data[6] }}
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="stockDialog" :style="{ width: '450px' }" header="Editar Produto " :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="stock">Categoria</label>
                        <InputText id="stock" v-model.trim="stock.name" required="true" autofocus :invalid="submitted && !stock.name" />
                        <small class="p-invalid" v-if="!stock.name">Categoria é obrigatória</small>
                    </div>
                    <div class="field">
                        <label for="inventoryStatus" class="mb-3">Status</label>
                        <Dropdown id="inventoryStatus" v-model="stock.inventoryStatus" :options="statuses" optionLabel="label" placeholder="Selecione o status">
                            <template #value="slotProps">
                                <div v-if="slotProps.value && slotProps.value.value">
                                    <span :class="'stock-badge status-' + slotProps.value.value">{{ slotProps.value.label }}</span>
                                </div>
                                <div v-else-if="slotProps.value && !slotProps.value.value">
                                    <span :class="'stock-badge status-' + slotProps.value.toLowerCase()">{{ slotProps.value }}</span>
                                </div>
                                <span v-else>
                                    {{ slotProps.placeholder }}
                                </span>
                            </template>
                        </Dropdown>
                        <small class="p-invalid" v-if="!stock.inventoryStatus">Status é obrigatório</small>
                    </div>

                    <template #footer>
                        <Button label="Cancel" icon="pi pi-times" text="" @click="hideDialog" />
                        <Button label="Save" icon="pi pi-check" text="" @click="saveProduct(stock.name, stock.inventoryStatus.value)" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="deleteProductDialog" :style="{ width: '450px' }" header="Confirmação" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="stock"
                            >Deseja desativar a categoria<b>{{ stock.name }}</b
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
                        <span v-if="stock">Quer mesmo excluir as categorias selecionadas?</span>
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
