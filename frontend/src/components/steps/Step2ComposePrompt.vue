<template>
    <div class="p-4 h-full flex flex-col">
        <!-- error display for context generation failures -->
        <div
            v-if="isErrorContext"
            class="mb-4 p-4 border-2 border-red-300 dark:border-red-700 rounded-[0.4rem] bg-red-50 dark:bg-red-900 dark:bg-opacity-20 shadow-sm"
        >
            <h4
                class="text-lg font-semibold mb-2 text-red-600 dark:text-red-400"
            >
                context generation error
            </h4>
            <pre
                class="text-sm whitespace-pre-wrap bg-white dark:bg-dark-surface text-gray-900 dark:text-gray-100 p-3 border-2 border-red-200 dark:border-red-700 rounded-[0.4rem] overflow-auto max-h-[150px]"
                >{{ errorMessage }}</pre
            >
            <p class="mt-3 text-sm text-red-600 dark:text-red-400">
                go back to step 1 to reduce the project scope by excluding more
                files or using a smaller project
            </p>
        </div>

        <!-- custom rules modal removed per user request -->
        <div class="flex-grow flex flex-col space-y-4 overflow-hidden">
            <!-- user query input section - positioned at top in vertical layout -->
            <div
                class="w-full h-[32.4rem] lg:h-[42.1rem] flex flex-col space-y-2 overflow-y-hidden px-2 py-2 border-2 border-accent rounded-[0.4rem] bg-white dark:bg-[#3a3b60]"
            >
                <div class="flex flex-col flex-grow-[3]">
                    <!-- <label
                        for="user-task-ai"
                        class="block text-base font-medium text-gray-700 dark:text-gray-300 mb-1"
                        >your query for ai:</label
                    > -->
                    <textarea
                        id="user-task-ai"
                        v-model="localUserTask"
                        spellcheck="false"
                        class="w-full p-2 border-2 border-accent rounded-[0.4rem] shadow-sm focus:ring-light-accent dark:focus:ring-dark-accent focus:border-light-accent dark:focus:border-dark-accent text-lg bg-white dark:bg-dark-surface text-gray-900 dark:text-gray-100 flex-grow resize-none"
                        placeholder="describe what the ai should do..."
                    ></textarea>
                </div>

                <!-- custom rules textarea commented out per user request -->
                <!-- <div class="flex flex-col flex-grow-[1]">
                    <label
                        for="file-list-context"
                        class="text-base font-medium text-gray-700 dark:text-gray-300 mb-1 flex items-center"
                    >
                        file list context:
                    </label>
                    <textarea
                        id="file-list-context"
                        :value="props.fileListContext"
                        readonly
                        spellcheck="false"
                        class="w-full p-2 border border-accent rounded-md shadow-sm bg-gray-100 dark:bg-dark-surface font-mono text-sm text-gray-900 dark:text-gray-100 flex-grow min-h-[50px]"
                        placeholder="file list from step 1 (prepare context) will appear here..."
                    ></textarea>
                </div> -->
            </div>

            <!-- generated prompt display section - positioned at bottom and takes remaining space -->
            <div
                class="w-full flex-1 flex flex-col overflow-y-auto p-2 border-2 border-accent rounded-[0.4rem] bg-white dark:bg-[#3a3b60]"
            >
                <div class="flex justify-between items-center mb-2">
                    <div class="flex items-center space-x-2">
                        <div class="flex flex-row space-x-2">
                            <!-- mode selector: buttons for large screens, dropdown for small screens -->
                            <template v-if="!isSmallScreen">
                                <BaseButton
                                    v-for="(template, key) in promptTemplates"
                                    :key="key"
                                    @click="selectedPromptTemplateKey = key"
                                    :class="[
                                        'text-sm flex items-center font-semibold px-4 py-2.5',
                                        selectedPromptTemplateKey === key
                                            ? 'bg-sidebar-primary text-sidebar-primary-foreground'
                                            : '',
                                    ]"
                                    :disabled="isLoadingFinalPrompt"
                                    :title="template.name"
                                >
                                    <span class="font-bold">
                                        {{ getShortName(key) }}
                                    </span>
                                </BaseButton>
                            </template>
                            <template v-else>
                                <div class="relative inline-block" style="width: 200px;" ref="dropdownRef">
                                    <button
                                        @click="isDropdownOpen = !isDropdownOpen"
                                        :disabled="isLoadingFinalPrompt"
                                        class="appearance-none pr-8 p-2 text-sm flex items-center font-semibold focus:outline-none bg-background dark:bg-[#2a2a48] focus-visible:ring-primary text-gray-900 dark:text-gray-200 w-full border-2 border-accent rounded-[0.4rem]"
                                        ref="dropdownBtn"
                                    >
                                        {{ getShortName(selectedPromptTemplateKey) }}
                                    </button>
                                    <!-- chevron arrow icon -->
                                    <svg
                                        class="absolute right-2 top-1/2 -translate-y-1/2 pointer-events-none h-4 w-4 text-gray-500 dark:text-gray-300"
                                        xmlns="http://www.w3.org/2000/svg"
                                        viewBox="0 0 20 20"
                                        fill="currentColor"
                                    >
                                        <path
                                            fill-rule="evenodd"
                                            d="M5.23 7.21a.75.75 0 011.06.02L10 10.92l3.71-3.69a.75.75 0 111.06 1.06l-4.24 4.25a.75.75 0 01-1.06 0L5.23 8.29a.75.75 0 01.02-1.08z"
                                            clip-rule="evenodd"
                                        />
                                    </svg>
                                    <!-- custom dropdown menu - using fixed positioning to escape overflow-hidden parent -->
                                    <div
                                        v-if="isDropdownOpen"
                                        :style="dropdownMenuStyles"
                                        class="fixed bg-background dark:bg-[#2a2a48] border-2 border-accent rounded-[0.4rem] shadow-lg z-50 max-h-48 overflow-y-auto"
                                    >
                                        <button
                                            v-for="(template, key) in promptTemplates"
                                            :key="key"
                                            @click="selectedPromptTemplateKey = key; isDropdownOpen = false;"
                                            :class="[
                                                'w-full px-4 py-2 text-left text-sm font-semibold hover:bg-muted transition-colors',
                                                selectedPromptTemplateKey === key ? 'bg-sidebar-primary text-sidebar-primary-foreground' : 'text-foreground'
                                            ]"
                                        >
                                            {{ getShortName(key) }}
                                        </button>
                                    </div>
                                </div>
                            </template>
                        </div>
                    </div>
                    <!-- <div class="flex flex-row space-x-2">
                        <span
                            v-if="isCountingTokens"
                            class="text-sm text-gray-500"
                        >
                            counting...
                        </span>
                        <span
                            v-else-if="tokenCountError"
                            class="text-sm font-bold px-2 py-1 rounded-xl bg-red-100 dark:bg-red-900/30 text-red-600 dark:text-red-400 max-w-[300px] truncate"
                            :title="tokenCountError"
                        >
                            {{ tokenCountError }}
                        </span>
                        <span
                            v-else
                            :class="[
                                'text-sm font-bold px-2 py-1 ml-2 rounded-xl',
                                charCountColorClass === 'text-green-600'
                                    ? 'bg-green-100 dark:bg-green-900/30'
                                    : charCountColorClass === 'text-yellow-500'
                                      ? 'bg-yellow-100 dark:bg-yellow-900/30'
                                      : 'bg-red-100 dark:bg-red-900/30',
                            ]"
                            :title="tooltipText"
                        >
                            {{ geminiTokenCount.toLocaleString() }}
                        </span>
                    </div> -->
                    <div class="flex items-center space-x-3">
                        <BaseButton
                            @click="copyFinalPromptToClipboard"
                            :disabled="
                                !props.finalPrompt || isLoadingFinalPrompt
                            "
                            class="px-3 py-2 bg-sidebar-primary text-sidebar-primary-foreground text-base font-semibold rounded-[0.4rem] hover:bg-sidebar-primary/90 focus:outline-none disabled:bg-gray-300 dark:disabled:bg-gray-700 flex items-center gap-1"
                            :class="{
                                'bg-green-600 dark:bg-green-700': copySuccess,
                            }"
                        >
                            <template #icon>
                                <svg
                                    v-if="!copySuccess"
                                    xmlns="http://www.w3.org/2000/svg"
                                    class="h-4 w-4"
                                    fill="none"
                                    viewBox="0 0 24 24"
                                    stroke="currentColor"
                                >
                                    <path
                                        stroke-linecap="round"
                                        stroke-linejoin="round"
                                        stroke-width="2"
                                        d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2m0 0h2a2 2 0 012 2v3m2 4H10m0 0l3-3m-3 3l3 3"
                                    />
                                </svg>
                                <svg
                                    v-else
                                    xmlns="http://www.w3.org/2000/svg"
                                    class="h-4 w-4"
                                    fill="none"
                                    viewBox="0 0 24 24"
                                    stroke="currentColor"
                                >
                                    <path
                                        stroke-linecap="round"
                                        stroke-linejoin="round"
                                        stroke-width="2"
                                        d="M5 13l4 4L19 7"
                                    />
                                </svg>
                            </template>
                            <span class="text-base">{{ copyButtonText }}</span>
                        </BaseButton>
                    </div>
                </div>

                <div
                    v-if="isLoadingFinalPrompt"
                    class="flex-grow flex justify-center items-center"
                >
                    <div
                        class="animate-spin rounded-full h-8 w-8 border-b-2 border-light-accent dark:border-dark-accent"
                    ></div>
                    <p class="text-gray-500 dark:text-gray-300 ml-2">
                        updating prompt...
                    </p>
                </div>

                <div
                    v-else-if="props.finalPrompt"
                    class="flex items-start justify-center"
                >
                    <div
                        class="w-full px-4 py-2 border-2 border-accent rounded-[0.4rem] bg-gray-50 dark:bg-[#3a3b60] shadow-sm"
                    >
                        <div class="flex flex-col lg:flex-row items-start lg:items-center justify-between gap-4">
                            <div class="flex items-center gap-2">
                                <div class="flex flex-col">
                                    <h3
                                        class="text-base font-bold text-gray-800 dark:text-gray-200"
                                    >
                                        prompt ready
                                    </h3>
                                    <span
                                        class="text-xs text-gray-500 dark:text-gray-400"
                                    >
                                        stored in memory and ready to paste
                                    </span>
                                </div>
                            </div>

                            <div class="flex flex-col lg:flex-row items-start lg:items-center gap-2 lg:gap-4">
                                <div class="flex items-center gap-2">
                                    <span
                                        class="text-xs text-gray-500 dark:text-gray-400 uppercase"
                                        >characters</span
                                    >
                                    <span
                                        class="text-xl font-bold text-gray-800 dark:text-gray-200"
                                        >{{ charCount.toLocaleString() }}</span
                                    >
                                </div>
                                <div class="hidden lg:block h-6 w-px bg-accent"></div>
                                <div class="flex items-center gap-2">
                                    <span
                                        class="text-xs text-gray-500 dark:text-gray-400 uppercase"
                                        >size</span
                                    >
                                    <span
                                        class="text-xl font-bold text-gray-800 dark:text-gray-200"
                                        >{{
                                            (charCount / 1024).toFixed(1)
                                        }}
                                        kb</span
                                    >
                                </div>
                                <div class="hidden lg:block h-6 w-px bg-accent"></div>
                                <div class="flex items-center gap-2">
                                    <span
                                        class="text-xs text-gray-500 dark:text-gray-400 uppercase"
                                        >token</span
                                    >
                                    <span
                                        v-if="isCountingTokens"
                                        class="text-xl font-bold text-gray-500 dark:text-gray-400 animate-pulse"
                                        >recounting</span
                                    >
                                    <span
                                        v-else-if="tokenCountError"
                                        class="text-xl font-bold text-red-600 dark:text-red-400"
                                        :title="tokenCountError"
                                        >error</span
                                    >
                                    <span
                                        v-else
                                        class="text-xl font-bold text-gray-800 dark:text-gray-200"
                                        >{{ geminiTokenCount.toLocaleString() }}</span
                                    >
                                </div>
                                <div class="hidden lg:block h-6 w-px bg-accent"></div>
                                <div class="flex items-center gap-2">
                                    <span
                                        class="text-xs text-gray-500 dark:text-gray-400 uppercase"
                                        >cost</span
                                    >
                                    <span
                                        v-if="isCountingTokens"
                                        class="text-xl font-bold text-gray-500 dark:text-gray-400 animate-pulse"
                                        >recounting</span
                                    >
                                    <span
                                        v-else-if="tokenCountError"
                                        class="text-xl font-bold text-red-600 dark:text-red-400"
                                        :title="tokenCountError"
                                        >error</span
                                    >
                                    <span
                                        v-else
                                        class="text-xl font-bold text-gray-800 dark:text-gray-200"
                                        >{{ promptCost.toFixed(4) }} $</span
                                    >
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <div v-else class="flex-grow flex justify-center items-center">
                    <p class="text-gray-500 dark:text-gray-300">
                        prompt will be generated automatically
                    </p>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, watch, onMounted, computed, onUnmounted } from "vue";
