<script setup lang="ts">
const props = defineProps(["schedule", "activeNum"]);
</script>

<template>
  <table class="schedule-table">
    <thead>
      <tr>
        <th>#</th>
        <th>Дисциплина</th>
        <th>Преподаватель</th>
        <th>Кабинет</th>
        <th>Корпус</th>
      </tr>
    </thead>
    <tbody>
      <tr
        :class="{ active: props.activeNum == index }"
        v-for="(
          { disciplineName, teacherName, auditorium, building }, index
        ) in props.schedule"
        :key="index"
      >
        <td data-label="#">{{ index + 1 }}</td>
        <td data-label="Дисциплина">{{ disciplineName }}</td>
        <td data-label="Преподаватель">{{ teacherName }}</td>
        <td data-label="Кабинет">{{ auditorium }}</td>
        <td data-label="Корпус">{{ building }}</td>
      </tr>
    </tbody>
  </table>
</template>

<style scoped lang="scss">
.schedule-table {
  border-collapse: collapse;
  margin: 25px 0;
  font-size: 0.9em;
  font-family: sans-serif;
  width: 600px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);

  thead tr {
    background-color: #009879;
    color: #ffffff;
    text-align: left;
  }

  th,
  td {
    padding: 12px 15px;
  }

  tbody {
    tr {
      border-bottom: 1px solid #dddddd;

      &:nth-of-type(even) {
        background-color: #f3f3f3;
      }

      &.active {
        font-weight: bold;
        color: #009879;
      }
    }
  }

  @media (max-width: 650px) {
    border: 0;
    width: 100%;

    thead {
      border: none;
      clip: rect(0 0 0 0);
      height: 1px;
      margin: -1px;
      overflow: hidden;
      padding: 0;
      position: absolute;
      width: 1px;
    }

    tr {
      border-bottom: 3px solid #ddd;
      display: block;
      margin-bottom: 0.625em;
    }

    td {
      border-bottom: 1px solid #ddd;
      display: block;
      font-size: 0.8em;
      text-align: right;
    }

    td::before {
      content: attr(data-label);
      float: left;
      font-weight: bold;
      text-transform: uppercase;
    }

    td:last-child {
      border-bottom: 0;
    }
  }
}
</style>
