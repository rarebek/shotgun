<template>
    <div
        v-if="isVisible"
        class="fixed inset-0 bg-black/50 backdrop-blur-sm overflow-y-auto h-full w-full z-50 flex justify-center items-center"
        @click.self="handleCancel"
    >
        <div
            class="relative mx-auto p-5 border w-full max-w-2xl shadow-lg rounded-[0.4rem] bg-card border-border"
        >
            <div class="mt-3 text-center">
                <h3
                    class="text-lg leading-6 font-medium text-card-foreground"
                >
                    {{ title }}
                </h3>
                <div class="mt-2 px-7 py-3">
                    <textarea
                        v-model="editableRules"
                        rows="15"
                        spellcheck="false"
                        class="w-full p-2 border border-border rounded-md shadow-sm focus:ring-primary focus:border-primary text-sm font-mono bg-background text-foreground"
                        :placeholder="`enter ${ruleType} patterns here...`"
                    ></textarea>
                    <p
                        class="text-sm text-muted-foreground mt-1 text-left"
                    >
                        {{ descriptionText }}
                    </p>
                    <p
                        v-if="saveError"
                        class="text-sm text-red-500 mt-2 text-left"
                    >
                        error: {{ saveError }}
                    </p>
                </div>
                <div class="items-center px-4 py-3">
                    <BaseButton
                        @click="handleSave"
                        :disabled="isSaving"
                        class="px-4 py-2 mr-2 bg-sidebar-primary text-sidebar-primary-foreground text-base font-semibold rounded-md hover:bg-sidebar-primary/90 focus:outline-none"
                    >
                        <span class="text-base"> {{ isSaving ? 'saving...' : 'save' }} </span>
                    </BaseButton>
                    <BaseButton
                        @click="handleCancel"
                        :disabled="isSaving"
                        class="px-4 py-2"
                    >
                        <span class="text-base"> cancel </span>
                    </BaseButton>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, watch, computed } from "vue";
import BaseButton from './BaseButton.vue';

const props = defineProps({
    isVisible: {
        type: Boolean,
        required: true,
    },
    initialRules: {
        type: String,
        default: "",
    },
    title: {
        type: String,
        default: "edit custom rules",
    },
    ruleType: {
        type: String,
        required: true,
        validator: (value) => ["ignore", "prompt"].includes(value),
    },
});

const emit = defineEmits(["save", "cancel", "save-complete", "save-error"]);

const editableRules = ref("");
const isSaving = ref(false);
const saveError = ref("");

const descriptionText = computed(() => {
    if (props.ruleType === "prompt") {
        return "these rules provide specific instructions or pre-defined text for the ai. they will be included in the final prompt.";
    }
    // default to the description for ignore rules
    return 'these rules use .gitignore pattern syntax. they are applied globally when "use custom rules" is checked.';
});

watch(
    () => props.initialRules,
    (newVal) => {
        editableRules.value = newVal;
        saveError.value = ""; // clear error when rules change
    },
    { immediate: true }
);

watch(
    () => props.isVisible,
    (newVal, oldVal) => {
        console.log(`customrulesmodal: isVisible changed from ${oldVal} to ${newVal}`);
        console.log(`customrulesmodal: current isSaving state: ${isSaving.value}`);

        if (newVal) {
            // when modal becomes visible, ensure textarea reflects the latest initialrules
            console.log("customrulesmodal: modal opening, resetting state");
            editableRules.value = props.initialRules;
            saveError.value = ""; // clear error when modal opens
            isSaving.value = false; // reset saving state
            console.log(`customrulesmodal: after reset, isSaving: ${isSaving.value}`);
        } else {
            // when modal closes, reset saving state for next time
            console.log("customrulesmodal: modal closing, resetting state");
            isSaving.value = false;
            saveError.value = "";
            console.log(`customrulesmodal: after reset on close, isSaving: ${isSaving.value}`);
        }
    }
);

function handleSave() {
    console.log("customrulesmodal: handlesave called with rules length:", editableRules.value.length);
    isSaving.value = true;
    saveError.value = "";
    emit("save", editableRules.value);
}

// expose method to parent to reset saving state
defineExpose({
    resetSavingState: (success = true, errorMessage = "") => {
        console.log("customrulesmodal: resetsavingstate called, success:", success);
        isSaving.value = false;
        if (!success) {
            saveError.value = errorMessage;
        }
    }
});

function handleCancel() {
    console.log("customrulesmodal: handlecancel called");
    emit("cancel");
}
</script>

<style scoped>
/* basic styling for modal, can be enhanced with tailwind further if needed */
</style>
