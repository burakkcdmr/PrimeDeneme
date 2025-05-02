<script setup>
import BestSellingWidget from '@/components/dashboard/BestSellingWidget.vue';
import NotificationsWidget from '@/components/dashboard/NotificationsWidget.vue';
import WaitingTasks from '@/components/dashboard/WaitingTasks.vue';
import CompletedTask from '@/components/dashboard/CompletedTask.vue';
import ReportCards from '@/components/dashboard/ReportCards.vue';

import Crud from '@/views/pages/Crud.vue';
</script>

<template>
    <div class="grid grid-cols-12 gap-8">
        <ReportCards />

        <div class="col-span-12 xl:col-span-6">
            <WaitingTasks />
            <BestSellingWidget />
        </div>
        <div class="col-span-12 xl:col-span-6">
            <CompletedTask />
            <Crud />
        </div>
    </div>
</template>
