<template>
  <div class="task-list">
    <TaskItem
      v-for="task in tasks"
      :key="task.id"
      :task="task"
      @toggle-task="$emit('toggle-task', $event)"
      @remove-task="$emit('remove-task', $event)"
    />
  </div>
</template>

<script>
import TaskItem from "./TaskItem.vue";

export default {
  name: "TaskList",
  components: { TaskItem },
  props: {
    tasks: { type: Array, required: true }
  },
  emits: ["toggle-task", "remove-task"]
};
</script>

<style scoped>
.task-list {
  margin-bottom: 20px;
}
</style>
