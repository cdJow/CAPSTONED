<script setup>
import { CustomerService } from "@/service/CustomerService";
import { ProductService } from "@/service/ProductService";
import { FilterMatchMode, FilterOperator } from "@primevue/core/api";
import { InputNumber } from "primevue";
import { onBeforeMount, ref } from "vue";

const customers1 = ref(null);
const customers2 = ref(null);
const customers3 = ref(null);
const loading1 = ref(null);
const expandedRows = ref([]);
const batchExpandedRows = ref([]);

const NonConsumablebatchDialog = ref(false);

const op2 = ref();
const selectedProduct = ref(null);

const deleteProductDialog = ref(false); // State for delete dialog
const deleteBatchDialog = ref(false);
const selectedBatchForDeletion = ref(null);

const product = ref({}); // Initialize product as a reactive reference
const batchDialog = ref(false);
const selectedBatch = ref({});

const productDialog = ref(false);
const products = ref([]); // Use `ref` for reactivity

const submitted = ref(false);

function editProduct(prod) {
    product.value = { ...prod }; // Clone the product data to avoid direct mutations
    productDialog.value = true; // Open the dialog
}

function hideDialog() {
    productDialog.value = false;
    batchDialog.value = false;
    submitted.value = false;
    NonConsumablebatchDialog.value = false;
}

function saveProduct() {
    submitted.value = true;

    if (product?.value.name?.trim()) {
        if (product.value.id) {
            // Update existing product
            const index = products.value.findIndex(
                (p) => p.id === product.value.id,
            );
            if (index !== -1) {
                products.value[index] = product.value;
            }
        } else {
            // Create new product
            product.value.id = createId();
            products.value.push(product.value);
        }
        productDialog.value = false;
        product.value = {};
    }
}

const unitOptions = ref([
    { label: "Kilograms", value: "kg" },
    { label: "Liters", value: "l" },
    { label: "Pieces", value: "pcs" },
]);

function saveNonConsumableBatchDetails() {
    // Validation: Check required fields
    if (
        !selectedBatch.value.batchNumber ||
        selectedBatch.value.quantity == null ||
        !selectedBatch.value.purchaseDate ||
        selectedBatch.value.purchasePrice == null
    ) {
        console.error("Some required fields are missing.");
        return;
    }

    // Find the product containing the batch
    const productIndex = products.value.findIndex((product) =>
        product.batches?.some(
            (batch) => batch.batchId === selectedBatch.value.batchId,
        ),
    );

    if (productIndex !== -1) {
        const batchIndex = products.value[productIndex].batches.findIndex(
            (batch) => batch.batchId === selectedBatch.value.batchId,
        );

        if (batchIndex !== -1) {
            // Update the batch
            products.value[productIndex].batches[batchIndex] = {
                ...selectedBatch.value,
            };
            console.log("Batch updated successfully:", selectedBatch.value);
        } else {
            console.warn("Batch not found in the product's list.");
        }
    } else {
        console.warn("Product not found containing this batch.");
    }

    // Close the dialog
    NonConsumablebatchDialog.value = false;

    console.log("Updated Products List:", products.value);
}

function saveBatchDetails() {
    submitted.value = true;

    if (selectedBatch?.value.batchNumber?.trim()) {
        const productIndex = products.value.findIndex(
            (p) => p.id === selectedBatch.value.productId,
        );
        if (productIndex !== -1) {
            const batchIndex = products.value[productIndex].batches.findIndex(
                (b) => b.batchId === selectedBatch.value.batchId,
            );
            if (batchIndex !== -1) {
                // Update existing batch
                products.value[productIndex].batches[batchIndex] =
                    selectedBatch.value;
            } else {
                // Add new batch
                selectedBatch.value.batchId = createId(); // Assuming batchId is similar to product id
                products.value[productIndex].batches.push(selectedBatch.value);
            }
        }
        batchDialog.value = false;
        selectedBatch.value = {};
    }
}

function createId() {
    return Math.random().toString(36).substr(2, 9);
}

function confirmDeleteProduct(prod) {
    product.value = prod; // Set the product to be deleted
    deleteProductDialog.value = true; // Show the confirmation dialog
}

