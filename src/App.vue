<script setup lang="ts">
import { ref, reactive } from 'vue'
import { DateTime } from 'luxon'
import VueLuxonCalendar from '@/components/vue_luxon_calendar.vue'

const selectType = ref<'base_start'|'base_end'|'diff_start'|'diff_end'>('base_start')

const diff_base = reactive({ // yyyy-MM-dd
  start: '',
  end: ''
})

const diff_target = reactive({ // yyyy-MM-dd
  start: '',
  end: ''
})

function updateDate (x: DateTime) {
  switch (selectType.value) {
    case 'base_start':
      selectType.value = 'base_end'
      diff_base.start = x.toFormat('yyyy-MM-dd')
      break;
    case 'base_end':
      selectType.value = 'diff_start'
      diff_base.end = x.toFormat('yyyy-MM-dd')
      break;
    case 'diff_start':
      selectType.value = 'diff_end'
      diff_target.start = x.toFormat('yyyy-MM-dd')
      break;
    case 'diff_end':
      selectType.value = 'base_start'
      diff_target.end = x.toFormat('yyyy-MM-dd')
      break;
  }
}

</script>

<style>
</style>
<template>
HI!
<VueLuxonCalendar
  :diff_base="diff_base"
  :diff_target="diff_target"
  @update:date="updateDate"
></VueLuxonCalendar>
</template>