import { LogError as LogErrorRuntime } from "../../../wailsjs/runtime/runtime";
import BaseButton from "../BaseButton.vue";

import devTemplateContentFromFile from "../../../../design/prompts/prompt_makeDiffGitFormat.md?raw";
import architectTemplateContentFromFile from "../../../../design/prompts/prompt_makePlan.md?raw";
import findBugTemplateContentFromFile from "../../../../design/prompts/prompt_analyzeBug.md?raw";
import projectManagerTemplateContentFromFile from "../../../../design/prompts/prompt_projectManager.md?raw";
import promptEnhancerTemplateContentFromFile from "../../../../design/prompts/prompt_promptEnhancer.md?raw";
import noneTemplateContentFromFile from "../../../../design/prompts/prompt_none.md?raw";

const props = defineProps({
    fileListContext: {
        type: String,
        default: "",
    },
    platform: {
        type: String,
        default: "unknown",
    },
    userTask: {
        type: String,
        default: "",
    },
    rulesContent: {
        type: String,
        default: "",
    },
    finalPrompt: {
        type: String,
        default: "",
    },
    geminiTokenCount: {
        type: Number,
        default: 0,
    },
    isCountingTokens: {
        type: Boolean,
        default: false,
    },
    tokenCountError: {
        type: String,
        default: "",
    },
    promptCost: {
        type: Number,
        default: 0,
    },
});

