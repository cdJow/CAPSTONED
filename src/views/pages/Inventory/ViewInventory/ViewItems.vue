<script setup>
import { CustomerService } from '@/service/CustomerService';
import { ProductService } from '@/service/ProductService';
import { FilterMatchMode, FilterOperator } from '@primevue/core/api';
import { onBeforeMount, reactive, ref } from 'vue';

const customers1 = ref(null);
const customers2 = ref(null);
const customers3 = ref(null);
const loading1 = ref(null);
const balanceFrozen = ref(false);
const products = ref(null);
const expandedRows = ref([]);
const statuses = reactive(['unqualified', 'qualified', 'new', 'negotiation', 'renewal', 'proposal']);
const batchExpandedRows = ref([]);
const visibleTables = ref({});
const op2 = ref();
const selectedProduct = ref(null);
const serialNumbers = ref([]);


function getOrderSeverity(order) {
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
}




function toggleDataTable(event) {
    op2.value.toggle(event);
}

function clearFilter() {
    filters.value = {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS },
        name: { value: null, matchMode: FilterMatchMode.STARTS_WITH },
        brand: { value: null, matchMode: FilterMatchMode.STARTS_WITH },
        category: { value: null, matchMode: FilterMatchMode.EQUALS },
        inventoryStatus: { value: null, matchMode: FilterMatchMode.EQUALS }
    };
}

function onProductSelect(event) {
    console.log('Selected serial number:', event.data);
}

function getSeverity(status) {
    switch (status) {
        case 'unqualified':
            return 'danger';

        case 'qualified':
            return 'success';

        case 'new':
            return 'info';

        case 'negotiation':
            return 'warn';

        case 'renewal':
            return null;
    }
}


function getStockSeverity(product) {
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
}

onBeforeMount(() => {
    ProductService.getProductsWithOrdersSmall().then((data) => (products.value = data));
    CustomerService.getCustomersLarge().then((data) => {
        customers1.value = data;
        loading1.value = false;
        customers1.value.forEach((customer) => (customer.date = new Date(customer.date)));
    });
    CustomerService.getCustomersLarge().then((data) => (customers2.value = data));
    CustomerService.getCustomersMedium().then((data) => (customers3.value = data));

    initFilters(); // Use the new initFilters instead of initFilters1
});

const filters = ref({
    global: { value: null, matchMode: FilterMatchMode.CONTAINS },
    name: { value: null, matchMode: FilterMatchMode.STARTS_WITH },
    brand: { value: null, matchMode: FilterMatchMode.STARTS_WITH },
    category: { value: null, matchMode: FilterMatchMode.EQUALS },
    inventoryStatus: { value: null, matchMode: FilterMatchMode.EQUALS }
});

onBeforeMount(() => {
    ProductService.getProductsWithOrdersSmall().then((data) => (products.value = data));
    CustomerService.getCustomersLarge().then((data) => {
        customers1.value = data;
        loading1.value = false;
        customers1.value.forEach((customer) => (customer.date = new Date(customer.date)));
    });
    CustomerService.getCustomersLarge().then((data) => (customers2.value = data));
    CustomerService.getCustomersMedium().then((data) => (customers3.value = data));
    initFilters();
});


function initFilters() {
    filters.value = {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS },
        name: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.STARTS_WITH }] },
        brand: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.STARTS_WITH }] },
        category: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }] },
        inventoryStatus: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }] }
    };
}




function expandAll() {
    expandedRows.value = products.value.reduce((acc, p) => (acc[p.id] = true) && acc, {});
}

function collapseAll() {
    expandedRows.value = null;
}

function formatCurrency(value) {
    return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
}

function formatDate(value) {
    return value.toLocaleDateString('en-US', {
        day: '2-digit',
        month: '2-digit',
        year: 'numeric'
    });
}

function calculateCustomerTotal(name) {
    let total = 0;
    if (customers3.value) {
        for (let customer of customers3.value) {
            if (customer.representative.name === name) {
                total++;
            }
        }
    }

    return total;
}
</script>

<template>



<div class="card">
    <div class="font-semibold text-xl mb-4">Item List</div>
    <DataTable
    v-model:expandedRows="expandedRows" 
    :value="products" 
    dataKey="id" 
    v-model:filters="filters"
    filterDisplay="menu"
    :globalFilterFields="['name', 'brand', 'category', 'inventoryStatus']"
    tableStyle="min-width: 60rem"
