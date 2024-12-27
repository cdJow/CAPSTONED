<script setup>
import { ref, computed } from 'vue';

const rooms = ref([
    {
        id: 1,
        name: 'Cozy Single Room',
        type: 'Single Size',
        status: 'AVAILABLE',
        price: 100,
        image: 'https://via.placeholder.com/300x200.png?text=Single+Room',
    },
    {
        id: 2,
        name: 'Luxury Double Room',
        type: 'Double Size',
        status: 'AVAILABLE',
        price: 150,
        image: 'https://via.placeholder.com/300x200.png?text=Double+Room',
    },
    {
        id: 3,
        name: 'Queen Comfort Room',
        type: 'Queen Size',
        status: 'AVAILABLE',
        price: 200,
        image: 'https://via.placeholder.com/300x200.png?text=Queen+Room',
    },
    {
        id: 4,
        name: 'Elegant Double Room',
        type: 'Double Size',
        status: 'AVAILABLE',
        price: 160,
        image: 'https://via.placeholder.com/300x200.png?text=Double+Room',
    },
    {
        id: 5,
        name: 'Premium Queen Room',
        type: 'Queen Size',
        status: 'AVAILABLE',
        price: 220,
        image: 'https://via.placeholder.com/300x200.png?text=Premium+Queen+Room',
    },
]);

const searchQuery = ref('');
const layout = ref('list');
const options = ref(['list', 'grid']);

// Filter rooms based on search query for name, type, or price
const filteredRooms = computed(() => {
    return rooms.value.filter((room) => {
        const query = searchQuery.value.toLowerCase();
        return (
            room.name.toLowerCase().includes(query) ||
            room.type.toLowerCase().includes(query) ||
            room.price.toString().includes(query)
        );
    });
});

// Determines tag color based on room status
function getRoomTagColor(status) {
    switch (status) {
        case 'AVAILABLE':
            return 'success';
        case 'BOOKED':
            return 'warning';
        case 'MAINTENANCE':
            return 'danger';
        default:
            return null;
    }
}
</script>

<template>
    <div class="p-4 flex flex-col items-center">
        <!-- Search and Layout Options -->
        <div class="flex flex-col sm:flex-row items-center justify-center gap-4 mb-6 w-full max-w-3xl">
            <input
                v-model="searchQuery"
                type="text"
                placeholder="Search"
                class="border border-gray-300 rounded px-4 py-2 w-full sm:max-w-md"
            />
            <SelectButton v-model="layout" :options="options" :allowEmpty="false">
                <template #option="{ option }">
                    <i :class="[option === 'list' ? 'pi pi-bars' : 'pi pi-table']" />
                </template>
            </SelectButton>
        </div>

        <!-- Rooms List/Grid -->
        <div class="card w-full max-w-5xl">
            <DataView :value="filteredRooms" :layout="layout">
                <!-- List View -->
                <template #list="slotProps">
                    <div
                        v-for="(room, index) in slotProps.items"
                        :key="index"
                        class="p-4 border rounded mb-4 shadow"
                    >
                        <div class="flex flex-col md:flex-row gap-4">
                            <div class="md:w-40">
                                <img
                                    :src="room.image"
                                    alt="Room Image"
                                    class="rounded w-full h-32 object-cover"
                                />
                            </div>
                            <div class="flex-1">
                                <div class="text-xl font-bold">{{ room.name }}</div>
                                <div class="text-sm text-gray-600">{{ room.type }}</div>
                                <Tag
                                    :value="room.status"
                                    :severity="getRoomTagColor(room.status)"
                                    class="mt-2"
                                ></Tag>
                            </div>
                            <div class="flex flex-col items-end justify-between">
                                <div class="text-2xl font-bold">${{ room.price }} / night</div>
                                <Button
                                    label="Book Now"
                                    class="bg-red-500 text-white px-4 py-2 rounded mt-4 hover:bg-red-600"
                                ></Button>
                            </div>
                        </div>
                    </div>
                </template>

                <!-- Grid View -->
                <template #grid="slotProps">
                    <div class="grid grid-cols-12 gap-4">
                        <div
                            v-for="(room, index) in slotProps.items"
                            :key="index"
                            class="col-span-12 sm:col-span-6 lg:col-span-4 p-4 border rounded shadow"
                        >
                            <img
                                :src="room.image"
                                alt="Room Image"
                                class="rounded w-full h-40 object-cover mb-4"
                            />
                            <div class="text-lg font-bold mb-1">{{ room.name }}</div>
                            <div class="text-sm text-gray-600 mb-2">{{ room.type }}</div>
                            <Tag :value="room.status" :severity="getRoomTagColor(room.status)"></Tag>
                            <div class="text-xl font-semibold mt-4">${{ room.price }} / night</div>
                            <Button
                                label="Book Now"
                                class="bg-red-500 text-white w-full mt-4 hover:bg-red-600"
                            ></Button>
                        </div>
                    </div>
                </template>
            </DataView>
        </div>
    </div>
</template>