const emit = defineEmits([
    "update:finalPrompt",
    "update:userTask",
    "update:rulesContent",
]);

const promptTemplates = {
    none: { name: "custom query", content: noneTemplateContentFromFile },
    promptEnhancer: {
        name: "prompt engineer of your task",
        content: promptEnhancerTemplateContentFromFile,
    },
    architect: {
        name: "strategic planner and designer",
        content: architectTemplateContentFromFile,
    },
    dev: { name: "builder of your plan", content: devTemplateContentFromFile },
    findBug: {
        name: "checker of the known & new bugs",
        content: findBugTemplateContentFromFile,
    },
    projectManager: {
        name: "project scanner & analyzer of implementation",
        content: projectManagerTemplateContentFromFile,
    },
};

// helper functions for template icons and short names
function getTemplateIcon(key) {
    const icons = {
        dev: "",
        architect: "",
        findBug: "",
        projectManager: "",
        promptEnhancer: "",
    };
    return icons[key] || "";
}

function getShortName(key) {
    const shortNames = {
        none: "RAW PROMPT",
        promptEnhancer: "ENHANCE PROMPT",
        dev: "CODE DIFF",
        architect: "BUILD PLAN",
        findBug: "ANALYZE BUG",
        projectManager: "REFLECT ON",
    };
    return shortNames[key] || key;
}

