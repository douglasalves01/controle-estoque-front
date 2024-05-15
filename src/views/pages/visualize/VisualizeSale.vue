<script setup>
import { FilterMatchMode } from 'primevue/api';
import { ref, onMounted, onBeforeMount } from 'vue';
// import { ProductService } from '@/service/ProductService';
import { useToast } from 'primevue/usetoast';
import axios from 'axios';
const toast = useToast();

const sales = ref([]);
const saleDialog = ref(false);
const deleteProductDialog = ref(false);
const deleteProductsDialog = ref(false);
const sale = ref({});
const selectedProducts = ref(null);
const dt = ref(null);
const filters = ref({});
const submitted = ref(false);
const statuses = ref([
    { label: 'entrada', value: 'entrada' },
    { label: 'saida', value: 'saida' }
]);

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .get('http://localhost:8000/relatorio/sales')
        .then((response) => {
            if (response.status === 200) {
                sales.value = response.data;
            }
        })
        .catch((error) => {
            const message = error.response.data.message;
            console.log(message);
        });
});

const hideDialog = () => {
    saleDialog.value = false;
    submitted.value = false;
};

const exportCSV = () => {
    dt.value.exportCSV();
};

const deleteSelectedProducts = () => {
    sales.value = sales.value.filter((val) => !selectedProducts.value.includes(val));
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
                    :value="sales"
                    v-model:selection="selectedProducts"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Mostrando {first} à {last} de {totalRecords} registros de vendas"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Gerenciar Vendas</h5>
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
                    <Column field="1" header="Quantidade" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Quantidade</span>
                            {{ slotProps.data[1] }}
                        </template>
                    </Column>
                    <Column field="1" header="Valor" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Valor</span>
                            {{ slotProps.data[2] }}
                        </template>
                    </Column>
                    <Column field="1" header="Total" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Total</span>
                            {{ slotProps.data[3] }}
                        </template>
                    </Column>
                    <Column field="1" header="Data" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Data</span>
                            {{ slotProps.data[4] }}
                        </template>
                    </Column>
                    <Column field="1" header="Código venda" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Código venda</span>
                            {{ slotProps.data[5] }}
                        </template>
                    </Column>
                    <Column field="1" header="Responsável" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Responsável</span>
                            {{ slotProps.data[6] }}
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="saleDialog" :style="{ width: '450px' }" header="Inserir Movimento para produto" :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="sale">Quantidade</label>
                        <InputText id="sale" v-model.trim="sale.quantidade" required="true" autofocus :invalid="submitted && !sale.quantidade" />
                        <small class="p-invalid" v-if="!sale.quantidade">Quantidade é obrigatória</small>
                    </div>
                    <div class="field">
                        <label for="sale">Motivo da transação</label>
                        <InputText id="sale" v-model.trim="sale.motivo_transacao" required="true" autofocus :invalid="submitted && !sale.motivo_transacao" />
                        <small class="p-invalid" v-if="!sale.motivo_transacao">Motivo é obrigatório</small>
                    </div>
                    <div class="field">
                        <label for="inventoryStatus" class="mb-3">Tipo de mevimentação</label>
                        <Dropdown id="inventoryStatus" v-model="sale.inventoryStatus" :options="statuses" optionLabel="label" placeholder="Selecione o status">
                            <template #value="slotProps">
                                <div v-if="slotProps.value && slotProps.value.value">
                                    <span :class="'sale-badge status-' + slotProps.value.value">{{ slotProps.value.label }}</span>
                                </div>
                                <div v-else-if="slotProps.value && !slotProps.value.value">
                                    <span :class="'sale-badge status-' + slotProps.value.toLowerCase()">{{ slotProps.value }}</span>
                                </div>
                                <span v-else>
                                    {{ slotProps.placeholder }}
                                </span>
                            </template>
                        </Dropdown>
                        <small class="p-invalid" v-if="!sale.inventoryStatus">Tipo de movimentação é obrigatório</small>
                    </div>
                    <div class="field" v-if="sale.inventoryStatus && sale.inventoryStatus.value === 'entrada'">
                        <label for="sale">Custo unitário</label>
                        <InputText id="sale" v-model.trim="sale.custo_unitario" required="true" autofocus :invalid="submitted && !sale.custo_unitario" />
                        <small class="p-invalid" v-if="!sale.custo_unitario">Custo unitário é obrigatório</small>
                    </div>
                    <template #footer>
                        <Button label="Cancel" icon="pi pi-times" text="" @click="hideDialog" />
                        <Button label="Save" icon="pi pi-check" text="" @click="saveProduct(sale.quantidade, sale.motivo_transacao, sale.inventoryStatus.value, sale.custo_unitario)" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="deleteProductDialog" :style="{ width: '450px' }" header="Confirmação" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="sale"
                            >Deseja desativar a categoria<b>{{ sale.name }}</b
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
                        <span v-if="sale">Quer mesmo excluir as categorias selecionadas?</span>
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
