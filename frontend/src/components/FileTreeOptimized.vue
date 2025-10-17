<template>
    <div class="file-tree-container" ref="scrollContainer" @scroll="handleScroll">
        <div :style="{ height: totalHeight + 'px' }" class="virtual-spacer">
            <div :style="{ transform: `translateY(${offsetY}px)` }">
                <div
                    v-for="(node, idx) in visibleNodes"
                    :key="node.path"
                    :class="{ 'excluded-node': node.excluded, 'border-b': idx !== visibleNodes.length - 1, 'even-row': getGlobalIndex(node, idx) % 2 === 0 }"
                    class="node-item p-1 border-none pr-3"
                    :style="{ 'padding-left': node.depth * 15 + 'px' }"
                    @click="handleNodeClick($event, node)"
                >
                    <template v-for="lineDepth in (node.depth > 1 ? node.depth - 1 : 0)" :key="`line-${lineDepth}`">
                        <div
                            class="tree-lines"
                            :style="{ 'left': lineDepth * 15 + 15 + 'px' }"
                        ></div>
                    </template>
                    <span class="node-content-wrapper pl-3">
                        <span
                            v-if="node.isDir"
                            @click.stop="toggleExpand(node)"
                            class="toggler"
                        >
                            <i
                                :class="expandedPaths.has(node.path) ? 'codicon-folder-opened' : 'codicon-folder'"
                                class="codicon text-decoration-none no-underline"
                            />
                        </span>
                        <span v-else class="file-icon">
                            <i :class="`codicon ${getFileIcon(node.name)}`" class="text-decoration-none no-underline"></i>
                        </span>
                        <span
                            @click.stop="node.isDir ? toggleExpand(node) : handleCheckboxChange(node)"
                            :class="{ 'folder-name': node.isDir }"
                            class="text-sm name-label"
                        >
                            {{ node.name }}
                        </span>
                    </span>

                    <span class="checkbox-wrapper" @click.stop>
                        <input
                            type="checkbox"
                            :checked="!node.excluded"
                            @change="handleCheckboxChange(node)"
                            class="exclude-checkbox"
                        />
                    </span>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch, nextTick } from "vue";

const props = defineProps({
    nodes: {
        type: Array,
        required: true
    },
    projectRoot: String,
    loadingError: String,
    useGitignore: {
        type: Boolean,
        default: true
    }
});

const emit = defineEmits(["toggle-exclude", "add-log"]);

// virtual scrolling configuration
const itemHeight = 30; // height of each item in pixels
const bufferSize = 10; // extra items to render outside viewport
const scrollContainer = ref(null);
const scrollTop = ref(0);
const containerHeight = ref(600);

// flatten the tree structure for virtual scrolling
const flattenedNodes = ref([]);
const expandedPaths = ref(new Set());

function flattenTree(nodes, depth = 0) {
    const flat = [];
    for (const node of nodes) {
        // skip nodes that are built-in gitignored when useGitignore is enabled
        if (props.useGitignore && node.isGitignored) {
            continue;
        }
        flat.push({ ...node, depth });
        if (node.isDir && node.children && expandedPaths.value.has(node.path)) {
            flat.push(...flattenTree(node.children, depth + 1));
        }
    }
    return flat;
}

// compute flattened nodes whenever tree or expansion state changes
const updateFlattenedNodes = () => {
    flattenedNodes.value = flattenTree(props.nodes);
};

// auto-expand root folders when nodes change
watch(() => props.nodes, (newNodes) => {
    console.log('debug: filetreeoptimized - nodes changed, checking for auto-expansion');
    if (newNodes && newNodes.length > 0) {
        // auto-expand all root-level directories
        newNodes.forEach(node => {
            if (node.isDir && node.expanded) {
                console.log(`debug: auto-expanding root folder: ${node.name}, path: ${node.path}`);
                expandedPaths.value.add(node.path);
            }
        });
    }
    updateFlattenedNodes();
}, { deep: true, immediate: true });

watch(expandedPaths, updateFlattenedNodes, { deep: true });

watch(() => props.useGitignore, () => {
    console.log('debug: filetreeoptimized - useGitignore changed, updating tree');
    updateFlattenedNodes();
});

const totalHeight = computed(() => flattenedNodes.value.length * itemHeight);

const visibleNodes = computed(() => {
    const start = Math.max(0, Math.floor(scrollTop.value / itemHeight) - bufferSize);
    const end = Math.min(
        flattenedNodes.value.length,
        Math.ceil((scrollTop.value + containerHeight.value) / itemHeight) + bufferSize
    );
    return flattenedNodes.value.slice(start, end);
});

const offsetY = computed(() => {
    const start = Math.max(0, Math.floor(scrollTop.value / itemHeight) - bufferSize);
    return start * itemHeight;
});

function handleScroll(event) {
    scrollTop.value = event.target.scrollTop;
}

function updateContainerHeight() {
    if (scrollContainer.value) {
        containerHeight.value = scrollContainer.value.clientHeight;
    }
}

onMounted(() => {
    updateContainerHeight();
    window.addEventListener("resize", updateContainerHeight);
});

onUnmounted(() => {
    window.removeEventListener("resize", updateContainerHeight);
});

