<script setup lang="ts">
import { ref } from "vue";

// Sample room data
const roomCategories = ref([
    {
        id: 1,
        name: "Standard",
        description: "Basic amenities, ideal for budget stays",
    },
    {
        id: 2,
        name: "Deluxe",
        description: "Enhanced features for a comfortable experience",
    },
    { id: 3, name: "Suite", description: "Luxury rooms with premium services" },
]);

const roomTypes = ref([
    {
        id: 1,
        categoryId: 1,
        name: "Single",
        rates: { "6hrs": 30, "12hrs": 50, "24hrs": 80 },
    },
    {
        id: 2,
        categoryId: 1,
        name: "Double",
        rates: { "6hrs": 50, "12hrs": 80, "24hrs": 120 },
    },
    {
        id: 3,
        categoryId: 2,
        name: "King",
        rates: { "6hrs": 100, "12hrs": 150, "24hrs": 200 },
    },
    {
        id: 4,
        categoryId: 3,
        name: "Family",
        rates: { "6hrs": 200, "12hrs": 250, "24hrs": 300 },
    },
]);

// Selected category and dialog states
const selectedCategoryId = ref<number | null>(null);
const showAddCategoryDialog = ref(false);
const showSetPricingDialog = ref(false);

// Form states
const newCategory = ref({ name: "", description: "" });
const selectedRoomType = ref<{
    id: number;
    name: string;
    rates: { [key: string]: number };
} | null>(null);
const newRates = ref({ "6hrs": 0, "12hrs": 0, "24hrs": 0 });

// Methods
const setCategoryFilter = (categoryId: number | null) => {
    selectedCategoryId.value = categoryId;
};

const addCategory = () => {
    if (newCategory.value.name.trim() && newCategory.value.description.trim()) {
        const newId = roomCategories.value.length + 1;
        roomCategories.value.push({
            id: newId,
            name: newCategory.value.name,
            description: newCategory.value.description,
        });
        newCategory.value = { name: "", description: "" };
        showAddCategoryDialog.value = false;
    }
};

const openPricingDialog = (roomType: {
    id: number;
    name: string;
    rates: { [key: string]: number };
}) => {
    selectedRoomType.value = roomType;
    newRates.value = { ...roomType.rates };
    showSetPricingDialog.value = true;
};

const setPricing = () => {
    if (selectedRoomType.value) {
        const roomType = roomTypes.value.find(
            (type) => type.id === selectedRoomType.value?.id,
        );
        if (roomType) {
            roomType.rates = { ...newRates.value };
        }
        showSetPricingDialog.value = false;
        selectedRoomType.value = null;
    }
};
</script>

