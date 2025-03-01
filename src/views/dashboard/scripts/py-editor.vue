<template lang="pug">
a-card.editor-card(:bordered="false")
  a-space.form-space(size="medium")
    a-form(layout="inline" :model="scriptForm")
      a-form-item(:label="$t('dataExplorer.scriptName')")
        a-input(
          v-model:model-value="scriptForm.scriptName"
          v-bind:disabled="!isNewScript"
          :placeholder="$t('dataExplorer.input')"
        ) 
    a-space
      a-button(v-if="isChanged" @click="saveCurrentScript()") {{ $t('dataExplorer.saveScript') }}
      a-button(v-if="isChanged" @click="saveScriptAndRun()") {{ $t('dataExplorer.saveAndRun') }}
      a-button(v-if="ifCanRun" @click="run()")
        .mr-4
          icon-loading(v-if="secondaryCodeRunning" spin)
          icon-play-arrow(v-else)
        | {{ $t('dataExplorer.runScriptAction') }}
  CodeMirror(
    v-model="pythonCode"
    :style="style"
    :spellcheck="spellcheck"
    :autofocus="autofocus"
    :indent-with-tab="indentWithTab"
    :tabSize="tabSize"
    :extensions="extensions"
    @ready="handleReady"
    @update="codeUpdate"
  )
</template>

<script lang="ts" name="PyEditor" setup>
  import { Codemirror as CodeMirror } from 'vue-codemirror'
  import { oneDark } from '@codemirror/theme-one-dark'
  import { python } from '@codemirror/lang-python'
  import usePythonCode from '@/hooks/python-code'
  import { useCodeRunStore } from '@/store'

  export interface Props {
    spellcheck?: boolean
    autofocus?: boolean
    indentWithTab?: boolean
    tabSize?: number
  }
  const props = withDefaults(defineProps<Props>(), {
    spellcheck: true,
    autofocus: true,
    indentWithTab: true,
    tabSize: 2,
  })

  const route = useRoute()
  const dataBaseStore = useDataBaseStore()
  const secondaryCodeRunning = ref(false)
  const { pythonCode, cursorAt, lastSavedCode, isNewScript, scriptName, isChanged, selectAfterSave, createNewScript } =
    usePythonCode()
  const { save: saveScript } = usePythonCode()
  const { runQuery } = useQueryCode()
  const { getScriptsTable } = dataBaseStore

  const codeType = 'python'
  const lineStart = ref()
  const lineEnd = ref()
  const selectedCode = ref()
  const view = shallowRef()

  const scriptForm = ref({
    scriptName,
  })

  const ifCanRun = computed(() => {
    if (!isChanged.value && pythonCode.value !== '' && scriptForm.value.scriptName) {
      return true
    }
    return false
  })
  const handleReady = (payload: any) => {
    view.value = payload.view
  }
  const codeUpdate = () => {
    if (view.value) {
      const { state } = view.value
      const { ranges } = state.selection
      cursorAt.value = [ranges[0].from, ranges[0].to]
      lineStart.value = state.doc.lineAt(ranges[0].from).number
      lineEnd.value = state.doc.lineAt(ranges[0].to).number
      if (state.doc.text) {
        selectedCode.value = state.doc.text.slice(lineStart.value - 1, lineEnd.value).join(' ')
      } else {
        let tempCode: Array<string> = []
        state.doc.children.forEach((leaf: { text: [] }) => {
          tempCode = tempCode.concat(leaf.text)
        })
        selectedCode.value = tempCode.slice(lineStart.value - 1, lineEnd.value).join(' ')
      }
    }
  }

  const refreshTableData = dataBaseStore.getTables

  // extensions: Passed to CodeMirror EditorState.create({ extensions })
  const style = {
    height: '250px',
  }

  const extensions = [python(), oneDark]
  const saveCurrentScript = async () => {
    const routeName = route.name as string

    try {
      await saveScript(scriptForm.value.scriptName, pythonCode.value.trim())
      await getScriptsTable()
      selectAfterSave(scriptForm.value.scriptName)
    } catch (error: any) {
      // error
    }
  }
  const saveScriptAndRun = async () => {
    const routeName = route.name as string
    try {
      secondaryCodeRunning.value = true
      await saveScript(scriptForm.value.scriptName, pythonCode.value.trim())
      lastSavedCode.value = pythonCode.value
      await runQuery(scriptForm.value.scriptName, codeType)
      await getScriptsTable()
      selectAfterSave(scriptForm.value.scriptName)
    } catch (error) {
      // error
    }
    secondaryCodeRunning.value = false
  }

  const run = async () => {
    const routeName = route.name as string

    secondaryCodeRunning.value = true
    await runQuery(scriptForm.value.scriptName, codeType)
    secondaryCodeRunning.value = false
  }
</script>
