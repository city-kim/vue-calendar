<script setup lang="ts">
import { onMounted, ref, computed, PropType } from 'vue'
import { DateTime, Interval } from 'luxon'

const today = ref(DateTime.now())

const props = defineProps({
  diff_base: {
    type: Object as PropType<diffType>
  },
  diff_target: {
    type: Object as PropType<diffType>
  }
})

const emit = defineEmits(['update:date'])

interface diffType {
  start?: string
  end?: string
}

const diff_base = computed(() => props.diff_base) // 비교기준
const diff_target = computed(() => props.diff_target) // 비교대상

interface daysType { // 특정 월의 날짜 배열타입
  [key: string]: Array<Array<dayType>>
}

interface dayType { // 특정 월의 날짜 요소타입
  date: DateTime
  isDisabled: boolean
}

let dayObject = ref<daysType>({})

function setMonth (base: string) {
  dayObject.value = Object.assign({}, {})
  for (let m=-1; m<2; m++) {
    // 총 세번함
    const baseDate = DateTime.fromISO(base).plus({ months: m }) // 베이스 날짜
    const firstDay = baseDate.startOf('month') // firstday
    const lastDay = baseDate.endOf('month') // lastDay
    let days = [] // 날짜 담아줄 배열

    if (Number(firstDay.toFormat('c')) < 7) {
      // 해당월의 1일의 index가 7보다 작을때만 만듬
      for (let i=0; i<Number(firstDay.toFormat('c')); i++) {
        days.push({
          date: firstDay.minus({days: i + 1}),
          isDisabled: true
        })
      }
    }
    // 해당월의 날짜를 만들어줌
    for (let i=0; i<Number(lastDay.toFormat('d')); i++) {
      days.push({
        date: firstDay.plus({days: i}),
        isDisabled: false
      })
    }

    let dayArray = []
    // 7개씩 배열 분할하기
    for (let i=0; i<days.length; i+=7) {
      dayArray.push(days.slice(i, i+7))
    }
    dayObject.value[baseDate.toFormat('yyyy-MM')] = dayArray
  }
}

function isActive (date: DateTime, isDisabled: boolean) {
  let className = ''
  if (!isDisabled) {
    if (diff_base.value?.start) {
      // 비교시작일이 있을때
      if (!diff_base.value.end) {
        // 종료일이 없지만 날짜는 같은경우
        if (date.hasSame(DateTime.fromISO(diff_base.value?.start), 'day')) {
          // 시작일과 같은경우
          className = `selected-base-start`
        }
      } else {
        // 둘다있음
        let classTarget = isContainDate(date, diff_base.value.start, diff_base.value.end)
        if (classTarget) {
          className = `selected-base-${classTarget}`
        }
      }
    }

    if (diff_target.value?.start) {
      // 비교대상이 있을때
      if (!diff_target.value?.end) {
        // 종료일이 없는경우
        if (date.hasSame(DateTime.fromISO(diff_target.value?.start), 'day')) {
          className = `selected-diff-start`
        }
      } else {
        let classTarget = isContainDate(date, diff_target.value.start, diff_target.value.end)
        if (classTarget) {
          className = `selected-diff-${classTarget}`
        }
      }
    }
  }

  return className
}

function isContainDate (date: DateTime, start: string, end: string) {
  const stDiff = date.diff(DateTime.fromISO(start), 'days').toObject().days as Number
  // 시작일과 해당일을 diff함
  if (stDiff > -1) {
    // 시작일과 같거나 큰 날짜만
    if (stDiff == 0) {
      // 0일경우 첫째날
      return 'start'
    }
    const endDiff = date.diff(DateTime.fromISO(end), 'days').toObject().days as Number
    if (endDiff < 1) {
    // 그중 종료일과 같거나 작은 날짜만
      if (endDiff == 0) {
        // 선택된 마지막날
        return 'end'
      }
      return 'base'
    }
  }
  return ''
}

function changeMonth (x: number) {
  today.value = today.value.plus({ months: x })
  setMonth(today.value.toFormat('yyyy-MM'))
}

onMounted(() => {
  setMonth(today.value.toFormat('yyyy-MM'))
})

</script>

<style scoped>
div{margin:0; padding:0; }

.calendar-group{display: flex;  }
.calendar-wrap{width:400px; border: 1px solid #333; }
.calendar-title{display: flex; padding: 1rem; justify-content: space-between; text-align: center; font-weight: 600; }
.calendar-title svg{width:1rem; cursor: pointer; }
.calendar-week{display: grid; grid-template-columns:1fr 1fr 1fr 1fr 1fr 1fr 1fr; }
.calendar-days.disabled{color:#ccc; }
.calendar-days.saturday{color:rgb(59 130 246); }
.calendar-days.sunday{color:rgb(239 68 68); }
.calendar-days span{display: block; padding:1rem; text-align: center; cursor: pointer; }
.calendar-days.selected-base-start span{border-radius: 100% 0 0 100%; background: rgb(147 197 253); }
.calendar-days.selected-base-end span{border-radius: 0 100% 100% 0; background: rgb(147 197 253); }
.calendar-days.selected-base-base span{background: rgb(219 234 254); }

.calendar-days.selected-diff-start span{border-radius: 100% 0 0 100%; background: rgb(249 115 22); }
.calendar-days.selected-diff-end span{border-radius: 0 100% 100% 0; background: rgb(249 115 22); }
.calendar-days.selected-diff-base span{background: rgb(255 237 213); }
</style>
<template>
<input type="date"/>
<input type="date"/>
<div class="calendar-group">
  <div class="calendar-wrap"
    v-for="(month, key) in dayObject"
  >
    <div class="calendar-title">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16"
        @click="changeMonth(-1)"
      >
        <path fill-rule="evenodd" d="M8.354 1.646a.5.5 0 0 1 0 .708L2.707 8l5.647 5.646a.5.5 0 0 1-.708.708l-6-6a.5.5 0 0 1 0-.708l6-6a.5.5 0 0 1 .708 0z"/>
        <path fill-rule="evenodd" d="M12.354 1.646a.5.5 0 0 1 0 .708L6.707 8l5.647 5.646a.5.5 0 0 1-.708.708l-6-6a.5.5 0 0 1 0-.708l6-6a.5.5 0 0 1 .708 0z"/>
      </svg>
      <div>{{key}}</div>
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16"
        @click="changeMonth(1)"
      >
        <path fill-rule="evenodd" d="M3.646 1.646a.5.5 0 0 1 .708 0l6 6a.5.5 0 0 1 0 .708l-6 6a.5.5 0 0 1-.708-.708L9.293 8 3.646 2.354a.5.5 0 0 1 0-.708z"/>
        <path fill-rule="evenodd" d="M7.646 1.646a.5.5 0 0 1 .708 0l6 6a.5.5 0 0 1 0 .708l-6 6a.5.5 0 0 1-.708-.708L13.293 8 7.646 2.354a.5.5 0 0 1 0-.708z"/>
      </svg>
    </div>
    <div class="calendar-base">
      <div class="calendar-week"
        v-for="week in month"
      >
        <div class="calendar-days"
          :class="[index == 6 ? 'saturday' : index == 0 ? 'sunday' : '',
          isActive(days.date, days.isDisabled)]"
          v-for="(days, index) in week"
        >
          <span v-show="!days.isDisabled"
            @click="emit('update:date', days.date)"
          >{{days.date.toFormat('d')}}</span>
        </div>
      </div>
    </div>
  </div>
</div>
</template>
