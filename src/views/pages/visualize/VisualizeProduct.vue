<script setup>
import { FilterMatchMode } from 'primevue/api';
import { ref, onMounted, onBeforeMount } from 'vue';
// import { ProductService } from '@/service/ProductService';
import { useToast } from 'primevue/usetoast';
import axios from 'axios';
const toast = useToast();
const suppliers = ref([]);
const categories = ref([]);
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
let idProduct = '';

const getBadgeSeverity = (inventoryStatus) => {
    switch (inventoryStatus.toLowerCase()) {
        case 'ativo':
            return 'success';
        case 'inativo':
            return 'danger';
    }
};

const fetchSupplier = () => {
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .get('http://localhost:8000/fornecedores-active')
        .then((response) => {
            // Mapeie os dados dos fornecedores para o formato necessário pelo Dropdown
            suppliers.value = response.data.map((supplier) => ({
                label: supplier[2],
                value: supplier[0]
            }));
        })
        .catch((error) => {
            console.error('Erro ao buscar fornecedores:', error);
        });
};
const fetchCategory = () => {
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .get('http://localhost:8000/categorias-active')
        .then((response) => {
            // Mapeie os dados dos fornecedores para o formato necessário pelo Dropdown
            categories.value = response.data.map((category) => ({
                label: category[1],
                value: category[0]
            }));
        })
        .catch((error) => {
            console.error('Erro ao buscar categorias:', error);
        });
};
fetchSupplier();
fetchCategory();
onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .get('http://localhost:8000/produto')
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

const hideDialog = () => {
    productDialog.value = false;
    submitted.value = false;
};

const saveProduct = (product, price, status, unit_measure, id_supplier, id_category, description) => {
    submitted.value = true;
    product = 'asdasd';
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .put(`http://localhost:8000/editar/produto/${idProduct}`, { product, price, status, unit_measure, id_supplier, id_category, description })
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

    productDialog.value = false;
    product.value = {};
};

const editProduct = (editProduct) => {
    idProduct = editProduct;
    product.value = { ...editProduct };
    productDialog.value = true;
};

const confirmDeleteProduct = (editProduct) => {
    product.value = editProduct;
    idProduct = editProduct;
    deleteProductDialog.value = true;
};

const deleteProduct = () => {
    /////aqui após confirmar
    const token = localStorage.getItem('authorization');
    // Configurar o token no cabeçalho 'Authorization'
    axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    axios
        .delete(`http://localhost:8000/excluir/produto/${idProduct}`)
        .then((response) => {
            if (response.status === 200) {
                deleteProductDialog.value = false;
                toast.add({ severity: 'success', summary: 'Produto deletada', detail: 'Produto deletado', life: 3000 });
                window.location.reload();
            }
        })
        .catch((error) => {
            const message = error.response.data.message;
            toast.add({ severity: 'error', summary: 'Erro ao deletar produto', detail: message, life: 3000 });
        });
};

const exportCSV = () => {
    dt.value.exportCSV();
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
                    currentPageReportTemplate="Mostrando {first} à {last} de {totalRecords} produtos"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Gerenciar Produtos</h5>
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
                    <Column field="1" header="Produto" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Produto</span>
                            {{ slotProps.data[1] }}
                        </template>
                    </Column>
                    <Column field="1" header="Valor" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Valor</span>
                            R$ {{ slotProps.data[2].toFixed(2) }}
                        </template>
                    </Column>
                    <Column field="1" header="Descrição" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Descrição</span>
                            {{ slotProps.data[7] }}
                        </template>
                    </Column>
                    <Column field="1" header="Unid medida" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Unid medida</span>
                            {{ slotProps.data[4] }}
                        </template>
                    </Column>
                    <Column field="1" header="Fornecedor" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Fornecedor</span>
                            {{ slotProps.data[5] }}
                        </template>
                    </Column>
                    <Column field="1" header="Categoria" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Categoria</span>
                            {{ slotProps.data[6] }}
                        </template>
                    </Column>
                    <Column field="1" header="ICMS" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">ICMS</span>
                            {{ slotProps.data[8] }}
                        </template>
                    </Column>
                    <Column field="2" header="Status" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Status</span>
                            <Tag :severity="getBadgeSeverity(slotProps.data[3])">{{ slotProps.data[3] }}</Tag>
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
                        <label for="product">Produto</label>
                        <InputText id="product" v-model.trim="product.name" required="true" autofocus :invalid="submitted && !product.name" />
                        <small class="p-invalid" v-if="!product.name">Produto é obrigatória</small>
                    </div>
                    <div class="field">
                        <label for="price">Valor</label>
                        <InputText id="price" v-model.trim="product.price" required="true" autofocus :invalid="submitted && !product.price" />
                        <small class="p-invalid" v-if="!product.price">valor é obrigatório</small>
                    </div>
                    <div class="field">
                        <label for="description">Descrição</label>
                        <InputText id="description" v-model.trim="product.description" required="true" autofocus :invalid="submitted && !product.description" />
                        <small class="p-invalid" v-if="!product.description">Descrição é obrigatória</small>
                    </div>
                    <div class="field">
                        <label for="unit_measure">Unidade medida</label>
                        <InputText id="unit_measure" v-model.trim="product.unit_measure" required="true" autofocus :invalid="submitted && !product.unit_measure" />
                        <small class="p-invalid" v-if="!product.unit_measure">Unidade de medida é obrigatória</small>
                    </div>
                    <div class="field">
                        <Dropdown id="supplier" v-model="product.id_supplier" :options="suppliers" optionLabel="label" placeholder="Selecione um fornecedor">
                            <template #value="suppliers">
                                <div class="width-dropdown">
                                    <!-- Adicione uma classe para o estilo específico do dropdown -->
                                    <div v-if="suppliers.value && suppliers.value.label">
                                        <span :class="'supplier-badge status-' + suppliers.value.value">{{ suppliers.value.label }}</span>
                                    </div>
                                    <div v-else-if="suppliers.value && !suppliers.value.value">
                                        <span :class="'supplier-badge status-' + suppliers.value.value.toLowerCase()">{{ suppliers.value }}</span>
                                    </div>
                                    <span v-else>
                                        {{ suppliers.placeholder }}
                                    </span>
                                </div>
                            </template>
                        </Dropdown>
                    </div>
                    <div class="field">
                        <Dropdown id="category" v-model="product.id_category" :options="categories" optionLabel="label" placeholder="Selecione uma categoria">
                            <template #value="categories">
                                <div class="width-dropdown">
                                    <div v-if="categories.value && categories.value.label">
                                        <span :class="'supplier-badge status-' + categories.value.value">{{ categories.value.label }}</span>
                                    </div>
                                    <div v-else-if="categories.value && !categories.value.value">
                                        <span :class="'supplier-badge status-' + categories.value.value.toLowerCase()">{{ categories.value }}</span>
                                    </div>
                                    <span v-else>
                                        {{ categories.placeholder }}
                                    </span>
                                </div>
                            </template>
                        </Dropdown>
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
                        <Button
                            label="Save"
                            icon="pi pi-check"
                            text=""
                            @click="saveProduct(product.name, product.price, product.inventoryStatus.value, product.unit_measure, product.id_supplier.value, product.id_category.value, product.description)"
                        />
                    </template>
                </Dialog>

                <Dialog v-model:visible="deleteProductDialog" :style="{ width: '450px' }" header="Confirmação" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="product"
                            >Deseja desativar o produto<b>{{ product.name }}</b
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
                        <span v-if="product">Quer mesmo excluir as produtos selecionadas?</span>
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
