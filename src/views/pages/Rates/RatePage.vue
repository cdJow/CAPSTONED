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
    { id: 1, categoryId: 1, name: "Single", baseRate: 50 },
    { id: 2, categoryId: 1, name: "Double", baseRate: 80 },
    { id: 3, categoryId: 2, name: "King", baseRate: 150 },
    { id: 4, categoryId: 3, name: "Family", baseRate: 250 },
]);

// Selected category and dialog states
const selectedCategoryId = ref<number | null>(null);
const showAddCategoryDialog = ref(false);
const showSetPricingDialog = ref(false);

// Form states
const newCategory = ref({ name: "", description: "" });
const selectedRoomType = ref<{ id: number; baseRate: number } | null>(null);
const newPrice = ref<number | null>(null);

// Methods
const setCategoryFilter = (categoryId: number | null) => {
    selectedCategoryId.value = categoryId;
};

const applyDynamicPricing = () => {
    roomTypes.value.forEach((type) => {
        type.baseRate += Math.random() * 10; // Example adjustment logic
    });
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

const openPricingDialog = (roomType: { id: number; baseRate: number }) => {
    selectedRoomType.value = roomType;
    newPrice.value = roomType.baseRate;
    showSetPricingDialog.value = true;
};

const setPricing = () => {
    if (selectedRoomType.value && newPrice.value !== null) {
        const roomType = roomTypes.value.find(
            (type) => type.id === selectedRoomType.value?.id,
        );
        if (roomType) {
            roomType.baseRate = newPrice.value;
        }
        showSetPricingDialog.value = false;
        selectedRoomType.value = null;
        newPrice.value = null;
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
                            Base Rate
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
                            ${{ type.baseRate.toFixed(2) }}
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

        <!-- Add Category Dialog -->
        <div
            v-if="showAddCategoryDialog"
            class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center"
        >
            <div
                class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg w-96"
            >
                <h2
                    class="text-lg font-semibold mb-4 text-gray-800 dark:text-gray-200"
                >
                    Add New Category
                </h2>
                <div class="mb-4">
                    <label
                        for="name"
                        class="block mb-1 text-gray-700 dark:text-gray-300"
                        >Category Name</label
                    >
                    <input
                        v-model="newCategory.name"
                        id="name"
                        type="text"
                        class="w-full px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg"
                    />
                </div>
                <div class="mb-4">
                    <label
                        for="description"
                        class="block mb-1 text-gray-700 dark:text-gray-300"
                        >Description</label
                    >
                    <textarea
                        v-model="newCategory.description"
                        id="description"
                        rows="3"
                        class="w-full px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg"
                    ></textarea>
                </div>
                <div class="flex justify-end gap-2">
                    <button
                        @click="showAddCategoryDialog = false"
                        class="px-4 py-2 bg-gray-400 text-white rounded-lg hover:bg-gray-500"
                    >
                        Cancel
                    </button>
                    <button
                        @click="addCategory"
                        class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600"
                    >
                        Add
                    </button>
                </div>
            </div>
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
                        for="price"
                        class="block mb-1 text-gray-700 dark:text-gray-300"
                        >New Price</label
                    >
                    <input
                        v-model="newPrice"
                        id="price"
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
