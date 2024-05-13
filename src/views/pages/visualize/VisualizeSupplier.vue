<script setup>
import { FilterMatchMode } from 'primevue/api';
import { ref, onMounted, onBeforeMount } from 'vue';
// import { ProductService } from '@/service/ProductService';
import { useToast } from 'primevue/usetoast';
import axios from 'axios';
const toast = useToast();

const suppliers = ref([]);
const supplierDialog = ref(false);
const deleteProductDialog = ref(false);
const deleteProductsDialog = ref(false);
const supplier = ref({});
const selectedProducts = ref(null);
const dt = ref(null);
const filters = ref({});
const submitted = ref(false);
const statuses = ref([
    { label: 'ATIVO', value: 'ativo' },
    { label: 'INATIVO', value: 'inativo' }
]);
let idSupplier = '';

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
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .get('http://localhost:8000/fornecedores')
        .then((response) => {
            if (response.status === 200) {
                suppliers.value = response.data;
            }
        })
        .catch((error) => {
            const message = error.response.data.message;
            console.log(message);
        });
});

const hideDialog = () => {
    supplierDialog.value = false;
    submitted.value = false;
};

const saveProduct = (cnpj, razao_social, nome_fantasia, endereco, telefone, status) => {
    telefone = ['17996666666'];
    submitted.value = true;
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .put(`http://localhost:8000/editar/fornecedor/${idSupplier}`, { cnpj, razao_social, nome_fantasia, endereco, telefone, status })
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

    supplierDialog.value = false;
    supplier.value = {};
};

const editProduct = (editProduct) => {
    idSupplier = editProduct;
    supplier.value = { ...editProduct };
    supplierDialog.value = true;
};

const confirmDeleteProduct = (editProduct) => {
    supplier.value = editProduct;
    idSupplier = editProduct;
    deleteProductDialog.value = true;
};

const deleteProduct = () => {
    /////aqui após confirmar
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .delete(`http://localhost:8000/excluir/fornecedor/${idSupplier}`)
        .then((response) => {
            if (response.status === 200) {
                deleteProductDialog.value = false;
                toast.add({ severity: 'success', summary: 'Fornecedor deletado', detail: 'Fornecedor deletado', life: 3000 });
                window.location.reload();
            }
        })
        .catch((error) => {
            const message = error.response.data.message;
            toast.add({ severity: 'error', summary: 'Erro ao deletar fornecedor', detail: message, life: 3000 });
        });
};

const exportCSV = () => {
    dt.value.exportCSV();
};

const deleteSelectedProducts = () => {
    suppliers.value = suppliers.value.filter((val) => !selectedProducts.value.includes(val));
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
                    :value="suppliers"
                    v-model:selection="selectedProducts"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Mostrando {first} à {last} de {totalRecords} fornecedores"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Gerenciar Fornecedores</h5>
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
                    <Column field="1" header="CNPJ" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">CNPJ</span>
                            {{ slotProps.data[1] }}
                        </template>
                    </Column>
                    <Column field="2" header="Razão Social" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Razão Social</span>
                            {{ slotProps.data[2] }}
                        </template>
                    </Column>
                    <Column field="2" header="Nome Fantasia" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Nome Fantasia</span>
                            {{ slotProps.data[3] }}
                        </template>
                    </Column>
                    <Column field="2" header="Endereço" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Endereço</span>
                            {{ slotProps.data[4] }}
                        </template>
                    </Column>
                    <Column field="2" header="Status" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Status</span>
                            <Tag :severity="getBadgeSeverity(slotProps.data[5])">{{ slotProps.data[5] }}</Tag>
                        </template>
                    </Column>

                    <Column headerStyle="min-width:10rem;">
                        <template #body="slotProps">
                            <Button icon="pi pi-pencil" class="mr-2" severity="success" rounded @click="editProduct(slotProps.data[0])" />
                            <Button icon="pi pi-trash" class="mt-2" severity="warning" rounded @click="confirmDeleteProduct(slotProps.data[0])" />
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="supplierDialog" :style="{ width: '450px' }" header="Editar Fornecedor " :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="cnpj">CNPJ</label>
                        <InputText id="supplier" v-model.trim="supplier.cnpj" required="true" autofocus :invalid="submitted && !supplier.cnpj" />
                        <small class="p-invalid" v-if="!supplier.cnpj">CNPJ é obrigatório</small>
                    </div>
                    <div class="field">
                        <label for="razao_social">Razão Social</label>
                        <InputText id="supplier" v-model.trim="supplier.razao_social" required="true" autofocus :invalid="submitted && !supplier.razao_social" />
                        <small class="p-invalid" v-if="!supplier.razao_social">Razão Social é obrigatória</small>
                    </div>
                    <div class="field">
                        <label for="supplier">Nome Fantasia</label>
                        <InputText id="supplier" v-model.trim="supplier.nome_fantasia" required="true" autofocus :invalid="submitted && !supplier.nome_fantasia" />
                        <small class="p-invalid" v-if="!supplier.nome_fantasia">Nome Fantasia é obrigatório</small>
                    </div>
                    <div class="field">
                        <label for="supplier">Endereço</label>
                        <InputText id="supplier" v-model.trim="supplier.endereco" required="true" autofocus :invalid="submitted && !supplier.endereco" />
                        <small class="p-invalid" v-if="!supplier.endereco">Endereço é obrigatório</small>
                    </div>
                    <div class="field">
                        <label for="inventoryStatus" class="mb-3">Status</label>
                        <Dropdown id="inventoryStatus" v-model="supplier.inventoryStatus" :options="statuses" optionLabel="label" placeholder="Selecione o status">
                            <template #value="slotProps">
                                <div v-if="slotProps.value && slotProps.value.value">
                                    <span :class="'supplier-badge status-' + slotProps.value.value">{{ slotProps.value.label }}</span>
                                </div>
                                <div v-else-if="slotProps.value && !slotProps.value.value">
                                    <span :class="'supplier-badge status-' + slotProps.value.toLowerCase()">{{ slotProps.value }}</span>
                                </div>
                                <span v-else>
                                    {{ slotProps.placeholder }}
                                </span>
                            </template>
                        </Dropdown>
                        <small class="p-invalid" v-if="!supplier.inventoryStatus">Status é obrigatório</small>
                    </div>
                    <template #footer>
                        <Button label="Cancel" icon="pi pi-times" text="" @click="hideDialog" />
                        <Button label="Save" icon="pi pi-check" text="" @click="saveProduct(supplier.cnpj, supplier.razao_social, supplier.nome_fantasia, supplier.endereco, supplier.inventoryStatus.value)" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="deleteProductDialog" :style="{ width: '450px' }" header="Confirmação" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="supplier"
                            >Deseja desativar o fornecedor<b>{{ supplier.name }}</b
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
                        <span v-if="supplier">Quer mesmo excluir os fornecedores selecionadas?</span>
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