function toggleExpand(node) {
    if (!node.isDir) return;

    const isCurrentlyExpanded = expandedPaths.value.has(node.path);
    console.log(`debug: toggleexpand called for ${node.name}, currently expanded: ${isCurrentlyExpanded}, path: ${node.path}`);

    if (isCurrentlyExpanded) {
        expandedPaths.value.delete(node.path);
        console.log(`debug: collapsed ${node.name}`);
    } else {
        // lazy load children if not loaded yet
        if (!node.children || node.children.length === 0) {
            console.log(`debug: node ${node.name} has no children, may need lazy loading`);
            // emit event to load children from backend
            emit("add-log", {
                message: `loading children for ${node.name}...`,
                type: "info"
            });
        }
        expandedPaths.value.add(node.path);
        console.log(`debug: expanded ${node.name}, children count: ${node.children?.length || 0}`);
    }

    // update flattened nodes after expansion state change
    nextTick(() => {
        updateFlattenedNodes();
    });
}

function getFileIcon(filename) {
    const ext = filename.split('.').pop()?.toLowerCase();
    const iconMap = {
        'js': 'codicon-file-code',
        'ts': 'codicon-file-code',
        'tsx': 'codicon-file-code',
        'jsx': 'codicon-file-code',
        'vue': 'codicon-file-code',
        'json': 'codicon-json',
        'css': 'codicon-file-code',
        'scss': 'codicon-file-code',
        'less': 'codicon-file-code',
        'html': 'codicon-file-code',
        'md': 'codicon-file-text',
        'markdown': 'codicon-file-text',
        'txt': 'codicon-file-text',
        'log': 'codicon-file-text',
        'yml': 'codicon-file-code',
        'yaml': 'codicon-file-code',
        'xml': 'codicon-file-code',
        'toml': 'codicon-file-code',
        'go': 'codicon-file-code',
        'py': 'codicon-file-code',
        'java': 'codicon-file-code',
        'cpp': 'codicon-file-code',
        'c': 'codicon-file-code',
        'h': 'codicon-file-code',
        'rb': 'codicon-file-code',
        'php': 'codicon-file-code',
        'sql': 'codicon-file-code',
        'sh': 'codicon-file-code',
        'bash': 'codicon-file-code',
        'png': 'codicon-file-media',
        'jpg': 'codicon-file-media',
        'jpeg': 'codicon-file-media',
        'gif': 'codicon-file-media',
        'svg': 'codicon-file-media',
        'webp': 'codicon-file-media',
        'ico': 'codicon-file-media',
        'zip': 'codicon-file-zip',
        'gz': 'codicon-file-zip',
        'tar': 'codicon-file-zip',
        'rar': 'codicon-file-zip',
        '7z': 'codicon-file-zip',
        'pdf': 'codicon-file-pdf',
        'doc': 'codicon-file-text',
        'docx': 'codicon-file-text',
        'xls': 'codicon-file-text',
        'xlsx': 'codicon-file-text',
        'env': 'codicon-file-code',
        'gitignore': 'codicon-file-code',
        'lock': 'codicon-lock',
    };
    return iconMap[ext] || 'codicon-file';
}

function handleCheckboxChange(node) {
    console.log(`debug: handlecheckboxchange called for node: ${node.name}, path: ${node.relPath}`);
    emit("toggle-exclude", node);
}

function handleNodeClick(event, node) {
    const target = event.target;

    // ignore clicks on checkbox area
    if (target.closest('.checkbox-wrapper') || target.classList.contains('exclude-checkbox')) {
        return;
    }

    if (node.isDir) {
        if (!target.closest('.toggler') && !target.classList.contains('folder-name')) {
            toggleExpand(node);
        }
    } else {
        console.log(`debug: handlenodeclick calling handlecheckboxchange for file: ${node.name}`);
        handleCheckboxChange(node);
    }
}

function getGlobalIndex(node, visibleIdx) {
    // find the global index of the node in the flattened array
    return flattenedNodes.value.findIndex(n => n.path === node.path);
}
</script>

<style scoped>
.file-tree-container {
    height: 100%;
    overflow-y: auto;
    position: relative;
}

.virtual-spacer {
    position: relative;
}

.node-item {
    height: 30px;
    display: flex;
    align-items: center;
    cursor: default;
    transition: background-color 0.15s ease;
    position: relative;
    background-color: rgba(131, 195, 231, 0.25);
}

:global(.dark) .node-item {
    background-color: rgba(5, 5, 15, 0.8);
}

.node-item.even-row {
    background-color: transparent;
}

:global(.dark) .node-item.even-row {
    background-color: transparent;
}

.node-item:hover {
    background-color: var(--accent);
}

.toggler {
    cursor: pointer;
    width: 20px;
    height: 20px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
}

.file-icon {
    width: 20px;
    height: 20px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
}

.folder-name {
    cursor: pointer;
    font-weight: bold;
}

.checkbox-wrapper {
    margin-left: auto;
    display: flex;
    align-items: center;
}

.exclude-checkbox {
    cursor: pointer;
    width: 20px;
    height: 20px;
}

.excluded-node > .node-item > span:not(.toggler, .file-icon) {
    color: var(--muted-foreground);
}

.name-label {
    margin-left: 8px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    flex-shrink: 1;
    min-width: 0;
}

.node-content-wrapper {
    display: flex;
    align-items: center;
    flex-grow: 1;
    min-width: 0;
    overflow: hidden;
    height: 100%;
}

.node-content-wrapper .codicon {
    display: flex;
    align-items: center;
    justify-content: center;
}

.tree-lines {
    position: absolute;
    top: 0;
    width: 3px;
    height: 100%;
    background-color: var(--accent);
    opacity: 0.6;
}
</style>
