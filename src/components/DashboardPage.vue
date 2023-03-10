<template>
    <header v-if="user?.username">
        <img :src="user.avatar" alt="" />
        <h3>{{ user.username }}</h3>
        <button type="button" @click="logout" class="log-out-btn">LogOut</button>
    </header>
   
    <div id="todos" class="todos">
        <form class="form" @submit.prevent="createTodo">
            <input class="f-input" type="text" placeholder="Add task..." maxlength="40" v-model="task" />
            <button class="btn add" type="submit">Add</button>
        </form>
        <TransitionGroup tag="ul" name="appear">
        <li class="item" v-for="todo in todos" :key="todo.id">
            <button :class="{complete: todo.completed, toggleItem: true}" @click="() => updateTodo(todo.id, todo.completed)" >{{ todo.task }}</button>
            <button class="btn remove-item" @click="() => deleteTodo(todo.id)">Delete</button>
        </li>
    </TransitionGroup>
    </div>
</template>

<script lang="ts">
import router from '@/router'
import { onMounted, ref } from 'vue'
import axios from 'axios'

export default {
    setup() {
        let user = ref(JSON.parse(localStorage.getItem('user') || '{}'))
        const logout = () => {
            localStorage.setItem('user', '');
            router.push('/');
        }
        let task = ref<string>('');
        //const complated = ref<boolean>(false);
        let todos = ref<{}>([])
        const getAuth = () => ({
            auth: { username: user.value.username, password: user.value.password }
        });
        const getTodos = async () =>
        (todos.value = await axios
            .get('http://localhost:4000/todos', getAuth())
            .then((res) => {
                return res.data;
            }));
        const createTodo = async () => {
            if (task.value.trim() != "") {
                await axios.post('http://localhost:4000/todos', { task: task.value }, getAuth()).then(() => {
                    console.log(user.value);
                 getTodos()
                    task.value = ''
                })
            } else {
                task.value = '';
            }
        };
        const deleteTodo = async (todoId: any) =>
            await axios.delete('http://localhost:4000/todos/' + todoId, getAuth()).then(() => getTodos());
        const updateTodo = async (todoId: any, todoComplated: boolean) => {
                 const toggle = todoComplated;
                       
            await axios.put('http://localhost:4000/todos/' + todoId, { completed: toggle}, getAuth()).then(() => {
                toggle != toggle;
                getTodos();
            })};

        onMounted(async () => {
            if (user.value && user.value.username) {
                getTodos();
            }
        })
        return {
            task,
            todos,
            user,
            getTodos,
            logout,
            createTodo,
            deleteTodo,
            updateTodo
        }
    }
}
</script>

<style >
@import url(@/assets/DashboardPage.css);
.container {
    position: relative;
    padding: 0;
}
.appear-move,
.appear-enter-active,
.appear-leave-active {
  transition: all .5s cubic-bezier(0.55, 0, 0.1, 1);
}
.appear-leave-active,
.appear-enter {
  transform: scaleY(0.01);
  opacity: 0;

}
.appear-leave-active {
  position: relative;
}
</style>
