<script setup lang="ts">
import schedule from "./assets/schedule.json";
import ScheduleTable from "./components/ScheduleTable.vue";
import { computed, ref } from "vue";

const startFirstLecture = 8 * 60 + 15; // 8:15
const sublecturePausingDuration = 5;
const sublectureDuration = 45;
const defaultPausingDuration = 10;
const largePausingDuration = 30;
const largePausingBeforeLecture = 1;
const maxLectureCount = 5;

enum Pausing {
  None,
  Lecture,
  Sublecture,
}

const currentDate = ref(new Date());
const isWeekend = computed(
  () => [0, 6].indexOf(currentDate.value.getDay()) != -1
);
const currentMinutes = computed(() => {
  return currentDate.value.getHours() * 60 + currentDate.value.getMinutes();
});

const studyWeekNum = computed(() => {
  let weekCount = 0;
  const year =
    currentDate.value.getFullYear() - Number(currentDate.value.getMonth() < 8);
  const date = new Date(year, 8, 1);
  while (date <= currentDate.value) {
    date.setDate(date.getDate() + (7 - date.getDay() + 1));
    weekCount++;
  }
  return weekCount - 1;
});

const isSecondStudyWeek = computed(() => {
  if (!studyWeekNum.value) return false;
  return (studyWeekNum.value + 1) % 2;
});

const weekSchedule = computed(() => {
  const weekSchedule = [];
  for (const daySchedule of schedule) {
    const weekdaySchedule = [];
    for (const lecture of daySchedule) {
      weekdaySchedule.push(lecture[+isSecondStudyWeek.value] || lecture[0]);
    }
    weekSchedule.push(weekdaySchedule);
  }
  return weekSchedule;
});

const currentTimeStatus = computed(() => {
  let lectureNum = 0;
  let sublectureNum = 0;
  let pausing = Pausing.None;
  let minutesLeft = currentMinutes.value - startFirstLecture;
  let step = 0;
  while (minutesLeft >= 0 && lectureNum < maxLectureCount) {
    switch (step) {
      case 0:
        minutesLeft -= sublectureDuration;
        pausing = Pausing.None;
        step = sublectureNum ? 2 : 1;
        break;
      case 1:
        minutesLeft -= sublecturePausingDuration;
        pausing = Pausing.Sublecture;
        sublectureNum = 1;
        step = 0;
        break;
      case 2:
        minutesLeft -=
          lectureNum == largePausingBeforeLecture
            ? largePausingDuration
            : defaultPausingDuration;
        pausing = Pausing.Lecture;
        lectureNum++;
        sublectureNum = 0;
        step = 0;
        break;
    }
  }
  return {
    lectureNum,
    sublectureNum,
    pausing,
    minutesLeft: -minutesLeft,
  };
});

const initLectureNum = computed(() => {
  if (isWeekend.value) return -1;
  let counter = 0;
  while (
    counter < maxLectureCount &&
    Object.keys(weekSchedule.value[currentDate.value.getDay() - 1][counter])
      .length === 0
  )
    counter++;
  return counter;
});

const lastTimeForInitLecture = computed(() => {
  if (isWeekend.value) return -1;
  let minutesLeft = currentMinutes.value - startFirstLecture;
  for (let i = 0; i < initLectureNum.value; i++) {
    minutesLeft -= sublectureDuration * 2 + sublecturePausingDuration;
    minutesLeft -=
      i == largePausingBeforeLecture
        ? largePausingDuration
        : defaultPausingDuration;
  }
  return -minutesLeft;
});

setInterval(() => {
  currentDate.value = new Date();
}, 1000);
</script>

<template>
  <header class="header">
    <p>Текущее время: {{ currentDate.toLocaleString() }}</p>
    <p>{{ isSecondStudyWeek ? "Вторая" : "Первая" }} учебная неделя</p>
    <p>{{ studyWeekNum + 1 }} неделя занятий</p>
    <p v-if="lastTimeForInitLecture > 0">
      До начала занятий осталось: {{ lastTimeForInitLecture }} мин.
    </p>
    <template v-else-if="currentTimeStatus.minutesLeft >= 0">
      <p>{{ currentTimeStatus.sublectureNum ? "II" : "I" }} полупара</p>
      <p>
        До {{ currentTimeStatus.pausing ? "начала" : "перерыва" }} осталось:
        {{ currentTimeStatus.minutesLeft }} мин.
      </p>
    </template>
  </header>
  <div class="weekSchedule">
    <div v-for="(daySchedule, index) in weekSchedule" :key="index">
      <h3>
        {{ ["Понедельник", "Вторник", "Среда", "Четверг", "Пятница"][index] }}
      </h3>
      <ScheduleTable
        :schedule="daySchedule"
        :activeNum="
          currentDate.getDay() - 1 == index ? currentTimeStatus.lectureNum : -1
        "
      />
    </div>
  </div>
</template>

<style scoped lang="scss">
.header {
  text-align: center;
  margin: 50px 10px;
}

.weekSchedule {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 0 10px;

  h3 {
    text-align: center;
  }

  @media (max-width: 650px) {
    align-items: unset;
  }
}
</style>
