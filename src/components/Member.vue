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
                        <button class="btn btn-primary fs-4 mb-3" @click="pay(m.id)">Tagdíj befizetés</button>
                        <div v-if="m.message != null && m.errorMsg == null" class="alert alert-success">{{ m.message }}</div>
                        <div v-if="m.errorMsg != null" class="alert alert-danger">{{ m.errorMsg }}</div>
                    </div>
                </div>
            </div>
            <div id="createMember" class="text-center p-5 d-flex flex-column">
                <input class="w-75 mx-auto fs-4 m-2" type="text" placeholder="Név" v-model="this.state.myMember.name">
                <span class="text-danger text-center" v-if="v$.myMember.name.$error">
                    {{ v$.myMember.name.$errors[0].$message }}
                </span>
                <select class="w-75 mx-auto fs-4 m-2" v-model="this.state.myMember.gender">
                    <option value="M">Férfi</option>
                    <option value="F">Nő</option>
                    <option value="">Egyéb</option>
                </select>
                <input class="w-75 mx-auto fs-4 m-2 mb-4" type="datetime-local" placeholder="Születési év" v-model="this.state.myMember.birth_date">
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
import useVuelidate from '@vuelidate/core'
import { required, helpers } from '@vuelidate/validators'
import { reactive, computed } from 'vue'

export default {
    name: 'Member',
    
    data() {
        return {
            members: []
        }
    },

    setup() {
        const state = reactive({
            myMember: {
                name: "",
                gender: "",
                birth_date: null
            }
        })

        const rules = computed(() => {
            return {
                myMember: {
                    name: {
                        required: helpers.withMessage('A név mező kitöltése kötelező!', required)
                    }
                }
            }
        })

        const v$ = useVuelidate(rules, state)

        return {
            state,
            v$
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
            this.v$.$validate()
            if (!this.v$.$error) {
                await axios
                    .post('http://127.0.0.1:8000/api/members', this.state.myMember)
                    .catch(error => console.log(error))
                    
                this.resetForm()
                this.loadMembers()
            }
        },

        async pay(id) {
            await axios
                .post('http://127.0.0.1:8000/api/members/' + id + '/pay')
                .then(response => {
                    let chosenMember = this.members.find(m => m.id === id)
                    chosenMember.message = "Sikeres befizetés!"
                })
                .catch(error => {
                    if (error.response.status >= 300) {
                        let chosenMember = this.members.find(m => m.id === id)
                        chosenMember.errorMsg = error.response.data.message
                    }
                })
        },

        resetForm() {
            this.state.myMember = {
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