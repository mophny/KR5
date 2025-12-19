<template>
  <div class="task-app">
    <h2>Практика 28: Работа с компонентами, props и emits</h2>

    <TaskInput @add-task="addTask" />

    <TaskFilter
      :current-filter="currentFilter"
      @update-filter="currentFilter = $event"
    />

    <TaskList
      :tasks="filteredTasks"
      @toggle-task="toggleTask"
      @remove-task="removeTask"
    />

    <TaskStats
      :total="tasks.length"
      :completed="completedCount"
      :active="activeCount"
    />
  </div>
</template>

<script>
import { ref, computed } from "vue";
import TaskInput from "./TaskInput.vue";
import TaskList from "./TaskList.vue";
import TaskFilter from "./TaskFilter.vue";
import TaskStats from "./TaskStats.vue";

export default {
  name: "TaskApp",
  components: { TaskInput, TaskList, TaskFilter, TaskStats },

  setup() {
    const tasks = ref([
      { id: 1, text: "Изучить props", done: false },
      { id: 2, text: "Разобраться с emits", done: true }
    ]);

    const currentFilter = ref("all");

    const addTask = (text) => {
      tasks.value.push({
        id: Date.now(),
        text,
        done: false
      });
    };

    const toggleTask = (id) => {
      const t = tasks.value.find((t) => t.id === id);
      if (t) t.done = !t.done;
    };

    const removeTask = (id) => {
      tasks.value = tasks.value.filter((t) => t.id !== id);
    };

    const filteredTasks = computed(() => {
      switch (currentFilter.value) {
        case "active":
          return tasks.value.filter((t) => !t.done);
        case "completed":
          return tasks.value.filter((t) => t.done);
        default:
          return tasks.value;
      }
    });

    const activeCount = computed(() => tasks.value.filter((t) => !t.done).length);
    const completedCount = computed(() => tasks.value.filter((t) => t.done).length);

    return {
      tasks,
      currentFilter,
      addTask,
      toggleTask,
      removeTask,
      filteredTasks,
      activeCount,
      completedCount
    };
  }
};
</script>

<style scoped>
.task-app {
  max-width: 650px;
  margin: 20px auto;
  padding: 20px;
  background: white;
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0,0,0,0.08);
}
</style>