function deleteProduct() {
    if (product.value) {
        const index = products.value.findIndex(
            (p) => p.id === product.value.id,
        );
        if (index !== -1) {
            products.value.splice(index, 1); // Remove the product from the list
        }
        deleteProductDialog.value = false; // Close the dialog
        product.value = null; // Reset the product reference
    }
}

function getOrderSeverity(order) {
    switch (order.status) {
        case "DELIVERED":
            return "success";

        case "CANCELLED":
            return "danger";

        case "PENDING":
            return "warn";

        case "RETURNED":
            return "info";

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
        inventoryStatus: { value: null, matchMode: FilterMatchMode.EQUALS },
    };
}

function onProductSelect(event) {
    console.log("Selected serial number:", event.data);
}

function getStockSeverity(product) {
    switch (product.inventoryStatus) {
        case "INSTOCK":
            return "success";

        case "LOWSTOCK":
            return "warn";

        case "OUTOFSTOCK":
            return "danger";

        default:
            return null;
    }
}

function formatDate(value) {
    return new Intl.DateTimeFormat("en-US", {
        year: "numeric",
        month: "2-digit",
        day: "2-digit",
    }).format(new Date(value));
}

onBeforeMount(() => {
    ProductService.getProductsWithOrdersSmall().then((data) => {
        console.log(data); // Check the data being loaded
        products.value = data;
    });
    // Other service calls...
});

const filters = ref({
    global: { value: null, matchMode: FilterMatchMode.CONTAINS },
    name: { value: null, matchMode: FilterMatchMode.STARTS_WITH },
    brand: { value: null, matchMode: FilterMatchMode.STARTS_WITH },
    category: { value: null, matchMode: FilterMatchMode.EQUALS },
    inventoryStatus: { value: null, matchMode: FilterMatchMode.EQUALS },
});

onBeforeMount(() => {
    ProductService.getProductsWithOrdersSmall().then(
        (data) => (products.value = data),
    );
    CustomerService.getCustomersLarge().then((data) => {
        customers1.value = data;
        loading1.value = false;
        customers1.value.forEach(
            (customer) => (customer.date = new Date(customer.date)),
        );
    });
    CustomerService.getCustomersLarge().then(
        (data) => (customers2.value = data),
    );
    CustomerService.getCustomersMedium().then(
        (data) => (customers3.value = data),
    );
    initFilters();
});

function editbatch(batchData) {
    if (!batchData) {
        console.error("Batch data is not defined");
        return;
    }
    selectedBatch.value = { ...batchData }; // Clone the batch data
    batchDialog.value = true; // Open the dialog
}

function openNonConsumableBatchDialog(batchData) {
    if (!batchData) {
        console.error("Batch data is undefined");
        return;
    }

    // Clone the batch data to avoid direct mutation
    selectedBatch.value = { ...batchData };

    // Open the dialog
    NonConsumablebatchDialog.value = true;

    console.log(
        "Opening Non-Consumable Batch Dialog with data:",
        selectedBatch.value,
    );
}

function formatPrice(value) {
    return parseFloat(value).toFixed(2);
}

function initFilters() {
    filters.value = {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS },
        name: {
            operator: FilterOperator.AND,
            constraints: [
                { value: null, matchMode: FilterMatchMode.STARTS_WITH },
            ],
        },
        brand: {
            operator: FilterOperator.AND,
            constraints: [
                { value: null, matchMode: FilterMatchMode.STARTS_WITH },
            ],
        },
        category: {
            operator: FilterOperator.AND,
            constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }],
        },
        inventoryStatus: {
            operator: FilterOperator.AND,
            constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }],
        },
    };
}

function expandAll() {
    expandedRows.value = products.value.reduce(
        (acc, p) => (acc[p.id] = true) && acc,
        {},
    );
}

function collapseAll() {
    expandedRows.value = null;
}

function confirmDeleteBatch(batch, productId) {
    if (!batch || !productId) {
        console.error(
            "Invalid batch or product ID passed to confirmDeleteBatch",
        );
        return;
    }
    selectedBatchForDeletion.value = { ...batch, productId }; // Set the batch and product ID
    deleteBatchDialog.value = true; // Open the delete confirmation dialog
    console.log("Batch set for deletion:", selectedBatchForDeletion.value);
}
</script>

