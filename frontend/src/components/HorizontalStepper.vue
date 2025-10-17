<template>
    <div class="bg-white dark:bg-card top-0 z-10 relative">
        <div class="flex items-center py-3 pt-4 px-4">
            <div class="flex items-center">
                <ol class="flex space-x-2 items-center">
                    <li
                        v-for="step in steps"
                        :key="step.id"
                        class="flex items-center"
                    >
                        <BaseButton
                            @click="navigateToStep(step.id)"
                            :disabled="!canNavigateTo(step.id)"
                            class="text-xs px-2 py-1 flex items-center gap-2 group"
                            :class="[
                                currentStep === step.id
                                    ? 'bg-sidebar-primary text-sidebar-primary-foreground'
                                    : 'text-foreground hover:bg-muted hover:text-foreground'
                            ]"
                        >
                            <span class="text-base">{{ step.title }}</span>
                        </BaseButton>
                        <div
                            v-if="step.id !== steps.length"
                            class="mx-2 flex items-center"
                        >
                            <svg class="w-4 h-4 text-sidebar-primary/50" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                            </svg>
                        </div>
                    </li>
                </ol>
            </div>
        </div>
    </div>
</template>
<script setup>
// no imports needed for defineProps and defineEmits in script setup
import BaseButton from "./BaseButton.vue";

const props = defineProps({
    currentStep: {
        type: Number,
        required: true,
    },
    steps: {
        type: Array,
        required: true,
    },
});

const emit = defineEmits(["navigate", "reset"]);

function navigateToStep(stepId) {
    if (canNavigateTo(stepId)) {
        emit("navigate", stepId);
    }
}

function reset() {
    emit("reset");
}

function canNavigateTo(stepId) {
    // allow navigation to the current step or any step that has been visited
    return (
        stepId === props.currentStep ||
        props.steps.find((step) => step.id === stepId)?.visited ||
        props.steps.find((step) => step.id === stepId - 1)?.completed
    );
}
</script>
