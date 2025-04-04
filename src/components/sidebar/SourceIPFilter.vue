<template>
  <select
    v-if="!horizontal"
    class="join-item select select-sm"
    v-model="sourceIPFilter"
  >
    <option :value="null">{{ $t('allSourceIP') }}</option>
    <option
      v-if="sourceIPOpts.every((i) => !isEqual(i.value, sourceIPFilter)) && sourceIPFilter !== null"
      :value="sourceIPFilter"
    >
      {{ getIPLabelFromMap(sourceIPFilter[0]) }}
    </option>
    <option
      v-for="opt in sourceIPOpts"
      :key="opt.value.join(',')"
      :value="opt.value"
    >
      {{ opt.label }}
    </option>
  </select>
</template>

<script setup lang="ts">
import { getIPLabelFromMap } from '@/helper'
import { connections, sourceIPFilter } from '@/store/connections'
import { isEqual, uniq } from 'lodash'
import { computed, ref, watch } from 'vue'
defineProps<{
  horizontal?: boolean
}>()

const sourceIPs = computed(() => {
  return uniq(connections.value.map((conn) => conn.metadata.sourceIP)).sort()
})
const sourceIPOpts = ref<{ label: string; value: string[] }[]>([])

// do not use computed here for firefox
watch(
  sourceIPs,
  (value, oldValue) => {
    if (isEqual(value, oldValue)) return
    sourceIPOpts.value = []

    sourceIPs.value.forEach((ip) => {
      const label = getIPLabelFromMap(ip)
      const index = sourceIPOpts.value.findIndex((opt) => opt.label === label)

      if (index === -1) {
        sourceIPOpts.value.push({
          label,
          value: [ip],
        })
      } else {
        sourceIPOpts.value[index].value.push(ip)
      }
    })
  },
  {
    immediate: true,
  },
)
</script>
