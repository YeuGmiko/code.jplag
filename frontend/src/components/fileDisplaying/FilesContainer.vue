<!--
  Container containing CodePanels for all of the files in a submission.
-->
<template>
  <Container class="flex flex-col print:!px-1">
    <div class="mb-2 mr-2 flex items-center space-x-5">
      <h3 class="flex-grow text-left text-lg font-bold">
        文件：
        {{ fileOwnerDisplayName }}:
      </h3>
      <div v-if="tokenCount >= 0" class="text-gray-600 dark:text-gray-300">
        {{ tokenCount }} total tokens
      </div>
      <Button @click="collapseAll()" class="space-x-2 print:hidden"
        ><FontAwesomeIcon :icon="['fas', 'compress-alt']" />
        <p>折叠所有</p></Button
      >
    </div>

    <ScrollableComponent class="flex-grow" ref="scrollContainer">
      <VueDraggableNext>
        <CodePanel
          v-for="(file, index) in files"
          :key="index"
          ref="codePanels"
          :file="file"
          :matches="
            !matches.get(file.fileName) ? [] : (matches.get(file.fileName) as MatchInSingleFile[])
          "
          :highlight-language="highlightLanguage"
          @match-selected="(match) => $emit('matchSelected', match)"
          class="mt-1 first:mt-0"
        />
      </VueDraggableNext>
    </ScrollableComponent>
  </Container>
</template>

<script setup lang="ts">
import type { SubmissionFile } from '@/model/File'
import CodePanel from './CodePanel.vue'
import Container from '../ContainerComponent.vue'
import Button from '../ButtonComponent.vue'
import ScrollableComponent from '../ScrollableComponent.vue'
import { VueDraggableNext } from 'vue-draggable-next'
import { computed, nextTick, ref, type PropType, type Ref } from 'vue'
import type { MatchInSingleFile } from '@/model/MatchInSingleFile'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
import { faCompressAlt } from '@fortawesome/free-solid-svg-icons'
import { library } from '@fortawesome/fontawesome-svg-core'
import type { Language } from '@/model/Language'

library.add(faCompressAlt)

const props = defineProps({
  /**
   * Files of the submission.
   */
  files: {
    type: Array<SubmissionFile>,
    required: true
  },
  /**
   * Matches of submission.
   */
  matches: {
    type: Map<string, MatchInSingleFile[]>,
    required: true
  },
  /**
   * Name of the owner of the files.
   */
  fileOwnerDisplayName: {
    type: String,
    required: true
  },
  /**
   * Language of the files.
   */
  highlightLanguage: {
    type: String as PropType<Language>,
    required: true
  }
})

defineEmits(['matchSelected'])

const codePanels: Ref<(typeof CodePanel)[]> = ref([])
const scrollContainer: Ref<typeof ScrollableComponent | null> = ref(null)

const tokenCount = computed(() => {
  return props.files.reduce((acc, file) => (file.tokenCount ?? 0) + acc - 1, 0)
})

/**
 * Scrolls to the given file and line in the container.
 * @param file Name of the file to scroll to.
 * @param line Line to scroll to.
 */
function scrollTo(file: string, line: number) {
  const fileIndex = Array.from(props.files).findIndex((f) => f.fileName === file)
  if (fileIndex !== -1) {
    console.log(fileIndex)
    codePanels.value[fileIndex].expand()
    nextTick(() => {
      if (!scrollContainer.value) {
        console.log('null')
        return
      }
      const childToScrollTo = codePanels.value[fileIndex].getLineRect(line) as DOMRect
      const scrollBox = scrollContainer.value.getRoot() as HTMLElement
      scrollBox.scrollTo({
        top: childToScrollTo.top + scrollBox.scrollTop - (scrollBox.clientHeight * 2) / 3
      })
    })
  }
}

/**
 * Collapses all of the code panels.
 */
function collapseAll() {
  codePanels.value.forEach((panel) => panel.collapse())
}

defineExpose({
  scrollTo
})
</script>
