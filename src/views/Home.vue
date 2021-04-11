<template>
  <div v-if="showAddTask">
    <AddTask @add-task="addTask" />
  </div>
  <Tasks
    @toggle-reminder="toggleReminder"
    @delete-task="deleteTask"
    :tasks="tasks"
  />
</template>

<script>
import Tasks from "../components/Tasks";
import AddTask from "../components/AddTask";

export default {
  name: "Home",
  components: { Tasks, AddTask },
  props: { showAddTask: Boolean },
  data() {
    return {
      tasks: [],
    };
  },
  methods: {
    async addTask(task) {
      const res = await fetch("api/tasks", {
        method: "POST",
        headers: {
          "Content-type": "application/json",
        },
        body: JSON.stringify(task),
      });

      const data = await res.json();

      this.tasks = [...this.tasks, data];
    },

    async toggleReminder(id) {
      const taskToggle = await this.fetchTask(id);

      const updTask = { ...taskToggle, reminder: !taskToggle.reminder };

      const res = await fetch(`/api/tasks/${id}`, {
        method: "PUT",
        headers: { "Content-type": "application/json" },
        body: JSON.stringify(updTask),
      });

      const data = res.json();

      this.tasks = this.tasks.map((task) =>
        task.id === id ? { ...task, reminder: data.reminder } : task
      );
    },

    async deleteTask(id) {
      if (confirm("Are You Sure")) {
        const res = await fetch(`api/tasks/${id}`, { method: "DELETE" });

        res.status === 200
          ? (this.tasks = this.tasks.filter((task) => task.id !== id))
          : alert("Error Deleting Task");
      }
    },

    async fetchTasks() {
      const res = await fetch("api/tasks");
      const data = await res.json();
      return data;
    },

    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`);
      const data = await res.json();
      return data;
    },
  },

  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>