const selectedPromptTemplateKey = ref(Object.keys(promptTemplates)[0]); // default to first template

const isLoadingFinalPrompt = ref(false);
const copyButtonText = ref("copy");
const copySuccess = ref(false);
const isDropdownOpen = ref(false);
const dropdownRef = ref(null);
const dropdownBtn = ref(null);

// computed dropdown menu position for fixed positioning
const dropdownMenuStyles = computed(() => {
    if (!dropdownBtn.value || !isDropdownOpen.value) {
        return { display: 'none' };
    }

    const rect = dropdownBtn.value.getBoundingClientRect();
    return {
        left: rect.left + 'px',
        top: (rect.top - 192) + 'px', // 192px = max-h-48 (12rem * 16px)
        width: rect.width + 'px',
    };
});

// refresh button state
const refreshing = ref(false);

let finalPromptDebounceTimer = null;
let userTaskInputDebounceTimer = null;

// modal state for prompt rules removed
// const isPromptRulesModalVisible = ref(false);
// const currentPromptRulesForModal = ref("");

const isFirstMount = ref(true);

const localUserTask = ref(props.userTask);
// track if prompt needs regeneration to avoid unnecessary updates
const promptNeedsUpdate = ref(false);
// manual trigger for prompt generation
const shouldGeneratePrompt = ref(false);

