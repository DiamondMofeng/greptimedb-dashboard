<template lang="pug">
a-tabs.result-tabs.logs-tab(type="rounded")
  template(#extra)
    a-button.clear-logs-button(
      v-if="logs.length"
      type="secondary"
      status="danger"
      @click="clear"
    ) {{ $t('dataExplorer.clear') }}
  a-tab-pane(title="Logs")
    a-card(:bordered="false")
      a-list.log-list(
        v-if="logs.length"
        size="small"
        :hoverable="true"
        :bordered="false"
        :split="false"
      )
        Log(v-for="log of logs" :key="log" :log="log")
</template>

<script lang="ts" name="Log" setup>
  import type { Log } from '@/store/modules/log/types'
  import { useClipboard } from '@vueuse/core'
  import { storeToRefs } from 'pinia'

  const props = defineProps<{
    logs: Log[]
    types: string[]
  }>()

  const route = useRoute()
  const { clearLogs } = useLog()
  const { codeType } = storeToRefs(useAppStore())
  const { copy, copied } = useClipboard()

  const clear = () => {
    clearLogs(props.types)
  }
</script>
