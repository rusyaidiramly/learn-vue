<template>
  <AddTask v-show="showAddTask" @add-task="addTask" />
  <Tasks
    @delete-task="deleteTask"
    @toggle-reminder="toggleReminder"
    :tasks="tasks"
  />
</template>

<script>
import Tasks from "../components/Tasks";
import AddTask from "../components/AddTask";

export default {
  name: "Home",
  components: {
    Tasks,
    AddTask,
  },
  props: {
    showAddTask: Boolean,
  },
  data() {
    return {
      tasks: [],
    };
  },
  methods: {
    async deleteTask(id) {
      const res = await fetch(`api/tasks/${id}`, { method: "DELETE" });
      res.status === 200
        ? (this.tasks = this.tasks.filter((task) => task.id !== id))
        : alert("Delete Failed");
    },
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id);
      const res = await fetch(`api/tasks/${id}`, {
        method: "PUT",
        headers: {
          "Content-type": "application/json",
        },
        body: JSON.stringify({
          ...taskToToggle,
          reminder: !taskToToggle.reminder,
        }),
      });

      const data = await res.json();

      if (res.status === 200)
        this.tasks = this.tasks.map((task) =>
          task.id === id ? { ...task, reminder: data.reminder } : task
        );
    },
    async addTask(task) {
      const res = await fetch("api/tasks", {
        method: "POST",
        headers: {
          "Content-type": "application/json",
        },
        body: JSON.stringify(task),
      });
      const data = await res.json();
      this.tasks.push(data);
    },
    async fetchTasks() {
      const res = await fetch("api/tasks");
      const data = await res.json();
      return data;
    },
    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`);
      const data = res.json();
      return data;
    },
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>