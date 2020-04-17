<template>
	<div id="todo-app">
		<header style="font-size: 50px;">todos</header>
		
		<div class="todo-body">
			<span class="icon-down el-icon-arrow-down" 
				v-show="todoList.length>0"
				@click="selectAllTodos">
			</span>
				
			<input type="text"
				class="add_todos" 
				placeholder="What needs to be done?" 
				@keyup.enter="addTodo"
				v-model="newTodoContent">
				
			<ul class="todo-list">
				<li 
					v-for="(todo,index) in todoList"
					:key="index"
					class="content_todoList"
					@mouseover="todo.showDelete=true" 
					@mouseleave="todo.showDelete=false"
					v-show="selectedStatus=='All' ||
						(selectedStatus=='Active'&&todo.isCompleted==false) ||
						(selectedStatus=='Completed'&&todo.isCompleted==true)"
				>
					<input type="checkbox" class="complete-checkbox" v-model="todo.isCompleted">
						
					<div 
						class="todo-content" 
						@dblclick="todo.isEditing=true" 
						v-show="!todo.isEditing" 
						:class="{completed:todo.isCompleted}">
						{{todo.content}}
					</div>
						
					<input type="text" class="todo-content todo-edit-input"
						v-model="todo.content"
						v-show="todo.isEditing"
						@keyup.enter="todo.isEditing=false"
						@blur="todo.isEditing=false">
						
					<span class="el-icon-close content_todoList_delete" :class="{'show-delete': todo.showDelete}" @click="deleteTodo(index)"></span>
				</li>
			</ul>
				
				<div class="under-list-option" v-show="todoList.length>0">
					<div class="no-finish-todo-count">
						<span>{{noFinishTodoCount}}</span>&nbsp;items left
					</div>
					
					<div class="todos-status">
						<a href="#" 
							:class="{active:selectedStatus==status}" 
							v-for="status in optionalTodoStatus" 
							@click="selectedStatus=status" 
							:key="status">
							{{status}}
						</a>
					</div>
					
					<div class="clear-completed-todos" @click="clearCompletedTodos">
						<a href="#" v-if="noFinishTodoCount<todoList.length">clear completed</a>
						<a href="#" v-else></a>
					</div>
					
				</div>
			</div>
	</div>
</template>

<script>
export default {
	name: 'todos',
	props: {},
	data(){
		return {
			newTodoContent:'',
			todoList: [],
			// todo的结构如下
			// todo:{
			// 	content: this.newTodoContent,
			// 	isEditing: false,
			// 	showDelete: false,
			// 	isCompleted: false
			// }
			optionalTodoStatus: ["All", "Active", "Completed"],
			selectedStatus: "All",
		}
	},
	watch:{
		todoList:{
			handler(){
				localStorage.setItem("todoList", JSON.stringify(this.todoList))
			},
			deep:true //true 深度监听
		}
	},
	created() {
		//使用本地localStorage存储todos信息，以代替后端数据库
		let todoList = localStorage.getItem('todoList')
		this.todoList = JSON.parse(todoList) || []
	},
	computed: {
		noFinishTodoCount(){
			//代办Todo数
			let noFinishTodoCount = 0;
			this.todoList.forEach(todo=>{
				if(!todo.isCompleted)
					noFinishTodoCount++;
			})
			return noFinishTodoCount;
		}
	},
	methods: {
		addTodo() {
			//输入内容为空
			if(this.newTodoContent === "") {
				return;
			}
			
			this.todoList = this.todoList.concat({
				content: this.newTodoContent, //todo content
				isEditing: false,
				showDelete: false, //是否展示删除图标
				isCompleted: false
			})
			this.newTodoContent = "";//添加todo成功，将输入内容清空
			//存入localStorage
			localStorage.setItem("todoList", JSON.stringify(this.todoList))
		},
		
		deleteTodo(index){
			//根据下标 删除todo
			this.todoList.splice(index, 1)
			//更新localStorage
			localStorage.setItem("todoList", JSON.stringify(this.todoList))
		},
		
		clearCompletedTodos() {
			//清空已完成的todoList
			this.todoList = this.todoList.filter(todo => todo.isCompleted === false)
			localStorage.setItem("todoList", JSON.stringify(this.todoList))
		},
		
		selectAllTodos(){
			//将所有todo设为已完成
			this.todoList.map(todo=>{
				todo.isCompleted = true;
			})
		},
	},
}
</script>

<style scoped>
* {
	padding: 0;
	margin: 0;
	box-sizing: border-box;
}

a{
	text-decoration: none;
}

#todo-app{
	width: 800px;
	height: 900px;
	margin: 0 auto;
	text-align: center;
	background-color: rgb(245, 245, 245);
	padding: 24px 0;
}
 
.todo-body {
	width: 72%;
	margin: 0 auto;
	box-shadow: 0 3px 3px 2px rgba(0, 0, 0, 0.25);
	position: relative;
}
.icon-down {
	position: absolute;
	font-size: 24px;
	top: 16px;
	left: 16px;
	cursor: pointer;
}
 
#todo-app .todo-body .add_todos {
	width: 100%;
	height: 56px;
	padding: 16px 56px;
	font-size: 24px;
	border: 1px solid transparent;
}
 
.todo-list {
	position: relative;
	z-index: 3;
}
 
.content_todoList {
	display: flex;
	flex-direction: row;
	border-top: 1px solid #ccc;
	font-size: 24px;
	padding: 8px;
	background-color: white;
	align-items: center;
}
 
.complete-checkbox {
	width: 20px;
	height: 20px;
	margin-left: 10px;
}
 
.todo-content {
	flex: 1;
	text-align: left;
	margin-left: 16px;
	font-size: 20px;
	padding: 6px 0;
}
 
.todo-edit-input {
	position: relative;
	z-index: 1;
}
 
.content_todoList_delete {
	position: absolute;
	right: 16px;
	color: rgb(252, 55, 55);
	font-weight: 500;
	display: none;
	cursor: pointer;
}
 
.show-delete {
	display: block;
}
.show-delete:hover {
	color: rgb(255, 0, 0);
	font-weight: 700;
}
 
.completed{
	text-decoration-line: line-through;
	color: #bbb;
}
 
::-moz-placeholder {
	color: rgb(221, 218, 218);
}
 
::-webkit-input-placeholder {
	color: rgb(221, 218, 218);
}
 
:-ms-input-placeholder {
	color: rgb(221, 218, 218);
}

.under-list-option {
	display: flex;
	justify-content: space-between;
	padding: 8px;
	font-size: 14px;
	font-weight: 300;
	color: rgb(145, 145, 145);
}
 
.no-finish-todo-count {
	width: 73px;
}

.todos-status a {
	display: inline-block;
	border: 1px solid transparent;
	border-radius: 2px;
	padding: 1px 4px;
	margin: 0 2px;
}
.todos-status a:hover {
	border-color: #bbb;
}
 
.active {
	box-shadow: 0 0 1px black;
	font-weight: bold;
}

.clear-completed-todos {
	width: 142px;
}
.clear-completed-todos a:hover {
	text-decoration-line: underline;
}

</style>