// Error detection (same logic as Step 1)
const isErrorContext = computed(() => {
    if (!props.fileListContext) return false;
    // consider only the first non-blank line to decide if the backend sent an error
    const firstLine = props.fileListContext
        .trimStart()
        .split("\n", 1)[0]
        .toLowerCase();
    return firstLine.startsWith("error:");
});

const errorMessage = computed(() => {
    if (!isErrorContext.value || !props.fileListContext) return "";

    // check if starts with "Error:" (case insensitive)
    const lowerCaseContext = props.fileListContext.toLowerCase();
    if (lowerCaseContext.startsWith("error:")) {
        return props.fileListContext
            .substring(props.fileListContext.indexOf(":") + 1)
            .trim();
    }

    // if it contains "error:" elsewhere, try to extract the message
    if (lowerCaseContext.includes("error:")) {
        const errorIndex = lowerCaseContext.indexOf("error:");
        return props.fileListContext.substring(errorIndex).trim();
    }

    return props.fileListContext.trim();
});

// character count and related computed properties
const charCount = computed(() => {
    return (props.finalPrompt || "").length;
});

const charCountColorClass = computed(() => {
    const count = geminiTokenCount.value;
    if (count < 1000000) {
        return "text-green-600";
    } else if (count <= 4000000) {
        return "text-yellow-500"; // using 500 for better visibility on white bg
    } else {
        return "text-red-600";
    }
});

const tooltipText = computed(() => {
    if (isCountingTokens.value) return "calculating tokens...";
    if (tokenCountError.value) return `error: ${tokenCountError.value}`;

    return `prompt contains ${geminiTokenCount.value.toLocaleString()} gemini tokens`;
});

const DEFAULT_RULES = `no additional rules`;

// responsive: detect narrow screens (< 900px)
const isSmallScreen = ref(window.innerWidth < 1200);

function updateScreenSize() {
    isSmallScreen.value = window.innerWidth < 1200;
}

onMounted(() => {
    window.addEventListener("resize", updateScreenSize);
});

onUnmounted(() => {
    window.removeEventListener("resize", updateScreenSize);
});

onMounted(async () => {
    try {
        localUserTask.value = props.userTask;
        // removed: load rules from the backend only on the first mount
        // if (isFirstMount.value) {
        //     const fetchedRules = await GetCustomPromptRules();
        //     if (!props.rulesContent) {
        //         emit("update:rulesContent", fetchedRules);
        //     }
        //     isFirstMount.value = false;
        // }
    } catch (error) {
        console.error("failed to load custom prompt rules:", error);
        LogErrorRuntime(
            `failed to load custom prompt rules: ${error.message || error}`
        );
        // if (isFirstMount.value && !props.rulesContent) {
        //     emit("update:rulesContent", DEFAULT_RULES);
        // }
        isFirstMount.value = false;
    }

    // automatically generate prompt on mount if we have context
    // generate even without userTask to show the prompt immediately
    if (props.fileListContext) {
        debouncedUpdateFinalPrompt();
    }
});

