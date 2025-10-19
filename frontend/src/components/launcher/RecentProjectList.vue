<template>
    <div class="recent-projects">
        <h2 class="text-xl font-semibold text-foreground mb-4 flex items-center gap-2">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"></path>
            </svg>
            recent projects
        </h2>
        
        <div v-if="loading" class="text-center py-8 text-muted-foreground">
            <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-primary mx-auto mb-2"></div>
            loading recent projects...
        </div>
        
        <div v-else-if="recentProjects.length === 0" class="text-center py-8 text-muted-foreground border-2 border-dashed border-border rounded-[0.4rem]">
            <svg class="w-12 h-12 mx-auto mb-2 opacity-50" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path>
            </svg>
            <p>no recent projects</p>
            <p class="text-sm mt-1">projects you open will appear here for quick access</p>
        </div>
        
        <div v-else class="space-y-2">
            <RecentProjectListItem
                v-for="project in recentProjects"
                :key="project"
                :project-path="project"
                @open="$emit('open-project', $event)"
                @remove="handleRemoveProject"
            />
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { GetRecentProjects, RemoveRecentProject } from '../../../wailsjs/go/main/App'
import RecentProjectListItem from './RecentProjectListItem.vue'

const emit = defineEmits(['open-project', 'project-removed'])

const recentProjects = ref([])
const loading = ref(true)

const loadRecentProjects = async () => {
    try {
        loading.value = true
        const projects = await GetRecentProjects()
        recentProjects.value = projects || []
    } catch (error) {
        console.error('Failed to load recent projects:', error)
        recentProjects.value = []
    } finally {
        loading.value = false
    }
}

const handleRemoveProject = async (projectPath) => {
    try {
        await RemoveRecentProject(projectPath)
        await loadRecentProjects() // Refresh the list
        emit('project-removed')
    } catch (error) {
        console.error('Failed to remove recent project:', error)
    }
}

onMounted(() => {
    loadRecentProjects()
})
</script>