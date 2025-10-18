<template>
    <div class="p-4 h-full flex flex-col space-y-4">
        <!-- box 1: title and description -->
        <div class="px-2 py-2 border-2 border-accent rounded-[0.4rem] bg-white dark:bg-[#3a3b60]">
            <h1 class="text-2xl text-gray-900 dark:text-gray-100">prepare the diff to apply</h1>
            <p class="text-sm text-gray-600 dark:text-gray-300">this tool will split the diff into smaller parts to make it easier to apply.</p>
        </div>

        <!-- box 2: textarea without buttons -->
        <div class="flex-grow flex flex-col overflow-hidden px-2 py-2 border-2 border-accent rounded-[0.4rem] bg-white dark:bg-[#3a3b60]">
            <textarea
                id="shotgun-git-diff-input"
                v-model="localShotgunGitDiffInput"
                rows="80"
                spellcheck="false"
                class="w-full p-2 border-2 border-accent rounded-[0.4rem] shadow-sm focus:ring-light-accent dark:focus:ring-dark-accent focus:border-light-accent dark:focus:border-dark-accent text-lg font-mono bg-white dark:bg-dark-surface text-gray-900 dark:text-gray-100 resize-none"
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
            <div class="flex items-center space-x-2">
                <input
                    type="number"
                    id="split-line-limit"
                    v-model.number="localSplitLineLimit"
                    min="50"
                    step="50"
                    class="w-1/8 p-2 border-2 border-accent rounded-[0.4rem] shadow-sm focus:ring-light-accent dark:focus:ring-dark-accent focus:border-light-accent dark:focus:border-dark-accent text-sm bg-white dark:bg-dark-surface text-gray-900 dark:text-gray-100"
                />
                <BaseButton
                    @click="handleSplitDiff"
                    :disabled="
                        !localShotgunGitDiffInput.trim() || localSplitLineLimit <= 0
                    "
                    class="text-xs px-2 py-1"
                >
                    <span class="text-base">{{
                        localSplitLineLimit === shotgunGitDiffInputLines
                            ? "proceed to apply"
                            : "split diff"
                    }}</span>
                </BaseButton>
            </div>
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
</script>
