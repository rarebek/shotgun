<template>
    <div>
        <CustomRulesModal
            ref="customIgnoreRulesModalRef"
            :is-visible="isCustomRulesModalVisible"
            :initial-rules="currentCustomRulesForModal"
            title="edit custom ignore rules"
            ruleType="ignore"
            @save="handleSaveCustomRules"
            @cancel="handleCancelCustomRules"
        />
        <CustomRulesModal
            ref="promptRulesModalRef"
            :is-visible="isPromptRulesModalVisible"
            :initial-rules="currentPromptRulesForModal_prompt"
            title="edit custom prompt rules"
            ruleType="prompt"
            @save="handleSavePromptRules_prompt"
            @cancel="handleCancelPromptRules_prompt"
        />
        <div
            class="sidebar-container flex item-top h-full"
        >
            <div
                class="sidebar-content w-64 lg:w-[450px] bg-sidebar p-4 border-sidebar-border flex flex-col flex-shrink-0 h-full max-[900px]:w-[415px]"
            >
                <!-- project selection and file tree -->
                <div class="flex flex-col flex-grow h-full">
                    <!-- project actions: open project & reset -->
                    <div
                        v-if="projectRoot"
                        class="mb-4 flex items-center gap-4 justify-between"
                    >
                        <BaseButton
                            @click="openPromptRulesModal_prompt"
                            title="edit custom prompt rules"
                            class="px-2 py-1"
                        >
                            <span class="text-base"> rules </span>
                        </BaseButton>
                        <BaseButton
                            @click="$emit('select-directory')"
                            class="flex-1 px-3 py-2 bg-sidebar-primary text-sidebar-primary-foreground text-base font-semibold rounded-md hover:bg-sidebar-primary/90 focus:outline-none"
                        >
                            <span class="text-base"> open another project </span>
                        </BaseButton>
                        <BaseButton
                            @click="handleReset"
                            :title="'reset application'"
                            variant="danger"
                            class="aspect-square text-base"
                        >
                            <!-- simple refresh icon -->
                            <span class="text-base"> reset </span>
                        </BaseButton>
                    </div>

                    <div
                        class="flex flex-row justify-between items-center mb-6"
                    >
                            <BaseButton
                                @click="selectAllFiles"
                                class="text-xs px-2 py-1 bg-sidebar-primary text-sidebar-primary-foreground rounded hover:bg-sidebar-primary/90"
                            >
                                <span class="text-base"> select all </span>
                            </BaseButton>
                            <BaseButton
                                @click="deselectAllFiles"
                                class="text-xs px-2 py-1"
                            >
                                <span class="text-base"> deselect </span>
                            </BaseButton>
                            <BaseButton
                                @click="resetFileSelections"
                                variant=""
                                class="text-xs px-2 py-1 bg-sidebar-primary text-sidebar-primary-foreground rounded hover:bg-sidebar-primary/90 disabled:opacity-50 disabled:cursor-not-allowed"
                            >
                                <span class="text-base"> default </span>
                            </BaseButton>
                            <BaseButton
                                @click="handleRefreshProject"
                                :disabled="isRefreshing"
                                title="refresh project files"
                            >
                                <span class="text-base"> refresh </span>
                            </BaseButton>
                    </div>

                    <!-- file tree -->
                    <div
                        class="rounded-[0.4rem] min-h-0 bg-white dark:bg-[#3a3b60] text-sm overflow-auto flex-grow h-0 border-2 border-accent"
                    >
                        <!-- use optimized file tree for better performance with large codebases -->
                        <FileTreeOptimized
                            v-if="useOptimizedFileTree && fileTreeNodes && fileTreeNodes.length > 0"
                            :nodes="fileTreeNodes"
                            :project-root="projectRoot"
                            :loading-error="loadingError"
                            :use-gitignore="useGitignore"
                            @toggle-exclude="handleToggleExclude"
                            @add-log="(log) => $emit('add-log', log)"
                        />
                        <!-- fallback to original file tree if needed -->
                        <FileTree
                            v-else-if="!useOptimizedFileTree && fileTreeNodes && fileTreeNodes.length > 0"
                            :nodes="fileTreeNodes"
                            :loading-error="loadingError"
                            :use-gitignore="useGitignore"
                            @toggle-exclude="handleToggleExclude"
                            @add-log="(log) => $emit('add-log', log)"
                        />
                        <div v-else-if="!projectRoot" class="flex items-center justify-center h-full p-3">
                            <span class="text-lg font-semibold">select a project folder to view files.</span>
                        </div>
                        <div
                            v-else-if="loadingError"
                            class="p-3 text-destructive"
                        >
                            error loading files: {{ loadingError }}
                        </div>
                        <div v-else class="p-3">loading files...</div>
                    </div>

                    <!-- <div class="mt-2 flex justify-around gap-4">
                        <label class="flex items-center text-sm">
                            <input
                                type="checkbox"
                                :checked="useGitignore"
                                @change="
                                    $emit(
                                        'toggle-gitignore',
                                        $event.target.checked
                                    )
                                "
                                class="form-checkbox h-4 w-4 text-sidebar-primary rounded border-border focus:ring-sidebar-primary mr-2"
                            />
                            <span class="text-base"> .gitignore rules </span>
                        </label> -->
                        <!-- <div class="flex items-center gap-2"> -->
                            <!-- <label class="flex items-center text-sm">
                                <input
                                    type="checkbox"
                                    :checked="useCustomIgnore"
                                    @change="
                                        $emit(
                                            'toggle-custom-ignore',
                                            $event.target.checked
                                        )
                                    "
                                    class="form-checkbox h-4 w-4 text-sidebar-primary rounded border-border focus:ring-sidebar-primary mr-2"
                                />
                                <span class="text-base"> custom ignore </span>
                            </label> -->
                            <!-- <BaseButton
                                @click="openCustomRulesModal"
                                title="edit custom ignore rules"
                                class="px-2 py-1 text-xs"
                                variant="secondary"
                            >
                                <span class="text-base"> edit </span>
                            </BaseButton> -->
                        <!-- </div> -->
                    <!-- </div> -->
                </div>
            </div>

        </div>
    </div>
