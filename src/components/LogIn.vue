<template>
    <div>
        <form action="#" @submit.prevent="logIn">
            <img class="avatar" v-if="selectedUser?.avatar" :src="selectedUser.avatar" alt="">


            <div class="contanier">
                <label class="lab" for="user"><b>User name</b></label>
                <select name="select-user" v-model="username">

                    <option v-for="user in users" :key="user.id" :value="user.username">
                        {{ user.name }}
                    </option>

                </select>
                <label class="lab" for="psw"><b>Password</b></label>
                <input type="password" name="psw" placeholder="password..." required v-model="password">
                <button type="submit" >Login</button>

            </div>
            <div class="contanier fotter">

            </div>
        </form>
    </div>
</template>

<script lang="ts">
import router from '@/router';
import axios from 'axios';
import { computed, onMounted, ref } from 'vue';
export default {
    setup() {

        const users = ref<{ id: number, name: string, username: string, password: string, avatar: string }[]>([]);
        let password = ref<string>('');
        let username = ref<string>('');
        let selectedUser = computed(() => users.value.find(u => u.username === username.value));
        let user = localStorage.setItem('user', JSON.stringify(selectedUser.value));
        let logIn = async () => {
            return await axios.get('http://localhost:4000/todos', {
                auth: {
                    username: username.value,
                    password: password.value
                }
            }).then( (response: any) => {   
                console.log(response.data);
                             
             localStorage.setItem('user', JSON.stringify({...selectedUser.value, password: password.value }));
             router.push('/dashboard');
                 
                /* if(response.statusText == "OK") {
                    console.log(selectedUser);
                    localStorage.setItem('user', JSON.stringify(selectedUser.value));
                   
                }else {
                    alert("Please try again")
                } */
            });
        }




        const getUsers = async (): Promise<any> => {
            return await axios.get('http://localhost:4000/users')
                .then(function (response: any): any {

                    return response.data;

                })

        };
        onMounted(() => {
            getUsers().then((u) => {
                users.value = u;

            });

        })




        return {
            users, username, password, selectedUser, logIn, user
        }
    }
}
</script>

<style scoped>
@import url('@/assets/LogIn.css');
</style>
