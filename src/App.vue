<script setup lang="ts">
import { onMounted, ref, reactive } from 'vue'
import { DateTime } from 'luxon'

const stDate = ref('2022-04-04')
const edDate = ref('2022-04-20')

const dayObject = reactive<any>({})

function setMonth (base: string) {
  for (let m=-1; m<2; m++) {
    // 총 세번함
    const baseDate = DateTime.fromISO(base).plus({ months: m }) // 베이스 날짜
    const firstDay = baseDate.startOf('month') // firstday
    const lastDay = baseDate.endOf('month') // lastDay
    let days = [] // 날짜 담아줄 배열

    if (firstDay.toFormat('c') < 7) {
      // 해당월의 1일의 index가 7보다 작을때만 만듬
      for (let i=0; i<firstDay.toFormat('c'); i++) {
        days.push({
          date: firstDay.minus({days: i + 1}),
          isDisabled: true
        })
      }
    }
    // 해당월의 날짜를 만들어줌
    for (let i=0; i<lastDay.toFormat('d'); i++) {
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
    dayObject[baseDate.toFormat('yyyy-MM')] = dayArray
  }
}

function isActive (date: string) {
  if (stDate.value && edDate.value) {
    // 시작 및 종료일이 있을경우
    const start = DateTime.fromISO(stDate.value)
    const stDiff = start.diff(date, 'days').toObject().days
    // 시작일과 해당일을 diff함
    if (stDiff < 1) {
      // 시작일보다 적은경우에만
      if (stDiff == 0) {
        // 선택된 첫째날
        return 'selected-start'
      }
      const end = DateTime.fromISO(edDate.value)
      const endDiff = end.diff(date, 'days').toObject().days
      if (endDiff > -1) {
      // 종료일보다 많은경우에만
        if (endDiff == 0) {
          // 선택된 마지막날
          return 'selected-end'
        }
        return 'selected-base'
      }
    }
  }
}

onMounted(() => {
  setMonth(DateTime.now().toFormat('yyyy-MM'))
})

</script>

<style scoped>
div{margin:0; padding:0; }

.calendar-group{display: flex;  }
.calendar-wrap{width:400px; border: 1px solid #333; }
.calendar-title{display: flex; padding: 1rem 0; justify-content: space-around; text-align: center; font-weight: 600; }
.calendar-week{display: grid; grid-template-columns:1fr 1fr 1fr 1fr 1fr 1fr 1fr; }
.calendar-days.disabled{color:#ccc; }
.calendar-days.saturday{color:rgb(59 130 246); }
.calendar-days.sunday{color:rgb(239 68 68); }
.calendar-days span{display: block; padding:1rem; }
.calendar-days.selected-start span{border-radius: 100% 0 0 100%; background: rgb(147 197 253); }
.calendar-days.selected-end span{border-radius: 0 100% 100% 0; background: rgb(147 197 253); }
.calendar-days.selected-base span{background: rgb(219 234 254); }
</style>
<template>
<input type="date"
  v-model="stDate"
/>
<input type="date"
  v-model="edDate"
/>
<div class="calendar-group">
  <div class="calendar-wrap"
    v-for="(month, key) in dayObject"
  >
    <div class="calendar-title">
      {{key}}
    </div>
    <div class="calendar-base">
      <div class="calendar-week"
        v-for="week in month"
      >
        <div class="calendar-days"
          :class="[index == 6 ? 'saturday' : index == 0 ? 'sunday' : '',
          isActive(days.date)]"
          v-for="(days, index) in week"
        >
          <span v-show="!days.isDisabled">{{days.date.toFormat('d')}}</span>
        </div>
      </div>
    </div>
  </div>
</div>
</template>