</template>

<script setup>
import { ref } from "vue";
import FileTree from "./FileTree.vue"; // import the existing filetree
import FileTreeOptimized from "./FileTreeOptimized.vue"; // import optimized file tree for better performance
import CustomRulesModal from "./CustomRulesModal.vue";
import BaseButton from "./BaseButton.vue";

// flag to switch between normal and optimized file tree
const useOptimizedFileTree = ref(true);

// refs for modal components
const customIgnoreRulesModalRef = ref(null);
const promptRulesModalRef = ref(null);
import {
    GetCustomIgnoreRules,
    SetCustomIgnoreRules,
    GetCustomPromptRules,
    SetCustomPromptRules,
} from "../../wailsjs/go/main/App";
import {
    LogError as LogErrorRuntime,
    LogInfo as LogInfoRuntime,
} from "../../wailsjs/runtime/runtime";

/**
 * props for leftsidebar:
 * - usegitignore: enables .gitignore rules for file parsing
 * - usecustomignore: enables custom ignore.glob rules for file parsing
 */
const props = defineProps({
    currentStep: { type: Number, required: true },
    steps: { type: Array, required: true }, // array of { id: number, title: string, completed: boolean }
    projectRoot: { type: String, default: "" },
    fileTreeNodes: { type: Array, default: () => [] },
    useGitignore: { type: Boolean, default: true },
    useCustomIgnore: { type: Boolean, default: false },
    loadingError: { type: String, default: "" },
    isRefreshing: { type: Boolean, default: false },
});

const emit = defineEmits([
    "navigate",
    "toggle-gitignore",
    "toggle-custom-ignore",
    "toggle-exclude",
    "custom-rules-updated",
    "add-log",
    "select-all-files",
    "deselect-all-files",
    "reset-file-selections",
    "select-directory",
    "reset",
    "update:rulesContent",
    "refresh-project",
]);

const isCustomRulesModalVisible = ref(false);
const currentCustomRulesForModal = ref("");

// state for prompt rules modal
const isPromptRulesModalVisible = ref(false);
const currentPromptRulesForModal_prompt = ref("");


async function openCustomRulesModal() {
    try {
        currentCustomRulesForModal.value = await GetCustomIgnoreRules();
        isCustomRulesModalVisible.value = true;
    } catch (error) {
        console.error("error fetching custom ignore rules:", error);
        LogErrorRuntime(
            `error fetching custom rules: ${error.message || error}`
        );
        emit("add-log", {
            message: `failed to load custom rules: ${error.message || error}`,
            type: "error",
        });
        // show a placeholder or error message in the textarea if loading fails
        currentCustomRulesForModal.value =
            "# error loading rules. please check application logs.\n# you can still edit and save.";
        isCustomRulesModalVisible.value = true; // still open modal
    }
}

