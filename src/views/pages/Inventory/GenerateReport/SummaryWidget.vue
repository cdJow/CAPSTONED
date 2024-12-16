<template>
    <div class="flex flex-wrap gap-4">
        <!-- Total Items Card -->
        <div class="card flex-1 min-w-[16rem]">
            <div class="flex justify-between mb-4">
                <div>
                    <span class="block text-muted-color font-medium mb-4">Total Items</span>
                    <div class="text-surface-900 dark:text-surface-0 font-medium text-xl">1,500</div>
                </div>
                <div class="flex items-center justify-center bg-blue-100 dark:bg-blue-400/10 rounded-border" style="width: 2.5rem; height: 2.5rem">
                    <i class="pi pi-box text-blue-500 !text-xl"></i>
                </div>
            </div>
            <span class="text-primary font-medium">120 </span>
            <span class="text-muted-color">low stock alerts</span>
        </div>

        <!-- Total Value Card -->
        <div class="card flex-1 min-w-[16rem]">
            <div class="flex justify-between mb-4">
                <div>
                    <span class="block text-muted-color font-medium mb-4">Total Inventory Value</span>
                    <div class="text-surface-900 dark:text-surface-0 font-medium text-xl">$25,000</div>
                </div>
                <div class="flex items-center justify-center bg-orange-100 dark:bg-orange-400/10 rounded-border" style="width: 2.5rem; height: 2.5rem">
                    <i class="pi pi-dollar text-orange-500 !text-xl"></i>
                </div>
            </div>
            <span class="text-primary font-medium">%10 </span>
            <span class="text-muted-color">increase this month</span>
        </div>

        <!-- Categories Card -->
        <div class="card flex-1 min-w-[16rem]">
            <div class="flex justify-between mb-4">
                <div>
                    <span class="block text-muted-color font-medium mb-4">Total Categories</span>
                    <div class="text-surface-900 dark:text-surface-0 font-medium text-xl">3</div>
                </div>
                <div class="flex items-center justify-center bg-cyan-100 dark:bg-cyan-400/10 rounded-border" style="width: 2.5rem; height: 2.5rem">
                    <i class="pi pi-tags text-cyan-500 !text-xl"></i>
                </div>
            </div>
            <span class="text-primary font-medium">Consumable, Non-Consumable, Rental</span>
        </div>

        <!-- Damaged Items Card -->
        <div class="card flex-1 min-w-[16rem]">
            <div class="flex justify-between mb-4">
                <div>
                    <span class="block text-muted-color font-medium mb-4">Damaged Items</span>
                    <div class="text-surface-900 dark:text-surface-0 font-medium text-xl">80</div>
                </div>
                <div class="flex items-center justify-center bg-red-100 dark:bg-red-400/10 rounded-border" style="width: 2.5rem; height: 2.5rem">
                    <i class="pi pi-exclamation-circle text-red-500 !text-xl"></i>
                </div>
            </div>
            <span class="text-primary font-medium">15 </span>
            <span class="text-muted-color">added this week</span>
        </div>
    </div>
</template>

<style scoped>
.card {
    padding: 1rem;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: white;
}
</style>
