<template>
  <div class="task-item" :class="{ done: task.done }">
    <label class="task-label">
      <input type="checkbox" :checked="task.done" @change="$emit('toggle-task', task.id)" />
      <span>{{ task.text }}</span>
    </label>

    <button class="remove-btn" @click="$emit('remove-task', task.id)">✖</button>
  </div>
</template>

<script>
export default {
  name: "TaskItem",
  props: {
    task: { type: Object, required: true }
  },
  emits: ["toggle-task", "remove-task"]
};
</script>

<style scoped>
.task-item {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  border: 1px solid #eee;
  border-radius: 6px;
  margin-bottom: 10px;
  background: #fafafa;
}
.task-item.done span {
  text-decoration: line-through;
  color: #777;
}
.remove-btn {
  border: none;
  background: #ff6b6b;
  padding: 5px 9px;
  border-radius: 4px;
  color: white;
  cursor: pointer;
}
.task-label {
  display: flex;
  gap: 10px;
  cursor: pointer;
}
</style>
