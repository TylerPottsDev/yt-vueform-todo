<template>
	<div class="page">
		<div class="container">
			<h1>Tasks app</h1>

			<!-- Adding a task -->
			<Vueform size="lg" :endpoint="createTask">

				<!-- Task input -->
				<TextElement
					name="task"
					placeholder="Add a task"
					floating="Task name"
					rules="required"
				/>

				<!-- Task type -->
				<RadiogroupElement
					name="type"
					:items="['Personal', 'Business']"
					view="tabs"
					default="Personal"
				/>

				<!-- Submit -->
				<ButtonElement
					name="button"
					align="right"
					submits
				>
					Submit
				</ButtonElement>
			</Vueform>

			<hr class="divider" />

			<!-- Task list -->
			<Vueform v-model="tasksModel" sync> <!-- `sync` to update the list when `tasksModel` changes -->

				<!-- List of tasks -->
				<ListElement
					name="tasks"
					:controls="{
						add: false, // to disable adding new elements to the list
					}"
					:add-class="{
						handle: 'task-sort-handle' // for fixing top position of the handle
					}"
					sort
					@sort="syncToStorage"
					@remove="syncToStorage"
				>
					<template #default="{ index }">

						<!-- Task wrapper -->
						<ObjectElement
							:name="index"
							:add-class="[
								'task-container',
								tasksModel.tasks[index].type === 'Personal' ? 'is-personal' : 'is-business'
							]">

						<!-- Edit button, using the task name with index as label - only visible when not editing -->
						<ButtonElement
							:label="`#${index + 1} - ${tasksModel.tasks[index].task}`"
							name="edit"
							align="right"
							:conditions="[
								['editing', '!=', index]
							]"
							:columns="{
								label: 8,
							}"
							@click="edit(index)"
						>
							Edit
						</ButtonElement>

						<!-- Task input when editing -->
						<TextElement 
							name="task"
							:conditions="[
								['editing', index]
							]"
							:columns="6"
						/>

						<!-- Task type when editing -->
						<RadiogroupElement 
							name="type"
							view="tabs"
							:conditions="[
								['editing', index]
							]"
							:columns="2"
							:items="{
								'Personal': 'P',
								'Business': 'B',
							}"
						/>

						<!-- Cancel task editing -->
						<ButtonElement
							name="cancel"
							:conditions="[
								['editing', index]
							]"
							:columns="2"
							danger
							full
							@click="cancel"
						>
							Cancel
						</ButtonElement>

						<!-- Save task -->
						<ButtonElement
							name="save"
							:conditions="[
								['editing', index]
							]"
							:columns="2"
							full
							@click="save"
						>
							Save
						</ButtonElement>

						</ObjectElement>
					</template>
				</ListElement>

				<!-- Store which field we're editing so that conditions can rely on this -->
				<HiddenElement name="editing" />
			</Vueform>
		</div>
	</div>
</template>

<script setup>
import { onMounted, ref } from 'vue'

// The model for the task list form
const tasksModel = ref({
	tasks: [], // list of tasks
	editing: null, // current task we're editing
})

// Creating a task and saving it to localStorage
const createTask = (data, form$) => {
	addToStorage(form$.data) // first add to localStorage
	syncFromStorage() // then sync the `tasksModel` from localStorage

	form$.reset() // then reset the form
	// form$.clear() // This is used to completely wipe the form instead of resetting
}

// Adds a new task to the localStorage
const addToStorage = (data) => {
	let storageData = localStorage.getItem('tasks')
	storageData = storageData ? JSON.parse(storageData) : []

	storageData.push(data)
	localStorage.setItem('tasks', JSON.stringify(storageData))
}

// Syncs the localStorage to `tasksModel`
const syncFromStorage = () => {
	let tasks = localStorage.getItem('tasks')

	tasksModel.value = {
		tasks: tasks ? JSON.parse(tasks) : []
	}
}

// Syncs the `tasksModel.tasks` to localStorage
const syncToStorage = () => {
	localStorage.setItem('tasks', JSON.stringify(tasksModel.value.tasks))
}

// Sets the tasks to edit
const edit = (index) => {
	tasksModel.value.editing = index
}

// Cancels the task to editing
const cancel = (index) => {
	tasksModel.value.editing = null
	syncFromStorage()
}

// Saves the task
const save = () => {
	syncToStorage()
	tasksModel.value.editing = null
}

// Sync the `tasksModel` from localStorage upon pageload
onMounted(() => {
	syncFromStorage()
})

</script>

<style lang="scss">
.page {
	background: #f1f5f9;
	width: 100%;
	min-height: 100vh;
	padding-top: 2rem;
}

.container {
	max-width: 600px;
	margin: 0 auto;
}

h1 {
	font-size: 48px;
	margin-bottom: 2rem;
	font-weight: 600;
}

.divider {
	margin: 2rem 0;
	border-color: #e2e8f0;
}

.task-container {
	background: #ffffff;
	padding: 1rem;

	&.is-personal {
		border-left: 3px solid green;
	}

	&.is-business {
		border-left: 3px solid purple;
	}
}

.task-wrapper {
	display: flex;
	align-items: center;
}

.vf-list-handle.task-sort-handle {
	top: 1rem;
}
</style>