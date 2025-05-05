<script setup>
import { ProductService } from '@/service/ProductService';
import { FilterMatchMode, FilterOperator } from '@primevue/core/api';
import { useToast } from 'primevue/usetoast';
import { onMounted, ref } from 'vue';

onMounted(() => {
    ProductService.getProductsWithOrdersSmall().then((data) => (products.value = data));
});

const toast = useToast();
const dt = ref();
const products = ref();
const productDialog = ref(false);
const deleteProductDialog = ref(false);
const deleteProductsDialog = ref(false);
const product = ref({});
const filters = ref();
const expandedRows = ref({});
const selectedProducts = ref();
const pendingSaveProduct = ref(null);
const confirmSaveDialog = ref(false);
const confirmDeleteProductDialog = ref(false);
const confirmDeleteProductsDialog = ref(false);
const allColumns = ref([
    { field: 'code', header: 'Code' },
    { field: 'name', header: 'Name' },
    { field: 'image', header: 'Image' },
    { field: 'price', header: 'Price' },
    { field: 'category', header: 'Category' },
    { field: 'rating', header: 'Reviews' },
    { field: 'inventoryStatus', header: 'Status' }
]);
const initFilters = () => {
    filters.value = {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS },
        name: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.STARTS_WITH }] },
        category: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.STARTS_WITH }] },
        price: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }] },
        inventoryStatus: { value: null, matchMode: FilterMatchMode.EQUALS }
    };
};

initFilters();

const clearFilter = () => {
    initFilters();
};
const submitted = ref(false);
const statuses = ref([
    { label: 'INSTOCK', value: 'instock' },
    { label: 'LOWSTOCK', value: 'lowstock' },
    { label: 'OUTOFSTOCK', value: 'outofstock' }
]);

function formatCurrency(value) {
    if (value) return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
    return;
}

function openNew() {
    product.value = {};
    submitted.value = false;
    productDialog.value = true;
}

function hideDialog() {
    productDialog.value = false;
    submitted.value = false;
}

// function saveProduct() {
//     submitted.value = true;

//     if (product?.value.name?.trim()) {
//         if (product.value.id) {
//             product.value.inventoryStatus = product.value.inventoryStatus.value ? product.value.inventoryStatus.value : product.value.inventoryStatus;
//             products.value[findIndexById(product.value.id)] = product.value;
//             toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Updated', life: 3000 });
//         } else {
//             product.value.id = createId();
//             product.value.code = createId();
//             product.value.image = 'product-placeholder.svg';
//             product.value.inventoryStatus = product.value.inventoryStatus ? product.value.inventoryStatus.value : 'INSTOCK';
//             products.value.push(product.value);
//             toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Created', life: 3000 });
//         }

//         productDialog.value = false;
//         product.value = {};
//     }
// }

function saveProduct() {
    if (product?.value.name?.trim()) {
        pendingSaveProduct.value = { ...product.value };
        confirmSaveDialog.value = true;
        submitted.value = true;
    }
}

function confirmSave() {
    const prod = pendingSaveProduct.value;

    if (prod.id) {
        prod.inventoryStatus = prod.inventoryStatus.value ? prod.inventoryStatus.value : prod.inventoryStatus;
        products.value[findIndexById(prod.id)] = prod;
        toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Updated', life: 3000 });
    } else {
        prod.id = createId();
        prod.code = createId();
        prod.image = 'product-placeholder.svg';
        prod.inventoryStatus = prod.inventoryStatus ? prod.inventoryStatus.value : 'INSTOCK';
        products.value.push(prod);
        toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Created', life: 3000 });
    }

    productDialog.value = false;
    confirmSaveDialog.value = false;
    pendingSaveProduct.value = null;
    product.value = {};
    submitted.value = false;
}

function editProduct(prod) {
    product.value = { ...prod };
    productDialog.value = true;
}

function confirmDeleteProduct(prod) {
    product.value = prod;
    deleteProductDialog.value = true;
}
function proceedDeleteProduct() {
    deleteProductDialog.value = false;
    confirmDeleteProductDialog.value = true; // ikinci onay dialogu
}
function deleteProductConfirmed() {
    products.value = products.value.filter((val) => val.id !== product.value.id);
    confirmDeleteProductDialog.value = false;
    product.value = {};
    toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Deleted', life: 3000 });
}
// function deleteProduct() {
//     products.value = products.value.filter((val) => val.id !== product.value.id);
//     deleteProductDialog.value = false;
//     product.value = {};
//     toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Deleted', life: 3000 });
// }

