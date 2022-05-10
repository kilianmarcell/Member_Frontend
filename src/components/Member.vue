<template>
    <div>
        <nav class="container pt-5">
            <a class="ps-5 px-2" href="#createMember">Új tag felvétele</a>
            <a class="px-2" href="https://petrik.hu/">Petrik honlap</a>
            <h1 class="container pt-3">Petrik Könyvklub</h1>
        </nav>
        <div class="container mt-5">
            <div class="row">
                <div v-for="m in members" :key="m.id" class="col-xl-4 col-md-6 col-12">
                    <div class="card p-5 m-2">
                        <p class="fs-1">{{ m.name }}</p>
                        <p class="fs-4">Született: {{ m.birth_date }}</p>
                        <p class="fs-4">Csatlakozott: {{ m.created_at }}</p>
                        <img class="w-75" v-if="m.gender == 'M'" src="../assets/male.png">
                        <img class="w-75" v-if="m.gender == 'F'" src="../assets/female.png">
                        <img class="w-75" v-if="m.gender == null" src="../assets/other.png">
                        <button class="btn btn-primary fs-4" @click="pay(m.id)">Tagdíj befizetés</button>
                    </div>
                </div>
            </div>
            <div id="createMember" class="text-center p-5 d-flex flex-column">
                <input class="w-75 mx-auto fs-4 m-2" type="text" placeholder="Név" v-model="myMember.name">
                <input class="w-75 mx-auto fs-4 m-2" type="text" placeholder="Nem" v-model="myMember.gender">
                <input class="w-75 mx-auto fs-4 m-2 mb-4" type="datetime-local" placeholder="Születési év" v-model="myMember.birth_date">
                <button class="btn btn-success w-25 mx-auto fs-4" @click="addMember">Tagfelvétel</button>
            </div>
        </div>
        <footer class="container p-2">
            <p class="fs-5">Készítette: Kilián Marcell</p>
        </footer>
    </div>
</template>

<script>
import axios from 'axios'

export default {
    name: 'Member',
    
    data() {
        return {
            members: [],
            myMember: {
                name: "",
                gender: "",
                birth_date: null
            }
        }
    },

    methods: {
        async loadMembers() {
            await axios
                .get('http://127.0.0.1:8000/api/members')
                .then(response => this.members = response.data)
                .catch(error => console.log(error))
        },

        async addMember() {
            await axios
                .post('http://127.0.0.1:8000/api/members', this.myMember)
                .catch(error => console.log(error))
            
            this.resetForm()
            this.loadMembers()
        },

        async pay(id) {
            await axios
                .post('http://127.0.0.1:8000/api/members/' + id + '/pay')
                .catch(error => console.log(error))
        },

        resetForm() {
            this.myMember = {
                name: "",
                gender: "",
                birth_date: null
            }
        }
    },

    mounted() {
        this.loadMembers()
    }
}
</script>