>
<template #header>
    <div class="flex justify-between items-center">
        <!-- Filter Controls -->
        <div class="flex items-center gap-2">
            <Button 
                type="button" 
                icon="pi pi-filter-slash" 
                label="Clear" 
                outlined 
                @click="clearFilter()" 
            />
            <IconField>
                <InputIcon>
                    <i class="pi pi-search" />
                </InputIcon>
                <InputText 
                    v-model="filters['global'].value" 
                    placeholder="Keyword Search" 
                />
            </IconField>
        </div>

        <!-- Expand/Collapse Buttons -->
        <div class="flex flex-wrap justify-end gap-2">
            <Button text icon="pi pi-plus" label="Expand All" @click="expandAll" />
            <Button text icon="pi pi-minus" label="Collapse All" @click="collapseAll" />
        </div>
    </div>
</template>

        <Column expander style="width: 2rem" />
        
        <Column field="name" header="Product Name" sortable="true" style="min-width: ">
</Column>

        <Column field="brand" sortable header="Brand"></Column>
        <Column field="description" sortable header="Description"></Column>
        <Column field="category" sortable header="Category"></Column>
        <Column field="inventoryStatus" sortable header="Status">
            <template #body="slotProps">
                <Tag :value="slotProps.data.inventoryStatus" :severity="getStockSeverity(slotProps.data)" />
            </template>
        </Column>
        <template #expansion="slotProps">
            <div class="p-4">
                
             

                <div class="flex items-center gap-2 mb-4">
                    <h5>Batch List for {{ slotProps.data.name }}</h5>
            <Button 
                type="button" 
                icon="pi pi-filter-slash" 
                label="Clear" 
                outlined 
                @click="clearFilter()" 
            />
            <IconField>
                <InputIcon>
                    <i class="pi pi-search" />
                </InputIcon>
                <InputText 
                    v-model="filters['global'].value" 
                    placeholder="Keyword Search" 
                />
            </IconField>
        </div>
                <DataTable class="p-datatable-sm" v-model:expandedRows="batchExpandedRows" :value="slotProps.data.batches" dataKey="batchId">
                    
                    <Column field="batchNumber" header="Batch Number" sortable></Column>
                    <Column field="quantity" header="Quantity" sortable></Column>
                    <Column field="purchaseDate" header="Purchase Date" sortable></Column>
                    <Column field="purchasePrice" header="Purchase Price" sortable></Column>
                    <Column field="unit" header="Unit" sortable></Column>
                    <Column field="supplier" header="Supplier" sortable></Column>
                    <Column field="expDate" header="Exp Date" sortable>
                    <template #body="slotProps">
                            <span style="color: red;">{{ slotProps.data.expDate }}</span>
                    </template>
                    </Column>
                    <Column field="status" header="Status" sortable>
                        <template #body="slotProps">
                            <Tag :value="slotProps.data.status" :severity="getOrderSeverity(slotProps.data)" />
                        </template>
                    </Column>
                    <column header="Items">

                        <template #body="slotProps">
                            
                            <div class="flex flex-wrap gap-2">
                    <Button type="button" icon="pi pi-list" @click="toggleDataTable" rounded class="mr-2" />
                    <Popover ref="op2" id="overlay_panel" style="width: 450px">
                        
                        <DataTable v-model:selection="selectedProduct" :value="products" selectionMode="single" :paginator="true" :rows="5" @row-select="onProductSelect">
                            <div class="flex items-center gap-2">
                                <h6>Items for {{ slotProps.data.batchNumber }}</h6>
                    <Button 
                        type="button" 
                        icon="pi pi-filter-slash" 
                        label="Clear" 
                        outlined 
                        @click="clearFilter()" 
                    />
                    <IconField>
                        <InputIcon>
                            <i class="pi pi-search" />
                        </InputIcon>
                        <InputText 
                            v-model="filters['global'].value" 
                            placeholder="Keyword Search" 
                        />
                    </IconField>
                </div>
                                
                            <DataTable class="p-datatable-sm" :value="slotProps.data.serialNumbers">
                                <Column field="serialNumber" header="Serial Number" sortable></Column>
                            </DataTable>
                        </DataTable>
                    </Popover>
                </div>
                        </template>
                    </column>
                  
                </DataTable>
            </div>
        </template>
    </DataTable>
</div>

 
</template>

<style scoped lang="scss">
:deep(.p-datatable-frozen-tbody) {
    font-weight: bold;
}

:deep(.p-datatable-scrollable .p-frozen-column) {
    font-weight: bold;
}
</style>
