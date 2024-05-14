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
    { label: 'entrada', value: 'entrada' },
    { label: 'saida', value: 'saida' }
]);
let id_product = '';

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .get('http://localhost:8000/relatorio/movimentacoes')
        .then((response) => {
            if (response.status === 200) {
                stocks.value = response.data;
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

const saveProduct = (addStock, description, typeMoviment, unitCost) => {
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .post(`http://localhost:8000/controle-estoque/movimentacao/${id_product}`, { addStock, description, unitCost, typeMoviment })
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

    //stockDialog.value = false;
    //stock.value = {};
};

const editProduct = (editProduct) => {
    id_product = editProduct;

    stockDialog.value = true;
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
                    currentPageReportTemplate="Mostrando {first} à {last} de {totalRecords} registros de movimentações do estoque"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Gerenciar Movimentações</h5>
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
                    <Column field="1" header="Tipo movimento" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Tipo transação</span>
                            {{ slotProps.data[1] }}
                        </template>
                    </Column>
                    <Column field="1" header="Data" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Data</span>
                            {{ slotProps.data[2] }}
                        </template>
                    </Column>
                    <Column field="1" header="Quantidade" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Quantidade</span>
                            {{ slotProps.data[3] }}
                        </template>
                    </Column>
                    <Column field="1" header="Custo" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Custo</span>
                            {{ slotProps.data[4] }}
                        </template>
                    </Column>
                    <Column field="1" header="Motivo movimento" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Motivo movimento</span>
                            {{ slotProps.data[5] }}
                        </template>
                    </Column>
                    <Column field="1" header="Produto" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Produto</span>
                            {{ slotProps.data[6] }}
                        </template>
                    </Column>
                    <Column field="1" header="Responsável" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Responsável</span>
                            {{ slotProps.data[7] }}
                        </template>
                    </Column>

                    <Column headerStyle="min-width:10rem;">
                        <template #body="slotProps">
                            <Button icon="pi pi-plus" class="mr-2" severity="success" rounded @click="editProduct(slotProps.data[8])" />
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="stockDialog" :style="{ width: '450px' }" header="Inserir Movimento para produto" :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="stock">Quantidade</label>
                        <InputText id="stock" v-model.trim="stock.quantidade" required="true" autofocus :invalid="submitted && !stock.quantidade" />
                        <small class="p-invalid" v-if="!stock.quantidade">Quantidade é obrigatória</small>
                    </div>
                    <div class="field">
                        <label for="stock">Motivo da transação</label>
                        <InputText id="stock" v-model.trim="stock.motivo_transacao" required="true" autofocus :invalid="submitted && !stock.motivo_transacao" />
                        <small class="p-invalid" v-if="!stock.motivo_transacao">Motivo é obrigatório</small>
                    </div>
                    <div class="field">
                        <label for="inventoryStatus" class="mb-3">Tipo de mevimentação</label>
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
                        <small class="p-invalid" v-if="!stock.inventoryStatus">Tipo de movimentação é obrigatório</small>
                    </div>
                    <div class="field" v-if="stock.inventoryStatus && stock.inventoryStatus.value === 'entrada'">
                        <label for="stock">Custo unitário</label>
                        <InputText id="stock" v-model.trim="stock.custo_unitario" required="true" autofocus :invalid="submitted && !stock.custo_unitario" />
                        <small class="p-invalid" v-if="!stock.custo_unitario">Custo unitário é obrigatório</small>
                    </div>
                    <template #footer>
                        <Button label="Cancel" icon="pi pi-times" text="" @click="hideDialog" />
                        <Button label="Save" icon="pi pi-check" text="" @click="saveProduct(stock.quantidade, stock.motivo_transacao, stock.inventoryStatus.value, stock.custo_unitario)" />
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
