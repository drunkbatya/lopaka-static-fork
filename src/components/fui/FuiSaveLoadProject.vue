<script
    lang="ts"
    setup
>
import { ref } from 'vue';
import { useSession } from '/src/core/session';
import Button from '/src/components/layout/Button.vue';
import Icon from '/src/components/layout/Icon.vue';

const session = useSession();
const fileInput = ref<HTMLInputElement | null>(null);

const emit = defineEmits(['setInfoMessage', 'setErrorMessage']);

function saveProject() {
    const data = session.exportProject();
    const json = JSON.stringify(data, null, 2);
    const blob = new Blob([json], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `lopaka-${data.platform}-${data.display.x}x${data.display.y}.json`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    emit('setInfoMessage', 'Project saved');
}

function pickFile() {
    fileInput.value?.click();
}

async function onFileChange(e: Event) {
    const input = e.target as HTMLInputElement;
    const file = input.files?.[0];
    if (!file) return;
    try {
        const text = await file.text();
        const data = JSON.parse(text);
        await session.importProject(data);
        emit('setInfoMessage', 'Project loaded');
    } catch (err) {
        emit('setErrorMessage', `Failed to load project: ${(err as Error).message}`);
    } finally {
        input.value = '';
    }
}
</script>

<template>
    <div class="flex flex-row gap-2 items-center">
        <Button
            title="Save project to JSON file"
            @click="saveProject"
        >
            <Icon
                type="share"
                sm
                pointer
            />
            Save
        </Button>
        <Button
            title="Load project from JSON file"
            @click="pickFile"
        >
            <Icon
                type="upload"
                sm
                pointer
            />
            Load
        </Button>
        <input
            ref="fileInput"
            type="file"
            accept="application/json,.json"
            style="display: none"
            @change="onFileChange"
        />
    </div>
</template>
