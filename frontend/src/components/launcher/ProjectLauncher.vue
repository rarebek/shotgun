<template>
    <div class="flex flex-col items-center justify-center h-full p-8 bg-background">
        <!-- Logo/Title Section -->
        <div class="mb-12 text-center">
            <div class="mb-4">
                <!-- Placeholder for shotgun logo - you can replace with actual logo -->
                <div class="w-24 h-24 mx-auto mb-4 bg-gradient-to-br from-blue-500 to-purple-600 rounded-2xl flex items-center justify-center shadow-lg">
                    <span class="text-white text-3xl font-bold">SG</span>
                </div>
            </div>
            <h1 class="text-4xl font-bold text-foreground mb-2">shotgun</h1>
            <p class="text-muted-foreground text-lg">prompt engineering and project analysis tool</p>
        </div>

        <!-- Action Buttons -->
        <div class="flex flex-col gap-4 mb-12">
            <BaseButton @click="$emit('open-dialog')" class="h-14 px-12 py-5 text-xl min-w-80">
                <span class="flex items-center gap-2">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-6l-2-2H5a2 2 0 00-2 2z"></path>
                    </svg>
                    open project
                </span>
            </BaseButton>
        </div>

        <!-- Recent Projects Section -->
        <div class="w-full max-w-2xl">
            <RecentProjectList 
                @open-project="(path) => $emit('open-project', path)"
                @project-removed="$emit('project-removed')"
            />
        </div>
    </div>
</template>

<script setup>
import BaseButton from '../BaseButton.vue'
import RecentProjectList from './RecentProjectList.vue'

defineEmits(['open-dialog', 'open-project', 'project-removed'])
</script>