<template>
    <div class="card">
        <div class="font-semibold text-xl mb-4">Manage Items</div>
        <DataTable
            v-model:expandedRows="expandedRows"
            :value="products"
            dataKey="id"
            v-model:filters="filters"
            filterDisplay="menu"
            :globalFilterFields="[
                'name',
                'brand',
                'category',
                'inventoryStatus',
            ]"
            tableStyle="min-width: 60rem"
            paginator
            :rows="10"
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
                        <Button
                            text
                            icon="pi pi-plus"
                            label="Expand All"
                            @click="expandAll"
                        />
                        <Button
                            text
                            icon="pi pi-minus"
                            label="Collapse All"
                            @click="collapseAll"
                        />
                    </div>
                </div>
            </template>

            <Column expander style="width: 2rem" />
            <Column
                field="name"
                header="Product Name"
                style="min-width:"
            ></Column>
            <Column field="brand" sortable header="Brand"></Column>
            <Column field="description" sortable header="Description"></Column>
            <Column
                field="category"
                sortable
                header="Category"
                style="width: 10rem"
            ></Column>
            <Column field="inventoryStatus" sortable header="Status">
                <template #body="slotProps">
                    <Tag
                        :value="slotProps.data.inventoryStatus"
                        :severity="getStockSeverity(slotProps.data)"
                    />
                </template>
            </Column>
            <Column
                field="Actions"
                header="Actions"
                :exportable="false"
                style="min-width: 12rem"
            >
                <template #body="slotProps">
                    <Button
                        icon="pi pi-pencil"
                        outlined
                        rounded
                        class="mr-2"
                        @click="editProduct(slotProps.data)"
                    />
                    <Button
                        icon="pi pi-trash"
                        outlined
                        rounded
                        severity="danger"
                        @click="confirmDeleteProduct(slotProps.data)"
                    />
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
                    <DataTable
                        class="p-datatable-sm"
                        v-model:expandedRows="batchExpandedRows"
                        :value="slotProps.data.batches"
                        dataKey="batchId"
                        v-if="slotProps.data.category === 'Consumable'"
                    >
                        <Column
                            field="batchNumber"
                            header="Batch Number"
                            sortable
                        ></Column>
                        <Column
                            field="quantity"
                            header="Quantity"
                            sortable
                        ></Column>
                        <Column
                            field="purchaseDate"
                            header="Purchase Date"
                            sortable
                        ></Column>
                        <Column
                            field="purchasePrice"
                            header="Purchase Price"
                            sortable
                        >
                            <template #body="slotProps">
                                <span>{{
                                    formatPrice(slotProps.data.purchasePrice)
                                }}</span>
                            </template>
                        </Column>
                        <Column field="unit" header="Unit" sortable></Column>
                        <Column
                            field="supplier"
                            header="Supplier"
                            sortable
                        ></Column>
                        <Column field="expDate" header="Exp Date" sortable>
                            <template #body="slotProps">
                                <span style="color: red">{{
                                    formatDate(slotProps.data.expDate)
                                }}</span>
                            </template>
                        </Column>
                        <Column field="status" header="Status" sortable>
                            <template #body="slotProps">
                                <Tag
                                    :value="slotProps.data.status"
                                    :severity="getOrderSeverity(slotProps.data)"
                                />
                            </template>
                        </Column>
                        <Column
                            field="Actions"
                            header="Actions"
                            :exportable="false"
                            style="min-width: 3rem"
                            class="gap-2"
                        >
                            <template #body="slotProps">
                                <Button
                                    icon="pi pi-pencil"
                                    outlined
                                    rounded
                                    class="mb-1 mr-2"
                                    @click="editbatch(slotProps.data)"
                                />
                                <Button
                                    icon="pi pi-trash"
                                    outlined
                                    rounded
                                    severity="danger"
                                    @click="confirmDeleteBatch(slotProps.data)"
                                />
                            </template>
                        </Column>

                        <column header="Items">
                            <template #body="slotProps">
                                <div class="flex flex-wrap gap-2">
                                    <Button
                                        type="button"
                                        icon="pi pi-list"
                                        @click="toggleDataTable"
                                        rounded
                                        class="mr-2"
                                    />
                                    <Popover
                                        ref="op2"
                                        id="overlay_panel"
                                        style="width: 450px"
                                    >
                                        <DataTable
                                            v-model:selection="selectedProduct"
                                            :value="products"
                                            selectionMode="single"
                                            paginator
                                            :rows="5"
                                            :totalRecords="products.length"
                                            @row-select="onProductSelect"
                                        >
                                            <div
                                                class="flex items-center gap-2"
                                            >
                                                <h6>
                                                    Items for
                                                    {{
                                                        slotProps.data
                                                            .batchNumber
                                                    }}
                                                </h6>
                                                <Button
                                                    type="button"
                                                    icon="pi pi-filter-slash"
                                                    label="Clear"
                                                    outlined
                                                    @click="clearFilter()"
                                                />
                                                <IconField>
                                                    <InputIcon>
                                                        <i
                                                            class="pi pi-search"
                                                        />
                                                    </InputIcon>
                                                    <InputText
                                                        v-model="
                                                            filters['global']
                                                                .value
                                                        "
                                                        placeholder="Keyword Search"
                                                    />
                                                </IconField>
                                            </div>

                                            <DataTable
                                                class="p-datatable-sm"
                                                :value="
                                                    slotProps.data.serialNumbers
                                                "
                                            >
                                                <Column
                                                    field="serialNumber"
                                                    header="Serial Number"
                                                    sortable
                                                ></Column>
                                                <Column
                                                    :exportable="false"
                                                    style="min-width: 12rem"
                                                >
                                                    <template #body="slotProps">
                                                        <Button
                                                            icon="pi pi-pencil"
                                                            outlined
                                                            rounded
                                                            class="mr-2"
                                                            @click="
                                                                editProduct(
                                                                    slotProps.data,
                                                                )
                                                            "
                                                        />
                                                        <Button
                                                            icon="pi pi-trash"
                                                            outlined
                                                            rounded
                                                            severity="danger"
                                                            @click="
                                                                confirmDeleteProduct(
                                                                    slotProps.data,
                                                                )
                                                            "
                                                        />
                                                    </template>
                                                </Column>
                                            </DataTable>
                                        </DataTable>
                                    </Popover>
                                </div>
                            </template>
                        </column>
                    </DataTable>

                    <DataTable
                        class="p-datatable-sm"
                        v-model:expandedRows="batchExpandedRows"
                        :value="slotProps.data.batches"
                        dataKey="batchId"
                        v-if="slotProps.data.category === 'Non-Consumable'"
                    >
                        <Column
                            field="batchNumber"
                            header="Batch Number"
                            sortable
                        ></Column>
                        <Column
                            field="quantity"
                            header="Quantity"
                            sortable
                        ></Column>
                        <Column
                            field="purchaseDate"
                            header="Purchase Date"
                            sortable
                        ></Column>
                        <Column
                            field="purchasePrice"
                            header="Purchase Price"
                            sortable
                        >
                            <template #body="slotProps">
                                <span>{{
                                    formatPrice(slotProps.data.purchasePrice)
                                }}</span>
                            </template>
                        </Column>
                        <Column field="unit" header="Unit" sortable></Column>
                        <Column
                            field="supplier"
                            header="Supplier"
                            sortable
                        ></Column>
                        <Column
                            field="warranty"
                            header="Warranty"
                            sortable
                        ></Column>
                        <Column
                            field="minimumstocks"
                            header="Minimum Stocks"
                            sortable
                        ></Column>
                        <Column
                            field="stocklimit"
                            header="Stock Limit"
                            sortable
                        ></Column>
                        <Column field="status" header="Status" sortable>
                            <template #body="slotProps">
                                <Tag
                                    :value="slotProps.data.status"
                                    :severity="getOrderSeverity(slotProps.data)"
                                />
                            </template>
                        </Column>
                        <Column
                            field="Actions"
                            header="Actions"
                            :exportable="false"
                            style="min-width: 3rem"
                        >
                            <template #body="slotProps">
                                <div class="flex">
                                    <Button
                                        icon="pi pi-pencil"
                                        outlined
                                        rounded
                                        class="mr-2"
                                        @click="
                                            openNonConsumableBatchDialog(
                                                slotProps.data,
                                            )
                                        "
                                    />
                                    <Button
                                        icon="pi pi-trash"
                                        outlined
                                        rounded
                                        severity="danger"
                                        @click="
                                            confirmDeleteBatch(slotProps.data)
                                        "
                                    />
                                </div>
                            </template>
                        </Column>

                        <column header="Items">
                            <template #body="slotProps">
                                <div class="flex flex-wrap gap-2">
                                    <Button
                                        type="button"
                                        icon="pi pi-list"
                                        @click="toggleDataTable"
                                        rounded
                                        class="mr-2"
                                    />
                                    <Popover
                                        ref="op2"
                                        id="overlay_panel"
                                        style="width: 450px"
                                    >
                                        <DataTable
                                            v-model:selection="selectedProduct"
                                            :value="products"
                                            selectionMode="single"
                                            paginator
                                            :rows="5"
                                            :totalRecords="products.length"
                                            @row-select="onProductSelect"
                                        >
                                            <div
                                                class="flex items-center gap-2"
                                            >
                                                <h6>
                                                    Items for
                                                    {{
                                                        slotProps.data
                                                            .batchNumber
                                                    }}
                                                </h6>
                                                <Button
                                                    type="button"
                                                    icon="pi pi-filter-slash"
                                                    label="Clear"
                                                    outlined
                                                    @click="clearFilter()"
                                                />
                                                <IconField>
                                                    <InputIcon>
                                                        <i
                                                            class="pi pi-search"
                                                        />
                                                    </InputIcon>
                                                    <InputText
                                                        v-model="
                                                            filters['global']
                                                                .value
                                                        "
                                                        placeholder="Keyword Search"
                                                    />
                                                </IconField>
                                            </div>

                                            <DataTable
                                                class="p-datatable-xl"
                                                :value="
                                                    slotProps.data.serialNumbers
                                                "
                                            >
                                                <Column
                                                    field="serialNumber"
                                                    header="Serial Number"
                                                    sortable
                                                ></Column>
                                                <Column
                                                    field="status"
                                                    header="Status"
                                                    sortable
                                                ></Column>
                                                <Column
                                                    field="Assignedroom"
                                                    header="Assigned Room"
                                                    sortable
                                                ></Column>
                                                <Column
                                                    :exportable="false"
                                                    style="min-width: 9rem"
                                                >
                                                    <template #body="slotProps">
                                                        <Button
                                                            icon="pi pi-pencil"
                                                            outlined
                                                            rounded
                                                            class="mr-2"
                                                            @click="
                                                                editProduct(
                                                                    slotProps.data,
                                                                )
                                                            "
                                                        />
                                                        <Button
                                                            icon="pi pi-trash"
                                                            outlined
                                                            rounded
                                                            severity="danger"
                                                            @click="
                                                                confirmDeleteProduct(
                                                                    slotProps.data,
                                                                )
                                                            "
                                                        />
                                                    </template>
                                                </Column>
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

    <!-- Delete Product Dialog -->
    <Dialog
        v-model:visible="deleteProductDialog"
        :style="{ width: '450px' }"
        header="Confirm"
        :modal="true"
    >
        <div class="flex items-center gap-4">
            <i class="pi pi-exclamation-triangle !text-3xl" />
            <span v-if="product"
                >Are you sure you want to delete <b>{{ product.name }}</b
                >?</span
            >
        </div>
        <template #footer>
            <Button
                label="No"
                icon="pi pi-times"
                text
                @click="deleteProductDialog = false"
            />
            <Button label="Yes" icon="pi pi-check" @click="deleteProduct" />
        </template>
    </Dialog>

    <!-- Edit Product Dialog -->
    <Dialog
        :dismissableMask="true"
        v-model:visible="productDialog"
        :style="{ width: '450px' }"
        header="Product Details"
        :modal="true"
    >
        <div class="flex flex-col gap-6">
            <div>
                <label for="name" class="block font-bold mb-3">Name</label>
                <InputText
                    id="name"
                    v-model.trim="product.name"
                    required="true"
                    autofocus
                    :invalid="submitted && !product.name"
                    fluid
                />
                <small v-if="submitted && !product.name" class="text-red-500"
                    >Name is required.</small
                >
            </div>
            <div>
                <label for="brand" class="block font-bold mb-3">Brand</label>
                <InputText
                    id="brand"
                    v-model.trim="product.brand"
                    required="true"
                    autofocus
                    :invalid="submitted && !product.brand"
                    fluid
                />
                <small v-if="submitted && !product.brand" class="text-red-500"
                    >brand is required.</small
                >
            </div>
            <div>
                <label for="description" class="block font-bold mb-3"
                    >Description</label
                >
                <Textarea
                    id="description"
                    v-model="product.description"
                    required="true"
                    rows="3"
                    cols="20"
                    fluid
                />
            </div>
        </div>

        <template #footer>
            <Button
                label="Cancel"
                icon="pi pi-times"
                text
                @click="hideDialog"
            />
            <Button label="Save" icon="pi pi-check" @click="saveProduct" />
        </template>
    </Dialog>

    <!-- Delete batch Dialog -->
    <Dialog
        :dismissableMask="true"
        v-model:visible="deleteBatchDialog"
        :style="{ width: '450px' }"
        header="Confirm"
        :modal="true"
    >
        <div class="flex items-center gap-4">
            <i class="pi pi-exclamation-triangle !text-3xl" />
            <span v-if="selectedBatchForDeletion"
                >Are you sure you want to delete
                <b>{{ selectedBatchForDeletion.batchNumber }}</b
                >?</span
            >
        </div>
        <template #footer>
            <Button
                label="No"
                icon="pi pi-times"
                text
                @click="deleteBatchDialog = false"
            />
            <Button
                label="Yes"
                icon="pi pi-check"
                @click="confirmDeleteBatch(batch, product.id)"
            />
        </template>
    </Dialog>

    <!-- Edit  Consumable Batch  Details Dialog -->
    <Dialog
        :dismissableMask="true"
        v-model:visible="batchDialog"
        :style="{ width: '450px' }"
        header="Batch Details"
        :modal="true"
    >
        <div class="flex flex-col gap-6">
            <!-- Batch Number -->
            <div>
                <label for="batchNumber" class="block font-bold mb-3"
                    >Batch Number</label
                >
                <InputText
                    id="batchNumber"
                    v-model.trim="selectedBatch.batchNumber"
                    required="true"
                    autofocus
                    :invalid="submitted && !selectedBatch.batchNumber"
                    fluid
                />
                <small
                    v-if="submitted && !selectedBatch.batchNumber"
                    class="text-red-500"
                    >Batch Number is required.</small
                >
            </div>

            <!-- Quantity -->
            <div>
                <label for="quantity" class="block font-bold mb-3"
                    >Quantity</label
                >
                <InputText
                    id="quantity"
                    v-model.trim="selectedBatch.quantity"
                    required="true"
                    type="number"
                    :invalid="submitted && !selectedBatch.quantity"
                    fluid
                />
                <small
                    v-if="submitted && !selectedBatch.quantity"
                    class="text-red-500"
                    >Quantity is required.</small
                >
            </div>

            <!-- Purchase Date -->
            <div>
                <label for="purchaseDate" class="block font-bold mb-3"
                    >Purchase Date</label
                >
                <DatePicker
                    id="purchaseDate"
                    v-model="selectedBatch.purchaseDate"
                    required="true"
                    fluid
                />
                <small
                    v-if="submitted && !selectedBatch.purchaseDate"
                    class="text-red-500"
                    >Purchase Date is required.</small
                >
            </div>

            <!-- Purchase Price -->
            <div>
                <label for="purchasePrice" class="block font-bold mb-3"
                    >Purchase Price</label
                >
                <InputNumber
                    id="purchasePrice"
                    v-model="selectedBatch.purchasePrice"
                    mode="currency"
                    currency="USD"
                    required="true"
                    :invalid="submitted && !selectedBatch.purchasePrice"
                    fluid
                />
                <small
                    v-if="submitted && !selectedBatch.purchasePrice"
                    class="text-red-500"
                    >Purchase Price is required.</small
                >
            </div>

            <!-- Unit -->
            <div>
                <label for="unit" class="block font-bold mb-3">Unit</label>
                <Select
                    id="unit"
                    v-model="selectedBatch.unit"
                    :options="unitOptions"
                    optionLabel="label"
                    required="true"
                    placeholder="Select Unit"
                    fluid
                />
                <small
                    v-if="submitted && !selectedBatch.unit"
                    class="text-red-500"
                    >Unit is required.</small
                >
            </div>

            <!-- Supplier -->
            <div>
                <label for="supplier" class="block font-bold mb-3"
                    >Supplier</label
                >
                <InputText
                    id="supplier"
                    v-model="selectedBatch.supplier"
                    required="true"
                    fluid
                />
                <small
                    v-if="submitted && !selectedBatch.supplier"
                    class="text-red-500"
                    >Supplier is required.</small
                >
            </div>

            <!-- Expiration Date -->
            <div>
                <label for="expDate" class="block font-bold mb-3"
                    >Expiration Date</label
                >
                <DatePicker
                    id="expDate"
                    v-model="selectedBatch.expDate"
                    showIcon
                    required="true"
                    fluid
                />
                <small
                    v-if="submitted && !selectedBatch.expDate"
                    class="text-red-500"
                    >Expiration Date is required.</small
                >
            </div>
        </div>

        <!-- Footer -->
        <template #footer>
            <Button
                label="Cancel"
                icon="pi pi-times"
                text
                @click="hideDialog"
            />
            <Button label="Save" icon="pi pi-check" @click="saveBatchDetails" />
        </template>
    </Dialog>

    <!-- Edit  Non-Consumable Batch  Details Dialog -->
    <Dialog
        :dismissableMask="true"
        v-model:visible="NonConsumablebatchDialog"
        :style="{ width: '450px' }"
        header="Batch Details"
        :modal="true"
    >
        <div class="flex flex-col gap-6">
            <!-- Batch Number -->
            <div>
                <label for="batchNumber" class="block font-bold mb-3"
                    >Batch Number</label
                >
                <InputText
                    id="batchNumber"
                    v-model.trim="selectedBatch.batchNumber"
                    autofocus
                    fluid
                />
            </div>

            <!-- Quantity -->
            <div>
                <label for="quantity" class="block font-bold mb-3"
                    >Quantity</label
                >
                <InputText
                    id="quantity"
                    v-model.trim="selectedBatch.quantity"
                    type="number"
                    fluid
                />
            </div>

            <!-- Purchase Date -->
            <div>
                <label for="purchaseDate" class="block font-bold mb-3"
                    >Purchase Date</label
                >
                <DatePicker
                    id="purchaseDate"
                    v-model="selectedBatch.purchaseDate"
                    fluid
                />
            </div>

            <!-- Purchase Price -->
            <div>
                <label for="purchasePrice" class="block font-bold mb-3"
                    >Purchase Price</label
                >
                <InputNumber
                    id="purchasePrice"
                    v-model="selectedBatch.purchasePrice"
                    mode="currency"
                    currency="USD"
                    fluid
                />
            </div>

            <!-- Unit -->
            <div>
                <label for="unit" class="block font-bold mb-3">Unit</label>
                <Select
                    id="unit"
                    v-model="selectedBatch.unit"
                    :options="unitOptions"
                    optionLabel="label"
                    placeholder="Select Unit"
                    fluid
                />
            </div>

            <!-- Supplier -->
            <div>
                <label for="supplier" class="block font-bold mb-3"
                    >Supplier</label
                >
                <InputText
                    id="supplier"
                    v-model="selectedBatch.supplier"
                    fluid
                />
            </div>

            <!-- Warranty and Day/Month/Year Section -->
            <div class="grid grid-cols-2 items-start gap-4">
                <!-- Warranty Input and Slider -->
                <div class="flex flex-col gap-2">
                    <label for="batchNumber" class="font-medium"
                        >Warranty</label
                    >
                    <!-- Input Field for Warranty -->
                    <InputText
                        v-model.number="selectedBatch.warranty"
                        class="w-full"
                    />
                    <Slider
                        v-model.number="selectedBatch.warranty"
                        :min="0"
                        :max="100"
                        class="mt-2 w-full"
                    />
                </div>

                <div class="flex flex-col gap-2">
                    <label for="dropdown" class="font-medium"
                        >Select Duration</label
                    >
                    <Select
                        v-model="dropdownItem"
                        :options="dropdownItems"
                        optionLabel="name"
                        placeholder="Select Day / Month / Year"
                        class="w-full"
                    ></Select>
                </div>
            </div>
        </div>

        <!-- Footer -->
        <template #footer>
            <Button
                label="Cancel"
                icon="pi pi-times"
                text
                @click="hideDialog"
            />
            <Button
                label="Save"
                icon="pi pi-check"
                @click="saveNonConsumableBatchDetails"
            />
        </template>
    </Dialog>
</template>

<style scoped lang="scss">
:deep(.p-datatable-frozen-tbody) {
    font-weight: bold;
}

:deep(.p-datatable-scrollable .p-frozen-column) {
    font-weight: bold;
}
</style>
