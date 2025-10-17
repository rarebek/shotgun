<template>
    <div class="p-4 h-full flex flex-col space-y-4">
        <!-- box 1: title and description -->
        <div class="px-2 py-2 border-2 border-accent rounded-[0.4rem] bg-white dark:bg-[#3a3b60]">
            <h1 class="text-2xl text-gray-900 dark:text-gray-100">prepare the diff to apply</h1>
            <p class="text-sm text-gray-600 dark:text-gray-300">this tool will split the diff into smaller parts to make it easier to apply.</p>
        </div>

        <!-- box 2: textarea with copy/clear buttons -->
        <div class="flex-grow flex flex-col overflow-hidden px-2 py-2 border-2 border-accent rounded-[0.4rem] bg-white dark:bg-[#3a3b60]">
            <BaseButton
                v-if="localShotgunGitDiffInput.trim()"
                @click="copyDiffToClipboard"
                class="text-xs px-2 py-1"
                :class="{ 'bg-green-600 dark:bg-green-700': copySuccess }"
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
                <span class="text-base">{{
                    copySuccess ? "copied!" : "copy"
                }}</span>
            </BaseButton>
            <BaseButton
                v-if="localShotgunGitDiffInput.trim()"
                @click="clearTextarea"
                class="text-xs px-2 py-1"
                variant="danger"
                :class="{ 'bg-red-600 dark:bg-red-700': clearSuccess }"
            >
                <template #icon>
                    <svg
                        v-if="!clearSuccess"
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
                            d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
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
                <span class="text-base">{{
                    clearSuccess ? "cleared!" : "clear"
                }}</span>
            </BaseButton>
            <textarea
                id="shotgun-git-diff-input"
                v-model="localShotgunGitDiffInput"
                rows="80"
                spellcheck="false"
                class="w-full p-2 border-2 border-accent rounded-[0.4rem] shadow-sm focus:ring-light-accent dark:focus:ring-dark-accent focus:border-light-accent dark:focus:border-dark-accent text-sm font-mono bg-white dark:bg-dark-surface text-gray-900 dark:text-gray-100 resize-none"
                placeholder="paste the git diff output here, e.g., diff --git a/file.txt b/file.txt..."
            ></textarea>
        </div>

        <!-- box 3: description, input, and button -->
        <div class="px-2 py-2 border-2 border-accent rounded-[0.4rem] bg-white dark:bg-[#3a3b60]">
            <p class="text-gray-600 dark:text-gray-300 mb-2 text-sm">
                this will attempt to split the diff into the specified number of lines, while keeping the original structure and the chunks.
                <br />
                the exact number of lines per split is not guaranteed, but the diff will be split into as many parts as possible.
            </p>
            <div class="flex items-center space-x-2 mb-3">
                <input
                    type="number"
                    id="split-line-limit"
                    v-model.number="localSplitLineLimit"
                    min="50"
                    step="50"
                    class="w-1/8 p-2 border-2 border-accent rounded-[0.4rem] shadow-sm focus:ring-light-accent dark:focus:ring-dark-accent focus:border-light-accent dark:focus:border-dark-accent text-sm bg-white dark:bg-dark-surface text-gray-900 dark:text-gray-100"
                />
                <label
                    for="split-line-limit"
                    class="block text-base font-bold text-gray-700 dark:text-gray-300"
                    >* approx. lines per split</label
                >
            </div>
            <BaseButton
                @click="handleSplitDiff"
                :disabled="
                    !localShotgunGitDiffInput.trim() || localSplitLineLimit <= 0
                "
                class="text-xs px-2 py-1 self-start"
            >
                <span class="text-base">{{
                    localSplitLineLimit === shotgunGitDiffInputLines
                        ? "proceed to apply"
                        : "split diff"
                }}</span>
            </BaseButton>
        </div>
    </div>
</template>

<script setup>
import { ref, watch, computed, onMounted, onBeforeUnmount } from "vue";
import { LogError as LogErrorRuntime } from "../../../wailsjs/runtime/runtime";
import BaseButton from "../BaseButton.vue";

const emit = defineEmits([
    "action",
    "update:shotgunGitDiff",
    "update:splitLineLimit",
]);

const props = defineProps({
    initialGitDiff: {
        type: String,
        default: "",
    },
    initialSplitLineLimit: {
        type: Number,
        default: 0,
    },
    finalPrompt: {
        type: String,
        default: "",
    },
});

// copy functionality state
const copySuccess = ref(false);

// clear functionality state
const clearSuccess = ref(false);

const localShotgunGitDiffInput = ref(props.initialGitDiff);

const localSplitLineLimit = ref(
    props.initialSplitLineLimit > 0 ? props.initialSplitLineLimit : 500
);

