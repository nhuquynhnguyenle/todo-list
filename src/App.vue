<script setup>
// Import các hàm cần thiết từ Vue
import { ref, onMounted, computed, watch } from 'vue'

// Khai báo biến phản ứng (reactive)
// todos: danh sách công việc (mảng rỗng ban đầu)
// name: tên người dùng
const todos = ref([])
const name = ref('')

// input_content: nội dung công việc nhập vào ô input
// input_category: loại công việc (business hoặc personal)
const input_content = ref('')
const input_category = ref(null)

// computed: tạo danh sách todos sắp xếp theo thứ tự thời gian tạo (tăng dần)
const todos_asc = computed(() => todos.value.sort((a,b) =>{
	return a.createdAt - b.createdAt
}))

// Theo dõi biến name, mỗi khi thay đổi thì lưu lại vào localStorage
watch(name, (newVal) => {
	localStorage.setItem('name', newVal)
})

// Theo dõi biến todos, mỗi khi thay đổi thì lưu lại vào localStorage
// deep: true nghĩa là theo dõi cả thay đổi bên trong mảng (ví dụ sửa nội dung, tick done)
watch(todos, (newVal) => {
	localStorage.setItem('todos', JSON.stringify(newVal))
}, {
	deep: true
})

// Hàm thêm công việc mới
const addTodo = () => {
	// Nếu chưa nhập nội dung hoặc chưa chọn loại thì không thêm
	if (input_content.value.trim() === '' || input_category.value === null) {
		return
	}

	// Thêm 1 đối tượng todo mới vào mảng todos
	todos.value.push({
		content: input_content.value,      // nội dung công việc
		category: input_category.value,    // loại công việc
		done: false,                       // chưa hoàn thành
		editable: false,                   // chưa bật chế độ chỉnh sửa
		createdAt: new Date().getTime()    // thời gian tạo (dùng để sắp xếp)
	})
}

// Hàm xóa 1 công việc khỏi danh sách
const removeTodo = (todo) => {
	// Giữ lại những phần tử khác với todo được chọn
	todos.value = todos.value.filter((t) => t !== todo)
}

// Khi component được mount (hiển thị lần đầu)
// => Lấy lại dữ liệu đã lưu trong localStorage (nếu có)
onMounted(() => {
	name.value = localStorage.getItem('name') || ''
	todos.value = JSON.parse(localStorage.getItem('todos')) || []
})
</script>

<template>
	<main class="app">
		
		<!-- Phần nhập tên người dùng -->
		<section class="greeting">
			<h2 class="title">
				What's up, 
				<!-- Ô nhập tên, liên kết 2 chiều với biến name -->
				<input type="text" id="name" placeholder="Name here" v-model="name">
			</h2>
		</section>

		<!-- Phần tạo công việc mới -->
		<section class="create-todo">
			<h3>CREATE A TODO</h3>

			<!-- Form thêm công việc -->
			<!-- @submit.prevent: gọi hàm addTodo và ngăn reload trang -->
			<form id="new-todo-form" @submit.prevent="addTodo">
				<h4>What's on your todo list?</h4>

				<!-- Ô nhập nội dung công việc -->
				<input 
					type="text" 
					name="content" 
					id="content" 
					placeholder="e.g. make a video"
					v-model="input_content" />
				
				<h4>Pick a category</h4>
				<div class="options">

					<!-- Radio chọn loại công việc: Business -->
					<label>
						<input 
							type="radio" 
							name="category" 
							id="category1" 
							value="business"
							v-model="input_category" />
						<span class="bubble business"></span>
						<div>Business</div>
					</label>

					<!-- Radio chọn loại công việc: Personal -->
					<label>
						<input 
							type="radio" 
							name="category" 
							id="category2" 
							value="personal"
							v-model="input_category" />
						<span class="bubble personal"></span>
						<div>Personal</div>
					</label>

				</div>

				<!-- Nút thêm công việc -->
				<input type="submit" value="Add todo" />
			</form>
		</section>

		<!-- Phần hiển thị danh sách công việc -->
		<section class="todo-list">
			<h3>TODO LIST</h3>
			<div class="list" id="todo-list">

				<!-- Lặp qua danh sách todos_asc -->
				<!-- todo.done && 'done': nếu todo.done là true thì thêm class 'done' -->
				<div v-for="todo in todos_asc" :class="`todo-item ${todo.done && 'done'}`">
					
					<!-- Checkbox đánh dấu hoàn thành -->
					<label>
						<input type="checkbox" v-model="todo.done" />
						<!-- Bong bóng đổi màu tùy theo loại -->
						<span :class="`bubble ${
							todo.category == 'business' 
								? 'business' 
								: 'personal'
						}`"></span>
					</label>

					<!-- Ô nhập nội dung công việc (cho phép chỉnh sửa trực tiếp) -->
					<div class="todo-content">
						<input type="text" v-model="todo.content" />
					</div>

					<!-- Nút xóa công việc -->
					<div class="actions">
						<button class="delete" @click="removeTodo(todo)">Delete</button>
					</div>
				</div>

			</div>
		</section>

	</main>
</template>