async function updateFinalPrompt(forceUpdate = false) {
    // skip if not needed and not forced
    if (
        !forceUpdate &&
        !promptNeedsUpdate.value &&
        !shouldGeneratePrompt.value
    ) {
        return;
    }

    isLoadingFinalPrompt.value = true;
    promptNeedsUpdate.value = false;
    shouldGeneratePrompt.value = false;

    // use requestanimationframe for smoother ui updates
    await new Promise((resolve) => requestAnimationFrame(resolve));

    const currentTemplateContent =
        promptTemplates[selectedPromptTemplateKey.value].content;
    let populatedPrompt = currentTemplateContent;
    populatedPrompt = populatedPrompt.replace(
        "{TASK}",
        props.userTask || "no task provided by the user."
    );
    populatedPrompt = populatedPrompt.replace("{RULES}", props.rulesContent);
    populatedPrompt = populatedPrompt.replace(
        "{FILE_STRUCTURE}",
        props.fileListContext || "no file structure context provided."
    );

    // insert current date in yyyy-mm-dd format
    const now = new Date();
    const yyyy = now.getFullYear();
    const mm = String(now.getMonth() + 1).padStart(2, "0");
    const dd = String(now.getDate()).padStart(2, "0");
    const currentDate = `${yyyy}-${mm}-${dd}`;
    populatedPrompt = populatedPrompt.replaceAll("{CURRENT_DATE}", currentDate);

    // only update if the prompt has actually changed
    if (populatedPrompt !== props.finalPrompt) {
        emit("update:finalPrompt", populatedPrompt);
        // defer token counting to avoid blocking ui
        setTimeout(() => countTokensForPrompt(populatedPrompt), 100);
    }

    isLoadingFinalPrompt.value = false;
}

function debouncedUpdateFinalPrompt() {
    // mark prompt as needing update but don't generate immediately
    promptNeedsUpdate.value = true;
    clearTimeout(finalPromptDebounceTimer);
    finalPromptDebounceTimer = setTimeout(() => {
        shouldGeneratePrompt.value = true;
        updateFinalPrompt();
    }, 1200); // increased to 1200ms to reduce update frequency during typing
}

// refresh prompt functionality
function refreshPrompt() {
    if (isLoadingFinalPrompt.value) return;

    // visual feedback
    refreshing.value = true;

    // force prompt regeneration with the force flag
    updateFinalPrompt(true);

    // reset refreshing state after a short delay
    setTimeout(() => {
        refreshing.value = false;
    }, 500);
}

watch(
    () => props.userTask,
    (newValue) => {
        if (newValue !== localUserTask.value) {
            localUserTask.value = newValue;
        }
    }
);

watch(localUserTask, (currentValue) => {
    clearTimeout(userTaskInputDebounceTimer);
    // emit user task changes and trigger auto-regeneration
    userTaskInputDebounceTimer = setTimeout(() => {
        if (currentValue !== props.userTask) {
            emit("update:userTask", currentValue);
            debouncedUpdateFinalPrompt(); // trigger auto-update
        }
    }, 300);
});

// watch for file list context changes and auto-regenerate
watch(
    () => props.fileListContext,
    () => {
        debouncedUpdateFinalPrompt();
    },
    { deep: false }
);

// watch for template selection changes and auto-regenerate
watch(selectedPromptTemplateKey, () => {
    debouncedUpdateFinalPrompt();
});

async function copyFinalPromptToClipboard() {
    if (!props.finalPrompt) return;
    try {
        await navigator.clipboard.writeText(props.finalPrompt);
        copyButtonText.value = "copied!";
        copySuccess.value = true;
        setTimeout(() => {
            copyButtonText.value = "copy";
            copySuccess.value = false;
        }, 2000);
    } catch (err) {
        console.error("failed to copy final prompt: ", err);
        if (props.platform === "darwin" && err) {
            console.error(
                "darvin clipboardsettext failed for final prompt:",
                err
            );
        }
        copyButtonText.value = "failed!";
        copySuccess.value = false;
        setTimeout(() => {
            copyButtonText.value = "copy";
        }, 2000);
    }
}

// removed: openPromptRulesModal, handleSavePromptRules, handleCancelPromptRules

defineExpose({});
</script>

<style scoped>
@keyframes fastPulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

.animate-pulse {
  animation: fastPulse 0.8s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}
</style>
