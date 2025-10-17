<template>
    <div class="p-4 h-full flex flex-col space-y-4">
        <!-- box 1: title and description -->
        <div class="px-2 py-2 border-2 border-accent rounded-[0.4rem] bg-white dark:bg-[#3a3b60]">
            <h1 class="text-2xl text-gray-900 dark:text-gray-100">apply patch</h1>
            <p class="text-sm text-gray-600 dark:text-gray-300">
                the original diff has been split into smaller diffs. copy each part and apply it using your preferred tool. with an llm, just tell it to apply the diff.
            </p>
        </div>

        <div
            v-if="isLoading"
            class="flex-grow flex justify-center items-center"
        >
            <p class="text-gray-600 dark:text-gray-300">
                loading split diffs...
            </p>
        </div>

        <div
            v-else-if="splitDiffs && splitDiffs.length > 0"
            class="flex-grow overflow-y-auto space-y-2"
        >
            <div
                v-for="(diff, index) in splitDiffs"
                :key="index"
                :class="[
                    'border-2 border-accent rounded-[0.4rem] px-2 pt-2 pb-[0.1rem]',
                    isCopied[index]
                        ? 'bg-green-50 dark:bg-green-900 dark:bg-opacity-20'
                        : 'bg-white dark:bg-[#3a3b60]',
                ]"
            >
                <div class="flex justify-between items-center">
                    <h3
                        class="text-xl font-medium text-gray-700 dark:text-gray-300"
                    >
                        split {{ index + 1 }} of {{ splitDiffs.length }}
                    </h3>
                    <div class="flex items-center space-x-2">
                        <!-- pills -->
                        <span
                            class="inline-block px-2 py-1 border border-accent rounded-[0.4rem] text-sm font-medium"
                            :class="getLineMetricClass(diff.split('\n').length)"
                        >
                            {{ diff.split("\n").length }} lines
                        </span>
                        <span
                            class="inline-block px-2 py-1 border border-accent rounded-[0.4rem] bg-blue-50 dark:bg-blue-900 dark:bg-opacity-30 text-gray-900 dark:text-gray-200 text-sm font-medium"
                        >
                            {{ (diff.match(/^diff --git/gm) || []).length }} file{{
                                (diff.match(/^diff --git/gm) || []).length === 1
                                    ? ""
                                    : "s"
                            }}
                        </span>
                        <span
                            class="inline-block px-2 py-1 border border-accent rounded-[0.4rem] bg-purple-50 dark:bg-purple-900 dark:bg-opacity-30 text-gray-900 dark:text-gray-200 text-sm font-medium"
                        >
                            {{ (diff.match(/^@@ .* @@/gm) || []).length }} hunk{{
                                (diff.match(/^@@ .* @@/gm) || []).length === 1
                                    ? ""
                                    : "s"
                            }}
                        </span>
                        <!-- copy button -->
                        <BaseButton
                            @click="copyDiffToClipboard(diff, index)"
                            class="text-xs px-2 py-1"
                        >
                            <template #icon>
                                <svg
                                    v-if="!isCopied[index]"
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
                            <span class="text-base">{{ copyButtonTexts[index] || "copy" }}</span>
                        </BaseButton>
                    </div>
                </div>
                <textarea
                    :value="diff"
                    rows="12"
                    readonly
                    spellcheck="false"
                    class="w-full p-2 mt-2 border-2 border-accent rounded-[0.4rem] bg-white dark:bg-dark-surface text-gray-900 dark:text-gray-100 font-mono text-sm resize-none"
                ></textarea>
            </div>
        </div>

        <div v-else class="flex-grow flex justify-center items-center px-2 py-2 border-2 border-accent rounded-[0.4rem] bg-white dark:bg-[#3a3b60]">
            <h1 class="text-xl text-gray-600 dark:text-gray-300">
                no split diffs to display. go to step 3 to split a diff.
            </h1>
        </div>

    </div>
</template>

<script setup>
import { ref, watch } from "vue";
import BaseButton from "../BaseButton.vue";

const finishButtonText = ref("finish");
// import { clipboardsettext as wailsclipboardsettext } from '../../../wailsjs/runtime/runtime'; // if needed for specific platforms

const props = defineProps({
    splitDiffs: {
        type: Array,
        default: () => [],
    },
    isLoading: {
        // to indicate if mainlayout is fetching/processing splits
        type: Boolean,
        default: false,
    },
    platform: {
        type: String,
        default: "unknown",
    },
    splitLineLimit: {
        // add the new prop
        type: Number,
        default: 500, // provide a default value if the prop is not passed
    },
});

defineEmits(["action"]);

const copyButtonTexts = ref({});
const isCopied = ref({}); // tracks if a split has been successfully copied at least once

function getLineMetricClass(lineCount) {
    const limit = props.splitLineLimit;
    // clamp the thresholds to maximum 100 or 200 lines over the limit
    const orangeThreshold = Math.min(limit * 1.1, limit + 100);
    const redThreshold = Math.min(limit * 1.3, limit + 200);

    if (lineCount > redThreshold) {
        return "bg-red-50 dark:bg-red-900 dark:bg-opacity-30 text-red-700 dark:text-red-200";
    } else if (lineCount > orangeThreshold) {
        return "bg-orange-50 dark:bg-orange-900 dark:bg-opacity-30 text-orange-700 dark:text-orange-200";
    } else {
        return "bg-green-50 dark:bg-green-900 dark:bg-opacity-30 text-green-700 dark:text-green-200";
    }
}

watch(
    () => props.splitDiffs,
    (newVal) => {
        // reset copy button texts and copied states when diffs change
        const newTexts = {};
        const newCopiedStates = {};
        if (newVal) {
            newVal.forEach((_, index) => {
                newTexts[index] = "copy";
                newCopiedStates[index] = false; // initialize as not copied
            });
        }
        copyButtonTexts.value = newTexts;
        isCopied.value = newCopiedStates;
    },
    { immediate: true, deep: true }
); // use deep: true if splitdiffs could be mutated internally, though usually props are replaced.

async function copyDiffToClipboard(diffContent, index) {
    if (!diffContent) return;
    try {
        await navigator.clipboard.writeText(diffContent);

        isCopied.value[index] = true; // mark as successfully copied
        copyButtonTexts.value[index] = "copied";

        setTimeout(() => {
            copyButtonTexts.value[index] = "copied"; // persistent "copied" state text without emoji
        }, 2000);
    } catch (err) {
        console.error(`failed to copy diff split ${index + 1}: `, err);

        // temporarily show "failed!"
        const originalText = isCopied.value[index] ? "copied" : "copy";
        copyButtonTexts.value[index] = "failed!";

        setTimeout(() => {
            copyButtonTexts.value[index] = originalText; // revert to previous state ("copy" or "copied")
        }, 2000);
    }
}
</script>