function findIndexById(id) {
    let index = -1;
    for (let i = 0; i < products.value.length; i++) {
        if (products.value[i].id === id) {
            index = i;
            break;
        }
    }

    return index;
}

function createId() {
    let id = '';
    var chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    for (var i = 0; i < 5; i++) {
        id += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return id;
}

function exportCSV() {
    dt.value.exportCSV();
}

function confirmDeleteSelected() {
    deleteProductsDialog.value = true;
}

// function deleteSelectedProducts() {
//     products.value = products.value.filter((val) => !selectedProducts.value.includes(val));
//     deleteProductsDialog.value = false;
//     selectedProducts.value = null;
//     toast.add({ severity: 'success', summary: 'Successful', detail: 'Products Deleted', life: 3000 });
// }

function proceedDeleteSelectedProducts() {
    deleteProductsDialog.value = false;
    confirmDeleteProductsDialog.value = true;
}
function deleteSelectedProductsConfirmed() {
    products.value = products.value.filter((val) => !selectedProducts.value.includes(val));
    confirmDeleteProductsDialog.value = false;
    selectedProducts.value = null;
    toast.add({ severity: 'success', summary: 'Successful', detail: 'Products Deleted', life: 3000 });
}

function getStatusLabel(status) {
    switch (status) {
        case 'INSTOCK':
            return 'success';

        case 'LOWSTOCK':
            return 'warn';

        case 'OUTOFSTOCK':
            return 'danger';

        default:
            return null;
    }
}

const onRowExpand = (event) => {
    toast.add({ severity: 'info', summary: 'Product Expanded', detail: event.data.name, life: 3000 });
};
const onRowCollapse = (event) => {
    toast.add({ severity: 'success', summary: 'Product Collapsed', detail: event.data.name, life: 3000 });
};

const onColReorder = () => {
    toast.add({ severity: 'success', summary: 'Column Reordered', life: 3000 });
};
const onRowReorder = (event) => {
    products.value = event.value;
    toast.add({ severity: 'success', summary: 'Rows Reordered', life: 3000 });
};

const getSeverity = (product) => {
    switch (product.inventoryStatus) {
        case 'INSTOCK':
            return 'success';

        case 'LOWSTOCK':
            return 'warn';

        case 'OUTOFSTOCK':
            return 'danger';

        default:
            return null;
    }
};

const getOrderSeverity = (order) => {
    switch (order.status) {
        case 'DELIVERED':
            return 'success';

        case 'CANCELLED':
            return 'danger';

        case 'PENDING':
            return 'warn';

        case 'RETURNED':
            return 'info';

        default:
            return null;
    }
};

const selectedColumns = ref([...allColumns.value]);
const onToggle = (val) => {
    const defaultColumn = allColumns.value[0];
    if (val.length === 0) {
        toast.add({ severity: 'warn', summary: 'Warning', detail: 'En az bir sütun görünür olmalı', life: 3000 });
        selectedColumns.value = [defaultColumn];
        return;
    }

    selectedColumns.value = val;
};
</script>

<template>
    <div>
        <div class="card zoom-container">
            <Toolbar class="mb-6">
                <template #start>
                    <Button label="New" icon="pi pi-plus" severity="secondary" class="mr-2" @click="openNew" />
                    <Button label="Delete" icon="pi pi-trash" severity="secondary" @click="confirmDeleteSelected" :disabled="!selectedProducts || !selectedProducts.length" />
                    <Button type="button" icon="pi pi-filter-slash" class="ml-2" severity="secondary" label="Clear" outlined @click="clearFilter()" />
                </template>
                <template> </template>
                <template #end>
                    <Button label="Export" icon="pi pi-upload" severity="secondary" @click="exportCSV($event)" />
                </template>
            </Toolbar>
            <div class="flex flex-row">
                <MultiSelect v-model="selectedColumns" :options="allColumns" optionLabel="header" placeholder="Sütun Seç" display="chip" class="w-full md:w-20rem mb-2" @update:modelValue="onToggle" />
            </div>
            <DataTable
                ref="dt"
                v-model:expandedRows="expandedRows"
                v-model:selection="selectedProducts"
                v-model:filters="filters"
                :value="products"
                dataKey="id"
                :paginator="true"
                :rows="10"
                :filters="filters"
                filterDisplay="menu"
                paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                :rowsPerPageOptions="[5, 10, 25]"
                currentPageReportTemplate="Showing {first} to {last} of {totalRecords} products"
                :reorderableColumns="true"
                @columnReorder="onColReorder"
                @rowReorder="onRowReorder"
                @rowExpand="onRowExpand"
                @rowCollapse="onRowCollapse"
                :globalFilterFields="['name', 'price', 'category', 'status']"
                showGridlines
            >
                <template #header>
                    <div class="flex flex-wrap gap-2 items-center justify-between">
                        <h4 class="m-0">Manage Products</h4>
                        <IconField>
                            <InputIcon>
                                <i class="pi pi-search" />
                            </InputIcon>
                            <InputText v-model="filters['global'].value" placeholder="Search..." />
                        </IconField>
                    </div>
                </template>
                <Column expander style="width: 5rem" />
                <Column rowReorder headerStyle="width: 3rem" :reorderableColumn="false" />

                <Column :exportable="false">
                    <template #body="slotProps">
                        <Button icon="pi pi-pencil" outlined rounded class="mr-2" @click="editProduct(slotProps.data)" />
                        <Button icon="pi pi-trash" outlined rounded severity="danger" @click="confirmDeleteProduct(slotProps.data)" />
                    </template>
                </Column>
                <Column selectionMode="multiple" style="width: 3rem" :exportable="false"></Column>

                <Column v-for="col in selectedColumns" :key="col.field" :field="col.field" :header="col.header" sortable style="min-width: 12rem">
                    <Column v-if="col.field === 'image'" sortable style="min-width: 14rem">
                        <template #body="{ data }">
                            {{ data.name }}
                        </template>
                        <template #filter="{ filterModel }">
                            <InputText v-model="filterModel.value" type="text" placeholder="Search by name" />
                        </template>
                    </Column>
                    <template v-if="col.field === 'image'" #body="slotProps">
                        <img :src="`https://primefaces.org/cdn/primevue/images/product/${slotProps.data.image}`" :alt="slotProps.data.image" class="rounded" style="width: 64px" />
                    </template>

                    <template v-else-if="col.field === 'price'" #body="slotProps">
                        {{ formatCurrency(slotProps.data.price) }}
                    </template>

                    <template v-else-if="col.field === 'rating'" #body="slotProps">
                        <Rating :modelValue="slotProps.data.rating" :readonly="true" />
                    </template>

                    <template v-else-if="col.field === 'inventoryStatus'" #body="slotProps">
                        <Tag :value="slotProps.data.inventoryStatus" :severity="getStatusLabel(slotProps.data.inventoryStatus)" />
                    </template>

                    <template #filter="{ filterModel }">
                        <InputText v-model="filterModel.value" type="text" placeholder="Search by name" />
                    </template>
                </Column>
                <template #expansion="slotProps">
                    <div class="p-4">
                        <h5>Orders for {{ slotProps.data.name }}</h5>
                        <DataTable :value="slotProps.data.orders">
                            <Column field="id" header="Id" sortable></Column>
                            <Column field="customer" header="Customer" sortable></Column>
                            <Column field="date" header="Date" sortable></Column>
                            <Column field="amount" header="Amount" sortable>
                                <template #body="slotProps">
                                    {{ formatCurrency(slotProps.data.amount) }}
                                </template>
                            </Column>
                            <Column field="status" header="Status" sortable>
                                <template #body="slotProps">
                                    <Tag :value="slotProps.data.status.toLowerCase()" :severity="getOrderSeverity(slotProps.data)" />
                                </template>
                            </Column>
                            <Column headerStyle="width:4rem">
                                <template #body>
                                    <Button icon="pi pi-search" />
                                </template>
                            </Column>
                        </DataTable>
                    </div>
                </template>
            </DataTable>
        </div>

        <Dialog v-model:visible="productDialog" :style="{ width: '450px' }" header="Product Details" :modal="true">
            <div class="flex flex-col gap-6">
                <img v-if="product.image" :src="`https://primefaces.org/cdn/primevue/images/product/${product.image}`" :alt="product.image" class="block m-auto pb-4" />
                <div>
                    <label for="name" class="block font-bold mb-3">Name</label>
                    <InputText id="name" v-model.trim="product.name" required="true" autofocus :invalid="submitted && !product.name" fluid />
                    <small v-if="submitted && !product.name" class="text-red-500">Name is required.</small>
                </div>
                <div>
                    <label for="description" class="block font-bold mb-3">Description</label>
                    <Textarea id="description" v-model="product.description" required="true" rows="3" cols="20" fluid />
                </div>
                <div>
                    <label for="inventoryStatus" class="block font-bold mb-3">Inventory Status</label>
                    <Select id="inventoryStatus" v-model="product.inventoryStatus" :options="statuses" optionLabel="label" placeholder="Select a Status" fluid></Select>
                </div>

                <div>
                    <span class="block font-bold mb-4">Category</span>
                    <div class="grid grid-cols-12 gap-4">
                        <div class="flex items-center gap-2 col-span-6">
                            <RadioButton id="category1" v-model="product.category" name="category" value="Accessories" />
                            <label for="category1">Accessories</label>
                        </div>
                        <div class="flex items-center gap-2 col-span-6">
                            <RadioButton id="category2" v-model="product.category" name="category" value="Clothing" />
                            <label for="category2">Clothing</label>
                        </div>
                        <div class="flex items-center gap-2 col-span-6">
                            <RadioButton id="category3" v-model="product.category" name="category" value="Electronics" />
                            <label for="category3">Electronics</label>
                        </div>
                        <div class="flex items-center gap-2 col-span-6">
                            <RadioButton id="category4" v-model="product.category" name="category" value="Fitness" />
                            <label for="category4">Fitness</label>
                        </div>
                    </div>
                </div>

                <div class="grid grid-cols-12 gap-4">
                    <div class="col-span-6">
                        <label for="price" class="block font-bold mb-3">Price</label>
                        <InputNumber id="price" v-model="product.price" mode="currency" currency="USD" locale="en-US" fluid />
                    </div>
                    <div class="col-span-6">
                        <label for="quantity" class="block font-bold mb-3">Quantity</label>
                        <InputNumber id="quantity" v-model="product.quantity" integeronly fluid />
                    </div>
                </div>
            </div>

            <template #footer>
                <Button label="Cancel" icon="pi pi-times" text @click="hideDialog" />
                <Button label="Save" icon="pi pi-check" @click="saveProduct" />
            </template>
        </Dialog>

        <Dialog v-model:visible="confirmSaveDialog" :style="{ width: '450px' }" header="Confirm Save" :modal="true">
            <div class="flex items-center gap-4">
                <i class="pi pi-question-circle text-3xl" />
                <span
                    >Are you sure you want to save <b>{{ pendingSaveProduct?.name || '' }}</b
                    >?</span
                >
            </div>
            <template #footer>
                <Button label="No" icon="pi pi-times" text @click="confirmSaveDialog = false" />
                <Button label="Yes" icon="pi pi-check" text @click="confirmSave" />
            </template>
        </Dialog>

        <Dialog v-model:visible="deleteProductDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
            <div class="flex items-center gap-4">
                <i class="pi pi-exclamation-triangle !text-3xl" />
                <span v-if="product"
                    >Are you sure you want to delete <b>{{ product.name }}</b
                    >?</span
                >
            </div>
            <template #footer>
                <Button label="No" icon="pi pi-times" text @click="deleteProductDialog = false" />
                <Button label="Yes" icon="pi pi-check" text @click="proceedDeleteProduct" />
            </template>
        </Dialog>
        <Dialog v-model:visible="confirmDeleteProductDialog" :style="{ width: '450px' }" header="Final Confirmation" :modal="true">
            <div class="flex items-center gap-4">
                <i class="pi pi-exclamation-triangle text-3xl" />
                <span
                    >Are you really sure you want to permanently delete <b>{{ product.name }}</b
                    >?</span
                >
            </div>
            <template #footer>
                <Button label="No" icon="pi pi-times" text @click="confirmDeleteProductDialog = false" />
                <Button label="Yes, delete" icon="pi pi-check" text severity="danger" @click="deleteProductConfirmed" />
            </template>
        </Dialog>
        <Dialog v-model:visible="deleteProductsDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
            <div class="flex items-center gap-4">
                <i class="pi pi-exclamation-triangle text-3xl" />
                <span>Are you sure you want to delete the selected products?</span>
            </div>
            <template #footer>
                <Button label="No" icon="pi pi-times" text @click="deleteProductsDialog = false" />
                <Button label="Yes" icon="pi pi-check" text @click="proceedDeleteSelectedProducts" />
            </template>
        </Dialog>
        <Dialog v-model:visible="confirmDeleteProductsDialog" :style="{ width: '450px' }" header="Final Confirmation" :modal="true">
            <div class="flex items-center gap-4">
                <i class="pi pi-exclamation-triangle text-3xl" />
                <span>This action is irreversible. Are you really sure you want to permanently delete all selected products?</span>
            </div>
            <template #footer>
                <Button label="No" icon="pi pi-times" text @click="confirmDeleteProductsDialog = false" />
                <Button label="Yes, delete" icon="pi pi-check" text severity="danger" @click="deleteSelectedProductsConfirmed" />
            </template>
        </Dialog>
    </div>
</template>
<style scoped>
.zoom-container {
    transform: scale(0.9);
}
</style>
