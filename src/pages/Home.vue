<template>
	<div v-show="showAddTask">
		<AddTask @add-task="addTask" />
	</div>
	<Tasks
		@toggle-reminder="toggleReminder"
		@delete-task="deleteTask"
		:tasks="tasks"
	/>
</template>

<script>
import Tasks from '../components/Tasks.vue';
import AddTask from '../components/AddTask.vue';

export default {
	name: 'Home',
	props: {
		showAddTask: Boolean,
	},
	components: {
		Tasks,
		AddTask,
	},
	data() {
		return {
			tasks: [],
		};
	},
	methods: {
		async addTask(task) {
			const data = await fetch('api/tasks', {
				method: 'POST',
				headers: {
					'Content-type': 'application/json',
				},
				body: JSON.stringify(task),
			}).then((res) => res.json());

			this.tasks = [...this.tasks, data];
		},
		async deleteTask(id) {
			if (confirm('are you sure?')) {
				const res = await fetch(`api/tasks/${id}`, {
					method: 'DELETE',
				});

				res.status === 200
					? (this.tasks = this.tasks.filter((task) => task.id !== id))
					: alert('error deleting tasks');
			}
		},
		async toggleReminder(id) {
			const taskToToggle = await this.fetchTask(id);

			const updateTask = { ...taskToToggle, reminder: !taskToToggle.reminder };

			const data = await fetch(`api/tasks/${id}`, {
				method: 'PUT',
				headers: {
					'Content-type': 'application/json',
				},
				body: JSON.stringify(updateTask),
			}).then((res) => res.json());

			this.tasks = this.tasks.map((task) =>
				task.id === id ? { ...task, reminder: data.reminder } : task
			);
		},
		async fetchTasks() {
			const data = await fetch('api/tasks').then((res) => res.json());

			return data;
		},
		async fetchTask(id) {
			const data = await fetch(`api/tasks/${id}`).then((res) => res.json());

			return data;
		},
	},
	async created() {
		this.tasks = await this.fetchTasks();
	},
};
</script>