<template>
    <div class="p-6 bg-gray-50 dark:bg-gray-900 min-h-screen">
        <h1 class="text-2xl font-bold mb-4 text-gray-800 dark:text-gray-200">
            Room Rate Configuration
        </h1>

        <!-- Category Selector -->
        <div class="flex gap-4 mb-6">
            <button
                v-for="category in roomCategories"
                :key="category.id"
                @click="setCategoryFilter(category.id)"
                :class="[
                    selectedCategoryId === category.id
                        ? 'bg-blue-600'
                        : 'bg-blue-400 hover:bg-blue-500',
                    'px-4 py-2 rounded-lg text-white',
                ]"
            >
                {{ category.name }}
            </button>
            <button
                @click="setCategoryFilter(null)"
                class="px-4 py-2 bg-gray-400 text-white rounded-lg hover:bg-gray-500"
            >
                All Categories
            </button>
            <button
                @click="showAddCategoryDialog = true"
                class="px-4 py-2 bg-purple-500 text-white rounded-lg hover:bg-purple-600"
            >
                Add Category
            </button>
        </div>

        <!-- Room Type Table -->
        <div
            class="overflow-x-auto bg-white dark:bg-gray-800 shadow-md rounded-lg"
        >
            <table
                class="w-full border-collapse border border-gray-200 dark:border-gray-700"
            >
                <thead>
                    <tr class="bg-gray-100 dark:bg-gray-700">
                        <th
                            class="py-2 px-4 text-left text-gray-700 dark:text-gray-300"
                        >
                            Type
                        </th>
                        <th
                            class="py-2 px-4 text-left text-gray-700 dark:text-gray-300"
                        >
                            Category
                        </th>
                        <th
                            class="py-2 px-4 text-left text-gray-700 dark:text-gray-300"
                        >
                            Rates
                        </th>
                        <th
                            class="py-2 px-4 text-left text-gray-700 dark:text-gray-300"
                        >
                            Actions
                        </th>
                    </tr>
                </thead>
                <tbody>
                    <tr
                        v-for="type in roomTypes.filter((t) =>
                            selectedCategoryId === null
                                ? true
                                : t.categoryId === selectedCategoryId,
                        )"
                        :key="type.id"
                        class="border-b dark:border-gray-700"
                    >
                        <td class="py-2 px-4 text-gray-800 dark:text-gray-200">
                            {{ type.name }}
                        </td>
                        <td class="py-2 px-4 text-gray-800 dark:text-gray-200">
                            {{
                                roomCategories.find(
                                    (c) => c.id === type.categoryId,
                                )?.name
                            }}
                        </td>
                        <td class="py-2 px-4 text-gray-800 dark:text-gray-200">
                            6hrs: ${{ type.rates["6hrs"].toFixed(2) }}, 12hrs:
                            ${{ type.rates["12hrs"].toFixed(2) }}, 24hrs: ${{
                                type.rates["24hrs"].toFixed(2)
                            }}
                        </td>
                        <td class="py-2 px-4 text-gray-800 dark:text-gray-200">
                            <button
                                @click="openPricingDialog(type)"
                                class="px-2 py-1 bg-green-500 text-white rounded-lg hover:bg-green-600"
                            >
                                Set Pricing
                            </button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>

        <!-- Set Pricing Dialog -->
        <div
            v-if="showSetPricingDialog"
            class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center"
        >
            <div
                class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg w-96"
            >
                <h2
                    class="text-lg font-semibold mb-4 text-gray-800 dark:text-gray-200"
                >
                    Set Pricing for {{ selectedRoomType?.name || "" }}
                </h2>
                <div class="mb-4">
                    <label
                        for="6hrs"
                        class="block mb-1 text-gray-700 dark:text-gray-300"
                        >6hrs Price</label
                    >
                    <input
                        v-model.number="newRates['6hrs']"
                        id="6hrs"
                        type="number"
                        class="w-full px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg"
                    />
                </div>
                <div class="mb-4">
                    <label
                        for="12hrs"
                        class="block mb-1 text-gray-700 dark:text-gray-300"
                        >12hrs Price</label
                    >
                    <input
                        v-model.number="newRates['12hrs']"
                        id="12hrs"
                        type="number"
                        class="w-full px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg"
                    />
                </div>
                <div class="mb-4">
                    <label
                        for="24hrs"
                        class="block mb-1 text-gray-700 dark:text-gray-300"
                        >24hrs Price</label
                    >
                    <input
                        v-model.number="newRates['24hrs']"
                        id="24hrs"
                        type="number"
                        class="w-full px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg"
                    />
                </div>
                <div class="flex justify-end gap-2">
                    <button
                        @click="showSetPricingDialog = false"
                        class="px-4 py-2 bg-gray-400 text-white rounded-lg hover:bg-gray-500"
                    >
                        Cancel
                    </button>
                    <button
                        @click="setPricing"
                        class="px-4 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600"
                    >
                        Save
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>

<style lang="scss" scoped>
/* Add custom SCSS styles */
table {
    th,
    td {
        @apply border border-gray-200 dark:border-gray-700;
    }
    th {
        @apply bg-gray-50 dark:bg-gray-600 font-semibold text-sm uppercase tracking-wider;
    }
    td {
        @apply text-sm;
    }
}

textarea {
    resize: none;
}
</style>
