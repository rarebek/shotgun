<template>
    <div 
        class="group flex items-center justify-between p-3 bg-card border border-border rounded-[0.4rem] hover:bg-accent hover:border-accent-foreground transition-all duration-200 cursor-pointer"
        @click="handleOpenProject"
    >
        <div class="flex items-center gap-3 flex-1 min-w-0">
            <div class="flex-shrink-0">
                <svg class="w-5 h-5 text-muted-foreground" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-6l-2-2H5a2 2 0 00-2 2z"></path>
                </svg>
            </div>
            <div class="flex-1 min-w-0">
                <div class="font-medium text-foreground truncate">
                    {{ projectName }}
                </div>
                <div class="text-sm text-muted-foreground truncate">
                    {{ projectPath }}
                </div>
            </div>
        </div>
        
        <button
            @click.stop="handleRemoveProject"
            class="opacity-0 group-hover:opacity-100 p-2 text-muted-foreground hover:text-destructive hover:bg-destructive/10 rounded-[0.4rem] transition-all duration-200 flex-shrink-0 border border-border hover:border-destructive/50"
            title="Remove from recent projects"
        >
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
            </svg>
        </button>
    </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
    projectPath: {
        type: String,
        required: true
    }
})

const emit = defineEmits(['open', 'remove'])

const projectName = computed(() => {
    // Extract the folder name from the full path
    const pathParts = props.projectPath.split(/[/\\]/)
    return pathParts[pathParts.length - 1] || props.projectPath
})

const handleOpenProject = () => {
    emit('open', props.projectPath)
}

const handleRemoveProject = () => {
    emit('remove', props.projectPath)
}
</script>