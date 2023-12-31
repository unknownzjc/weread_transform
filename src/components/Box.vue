<template>
    <div class="flex h-screen">
        <div class="flex-1 p-4 bg-white">
            <n-input
                class="h-full"
                v-model:value="rawText"
                type="textarea"
                placeholder=""
            ></n-input>
        </div>
        <div class="border-r border-gray-300 h-full"></div>
        <div class="relative flex-1 p-4 bg-white">
            <div
                class="h-full overflow-scroll"
                v-html="textCollation"
            />
            <div class="absolute bottom-6 right-6">
                <n-button @click="copy(mdRaw)">Copy</n-button>
            </div>
        </div>
    </div>

</template>
<script setup lang="ts">
import { NInput, NButton } from 'naive-ui';
import { ref, watch } from 'vue';
import markdownit from 'markdown-it';
import { useClipboard } from '@vueuse/core';
type HighlightWithThought = {
    date: string,
    thought: string,
    highlight: string
};

type ChapterData = {
    title: string,
    highlightsWithThoughts: HighlightWithThought[]
};
function extractChapters(notes: string): ChapterData[] {
    const chapters: ChapterData[] = [];
    const chapterLines = notes.split('◆').slice(1); // 用“◆”分割章节，忽略第一个空元素

    chapterLines.forEach(chapter => {
        const lines = chapter.trim().split('\n');
        let title = lines[0]; // 假设章节的第一行是标题
        const highlightsWithThoughts: HighlightWithThought[] = [];
        let currentDate = '';
        let currentThought = '';

        lines.slice(1).forEach(line => { // 从第二行开始遍历
            if (line.trim().endsWith('发表想法')) {
                currentDate = line.trim();
            } else if (line.trim() && !line.trim().startsWith('>>')) {
                currentThought = line.trim();
            } else if (line.trim().startsWith('>>')) {
                const highlight = line.trim().substring(2).trim();
                const existingHighlight = highlightsWithThoughts.find(h => h.highlight === highlight);
                if(existingHighlight) {
                    currentThought.trim() !== '' && (existingHighlight.thought = currentThought.trim());
                }else {
                    highlightsWithThoughts.push({ date: currentDate, thought: currentThought, highlight });
                }
                currentThought = ''; // 重置想法，为下一个高亮准备
            }
        });

        chapters.push({ title, highlightsWithThoughts });
    });

    return chapters;
}
function dataToMarkDown(data: ChapterData[]) {
    let markdown = '';

    data.forEach(chapterData => {
        // 添加标题作为折叠区块
        markdown += `## ${chapterData.title}\n\n`;

        // 遍历 highlightsWithThoughts 并添加到 Markdown
        chapterData.highlightsWithThoughts.forEach(item => {
            if (item.thought) {
                markdown += `- ${item.thought}\n`;
                markdown += `  > ${item.highlight}\n\n`;
            } else {
                markdown += `- ${item.highlight}\n\n`;
            }
        });

        markdown += `\n\n`;
    });

    return markdown;
}
const md = new markdownit({
    html: true
});
const { copy } = useClipboard();
const rawText = ref('');
const textCollation = ref<string>('');
const mdRaw = ref('');
watch(rawText, raw => {
    if(raw) {
        const ec = extractChapters(raw);
        mdRaw.value = dataToMarkDown(ec);
        textCollation.value = md.render(mdRaw.value);
    }
})
</script>