onMounted(() => {
    localShotgunGitDiffInput.value = props.initialGitDiff;

    if (props.initialSplitLineLimit > 0) {
        localSplitLineLimit.value = props.initialSplitLineLimit;
    } else if (localSplitLineLimit.value <= 0) {
        localSplitLineLimit.value = 500;
    }
});

const shotgunGitDiffInputLines = computed(() => {
    return localShotgunGitDiffInput.value
        ? localShotgunGitDiffInput.value.split("\n").length
        : 0;
});

watch(
    () => props.initialGitDiff,
    (newVal, oldVal) => {
        if (newVal !== localShotgunGitDiffInput.value) {
            localShotgunGitDiffInput.value = newVal;
        }
    }
);

watch(
    () => props.initialSplitLineLimit,
    (newVal, oldVal) => {
        if (newVal > 0 && newVal !== localSplitLineLimit.value) {
            localSplitLineLimit.value = newVal;
        } else if (
            newVal <= 0 &&
            localSplitLineLimit.value !== 500 &&
            props.initialGitDiff === ""
        ) {
            localSplitLineLimit.value = 500;
        }
    }
);

// token counting removed - no longer needed since we don't display it

let diffInputDebounceTimer = null;
watch(localShotgunGitDiffInput, (newVal, oldVal) => {
    clearTimeout(diffInputDebounceTimer);

    diffInputDebounceTimer = setTimeout(() => {
        if (newVal !== props.initialGitDiff) {
            emit("update:shotgunGitDiff", newVal);
        } else {
        }
        if (newVal && newVal.trim() !== "") {
            const lines = newVal.split("\n").length;
            const currentLimit = localSplitLineLimit.value;

            if (
                currentLimit === 500 ||
                (currentLimit !== lines &&
                    currentLimit ===
                        newVal
                            .substring(
                                0,
                                newVal.length -
                                    (newVal.split("\n").pop().length + 1)
                            )
                            .split("\n").length)
            ) {
                if (lines > 0 && lines !== currentLimit) {
                    localSplitLineLimit.value = lines;
                }
            } else if (lines === 0 && currentLimit !== 500) {
                localSplitLineLimit.value = 500;
            }
        } else if (
            (!newVal || newVal.trim() === "") &&
            localSplitLineLimit.value !== 500
        ) {
            localSplitLineLimit.value = 500;
        }
    }, 300);
});

let limitDebounceTimer = null;
watch(localSplitLineLimit, (newVal) => {
    clearTimeout(limitDebounceTimer);
    limitDebounceTimer = setTimeout(() => {
        if (newVal > 0 && newVal !== props.initialSplitLineLimit) {
            emit("update:splitLineLimit", newVal);
        } else if (newVal <= 0 && props.initialSplitLineLimit > 0) {
        }
    }, 300);
});

onBeforeUnmount(() => {
    // clear any pending debounced updates
    clearTimeout(diffInputDebounceTimer);
    clearTimeout(limitDebounceTimer);

    // immediately emit the current value of localshotgungitdiffinput if it's different from the prop
    if (localShotgunGitDiffInput.value !== props.initialGitDiff) {
        emit("update:shotgunGitDiff", localShotgunGitDiffInput.value);
    } else {
    }

    // immediately emit the current value of localsplitlinelimit if it's valid and different from the prop
    if (
        localSplitLineLimit.value > 0 &&
        localSplitLineLimit.value !== props.initialSplitLineLimit
    ) {
        emit("update:splitLineLimit", localSplitLineLimit.value);
    } else {
    }
});

function handleSplitDiff() {
    if (
        !localShotgunGitDiffInput.value.trim() ||
        localSplitLineLimit.value <= 0
    ) {
        return;
    }
    emit("action", "executePromptAndSplitDiff", {
        gitDiff: localShotgunGitDiffInput.value,
        lineLimit: localSplitLineLimit.value,
    });
}

function copyDiffToClipboard() {
    if (localShotgunGitDiffInput.value) {
        navigator.clipboard
            .writeText(localShotgunGitDiffInput.value)
            .then(() => {
                copySuccess.value = true;
                // reset the success state after 2 seconds
                setTimeout(() => {
                    copySuccess.value = false;
                }, 2000);
            })
            .catch((err) => {
                LogErrorRuntime("failed to copy to clipboard: " + err);
            });
    }
}

function clearTextarea() {
    if (localShotgunGitDiffInput.value) {
        // clear the textarea
        localShotgunGitDiffInput.value = "";

        // show success message
        clearSuccess.value = true;

        // reset the success state after 2 seconds
        setTimeout(() => {
            clearSuccess.value = false;
        }, 2000);
    }
}
</script>
