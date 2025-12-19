<template>
  <div class="task-input">
    <input
      v-model="taskText"
      @keyup.enter="submitTask"
      placeholder="Введите задачу..."
      class="text-input"
    />
    <button @click="submitTask" class="add-btn">Добавить</button>
  </div>
</template>

<script>
import { ref } from "vue";

export default {
  name: "TaskInput",
  emits: ["add-task"],

  setup(_, { emit }) {
    const taskText = ref("");

    const submitTask = () => {
      if (!taskText.value.trim()) return;
      emit("add-task", taskText.value.trim());
      taskText.value = "";
    };

    return { taskText, submitTask };
  }
};
</script>

<style scoped>
.task-input {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}
.text-input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 6px;
}
.add-btn {
  padding: 8px 14px;
  background: #42b883;
  border: none;
  color: white;
  border-radius: 6px;
  cursor: pointer;
}
</style>
