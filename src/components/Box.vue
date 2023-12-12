<template>
    <div class="flex h-screen">
        <div class="flex-1 p-4 bg-white">
            <NInput
                class="h-full"
                v-model:value="rawText"
                type="textarea"
                placeholder=""
            ></NInput>
        </div>
        <div class="border-r border-gray-300 h-full"></div>
        <div class="flex-1 p-4 bg-white">第二列内容</div>
    </div>

</template>
<script setup lang="ts">
import { NInput } from 'naive-ui';
import { ref, watch } from 'vue';
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
const rawText = ref('');
const textCollation = ref<ChapterData[]>([]);
watch(rawText, raw => {
    if(raw) {
        textCollation.value = extractChapters(raw);
        debugger
    }
})
</script>