async function handleSaveCustomRules(newRules) {
    console.log("leftsidebar: handlesavecustomrules called, rules length:", newRules.length);
    try {
        console.log("leftsidebar: calling setcustomignorerules...");
        await SetCustomIgnoreRules(newRules);
        console.log("leftsidebar: setcustomignorerules completed successfully");

        // use nextTick to ensure modal state updates before closing
        console.log("leftsidebar: attempting to reset modal state");
        await new Promise(resolve => setTimeout(resolve, 0)); // let vue process any pending updates

        console.log("leftsidebar: closing modal");
        isCustomRulesModalVisible.value = false;

        LogInfoRuntime(
            "custom ignore rules saved successfully via leftsidebar."
        );
        emit("add-log", {
            message: "custom ignore rules saved.",
            type: "success",
        });
        emit("custom-rules-updated"); // notify mainlayout to refresh
    } catch (error) {
        console.error("leftsidebar: error saving custom ignore rules:", error);
        console.error("leftsidebar: error type:", typeof error);
        console.error("leftsidebar: error details:", JSON.stringify(error, null, 2));

        const errorMsg = error.message || String(error);

        // reset modal saving state with error
        if (customIgnoreRulesModalRef.value) {
            customIgnoreRulesModalRef.value.resetSavingState(false, errorMsg);
        }

        LogErrorRuntime(`error saving custom rules: ${errorMsg}`);
        emit("add-log", {
            message: `failed to save custom rules: ${errorMsg}`,
            type: "error",
        });
        // keep modal open on error so user can see what happened and try again
        alert(`failed to save custom rules: ${errorMsg}`);
    }
}

function handleCancelCustomRules() {
    isCustomRulesModalVisible.value = false;
}

async function openPromptRulesModal_prompt() {
    try {
        currentPromptRulesForModal_prompt.value = await GetCustomPromptRules();
        isPromptRulesModalVisible.value = true;
    } catch (error) {
        console.error("error fetching prompt rules for modal:", error);
        LogErrorRuntime(
            `error fetching prompt rules for modal: ${error.message || error}`
        );
        emit("add-log", {
            message: `failed to load prompt rules: ${error.message || error}`,
            type: "error",
        });
        currentPromptRulesForModal_prompt.value = "# error loading rules.";
        isPromptRulesModalVisible.value = true;
    }
}

async function handleSavePromptRules_prompt(newRules) {
    console.log("leftsidebar: handlesavepromptrules called, rules length:", newRules.length);
    try {
        console.log("leftsidebar: calling setcustompromptrules...");
        await SetCustomPromptRules(newRules);
        console.log("leftsidebar: setcustompromptrules completed successfully");

        // reset modal saving state and close
        if (promptRulesModalRef.value) {
            promptRulesModalRef.value.resetSavingState(true);
        }
        isPromptRulesModalVisible.value = false;

        LogInfoRuntime("custom prompt rules saved successfully.");
        emit("update:rulesContent", newRules);
    } catch (error) {
        console.error("leftsidebar: error saving prompt rules:", error);

        const errorMsg = error.message || String(error);

        // reset modal saving state with error
        if (promptRulesModalRef.value) {
            promptRulesModalRef.value.resetSavingState(false, errorMsg);
        }

        LogErrorRuntime(`error saving prompt rules: ${errorMsg}`);
        alert(`failed to save prompt rules: ${errorMsg}`);
    }
}

function handleCancelPromptRules_prompt() {
    isPromptRulesModalVisible.value = false;
}

function canNavigateToStep(stepId) {
    if (stepId === props.currentStep) return true;
    const targetStep = props.steps.find((s) => s.id === stepId);
    if (targetStep && targetStep.completed) return true;
    const firstUncompletedStep = props.steps.find((s) => !s.completed);
    const firstUncompletedStepId = firstUncompletedStep
        ? firstUncompletedStep.id
        : undefined;
    return (
        stepId === firstUncompletedStepId ||
        (firstUncompletedStepId === undefined && targetStep)
    ); // allow any if all completed
}

function selectAllFiles() {
    emit("select-all-files");
    emit("add-log", {
        message: "selecting all files",
        type: "info",
    });
}

function deselectAllFiles() {
    emit("deselect-all-files");
    emit("add-log", {
        message: "deselecting all files",
        type: "info",
    });
}

function resetFileSelections() {
    emit("reset-file-selections");
    emit("add-log", {
        message: "resetting file selections to default",
        type: "info",
    });
}

function handleToggleExclude(node) {
    console.log(
        `DEBUG: LeftSidebar received toggle-exclude for node: ${node.name}, path: ${node.relPath}`
    );
    emit("toggle-exclude", node);
}

function handleEditCustomRules() {
    openCustomRulesModal();
}

function handleReset() {
    emit("reset");
    emit("add-log", {
        message: "resetting application state",
        type: "info",
    });
}

function handleRefreshProject() {
    emit("refresh-project");
    emit("add-log", {
        message: "refreshing project files",
        type: "info",
    });
}
</script>

<style scoped>
.sidebar-container {
    position: relative;
}

</style>
