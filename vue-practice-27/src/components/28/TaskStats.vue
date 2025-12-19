<template>
  <div class="stats">
    <p>Всего задач: <strong>{{ total }}</strong></p>
    <p>Активных: <strong>{{ active }}</strong></p>
    <p>Завершённых: <strong>{{ completed }}</strong></p>
  </div>
</template>

<script>
export default {
  name: "TaskStats",
  props: {
    total: Number,
    active: Number,
    completed: Number
  }
};
</script>

<style scoped>
.stats {
  padding: 15px;
  background: #f8f9fa;
  border-radius: 8px;
}
</style>
