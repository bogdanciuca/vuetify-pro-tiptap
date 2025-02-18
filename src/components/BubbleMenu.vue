<script setup lang="ts">
import { computed, reactive, unref } from 'vue'
import type { NodeSelection } from '@tiptap/pm/state'
import { TextSelection } from '@tiptap/pm/state'
import type { Editor, Extension } from '@tiptap/vue-3'
import { BubbleMenu } from '@tiptap/vue-3'

import type { BaseKitOptions } from '@/extensions/base-kit'
import type { BubbleTypeMenu, NodeTypeKey } from '@/extensions/components/bubble'
import { useLocale } from '@/locales'

interface Props {
  editor: Editor
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  disabled: false
})

const { t } = useLocale()

const tippyOptions = reactive<Record<string, unknown>>({
  maxWidth: 'auto',
  zIndex: 20,
  appendTo: 'parent'
})

const nodeType = computed<NodeTypeKey | undefined>(() => {
  const selection = props.editor.state.selection as NodeSelection

  const isImage = selection.node?.type.name === 'image'
  const isVideo = selection.node?.type.name === 'video'
  const isText = selection instanceof TextSelection

  if (isImage) return 'image'
  if (isVideo) return 'video'
  if (isText) return 'text'
  return undefined
})

const nodeMenus = computed(() => {
  const { extensions = [] } = props.editor.extensionManager
  const find = extensions.find(k => k.name === 'base-kit') as Extension<BaseKitOptions>
  if (!find) return {}

  const { button } = find.options?.bubble ?? {}

  if (!button) return {}

  const _button: BubbleTypeMenu = button({
    editor: props.editor,
    extension: find,
    t: unref(t)
  })

  return _button
})

const items = computed(() => {
  if (!nodeType.value) return []
  return unref(nodeMenus)?.[nodeType.value] ?? []
})
</script>

<template>
  <BubbleMenu v-if="items.length > 0" :editor="editor" :tippy-options="tippyOptions">
    <VCard class="vuetify-pro-tiptap-editor__menu-bubble">
      <VCardText class="d-flex pa-0">
        <VToolbar density="compact" flat height="auto" class="py-1 ps-1">
          <template v-for="(item, key) in items" :key="key">
            <!-- Divider -->
            <VDivider v-if="item.type === 'divider'" vertical class="mx-1 me-2" />
            <!-- Buttons -->
            <component :is="item.component" v-else v-bind="item.componentProps" :editor="editor" :disabled="disabled" />
          </template>
        </VToolbar>
      </VCardText>
    </VCard>
  </BubbleMenu>
</template>
