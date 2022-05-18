<template>
     <div>
          <div class="container">
               <div class="row mt-5">
                    <div class="col-xl-4 col-md-6 col-12" v-for="m in this.members" :key="m.id">
                         <div class="card p-2 m-2" >
                              <p class="fs-3"><h1>{{ m.name }}</h1></p>
                              <p class="fs-4">Született: {{ m.birth_date }}</p>
                              <p class="fs-4">Csatlakozott: {{ m.created_at }}</p>
                              <img src="../assets/male.png" v-if="m.gender == 'M'">
                              <img src="../assets/female.png" v-if="m.gender == 'F'">
                              <img src="../assets/other.png" v-if="m.gender == null">
                              <button class="btn btn-success" @click="pay(m.id)">Tagdíj befizetése</button>
                              <span v-if="m.msg != ''" class="fs-5 w-100 mt-2 mx-auto alert-success">{{ m.msg }}</span>
                              <span v-if="m.errorMsg != ''" class="fs-5 w-100 mt-2 mx-auto alert-danger">{{ m.errorMsg }}</span>
                         </div>
                    </div>
               </div>
               <div class="mt-5 mb-5 w-100">
                    <div class="row w-75 mx-auto">
                         <input class="margin-auto fs-4" type="text" placeholder="Név" v-model="this.myMember.name">
                         <span class="fs-5 mt-2 mx-auto alert alert-danger" v-if="this.nameErrorMsg != ''">{{ this.nameErrorMsg }}</span>
                         <select class="mt-2 fs-4" v-model="this.myMember.gender">
                              <option value="M">Férfi</option>
                              <option value="F">Nő</option>
                              <option value="">Egyéb</option>
                         </select>
                         <input class="mt-2 fs-4" type="datetime-local" v-model="this.myMember.birth_date">
                         <span class="fs-5 mt-2 mx-auto alert alert-danger" v-if="this.birthErrorMsg != ''">{{ this.birthErrorMsg }}</span>
                         <button class="fs-4 btn btn-success mt-4" @click="addMember">
                              Tagfelvétel
                         </button>
                    </div>
               </div>
          </div>
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
                    name: '',
                    gender: '',
                    birth_date: null
               },
               nameErrorMsg: "",
               birthErrorMsg: ""
          }
     },

     methods: {
          async loadData() {
               await axios
                    .get('http://127.0.0.1:8000/api/members')
                    .then(response => this.members = response.data.data)
                    .catch(error => console.log(error))
          },

          async addMember() {
               let member = {
                    name: this.myMember.name,
                    birth_date: this.myMember.birth_date
               }

               if (this.myMember.gender != "") {
                    member.gender = this.myMember.gender
               }

               if (this.myMember.name == "") {
                    this.nameErrorMsg = "A név mező kitöltése kötelező!"
               } else {
                    this.nameErrorMsg = ""
               }

               if (this.myMember.birth_date == null) {
                    this.birthErrorMsg = "A születési idő megadása kötelező!"
               } else {
                    this.birthErrorMsg = ""
               }

               if (this.birthErrorMsg == "" && this.nameErrorMsg == "") {
                    await axios
                         .post('http://127.0.0.1:8000/api/members', member)
                         .catch(error => console.log(error))
                    
                    this.loadData()
                    this.resetForm()
               }
          },

          async pay(id) {
               await axios
                    .post('http://127.0.0.1:8000/api/members/' + id + '/pay')
                    .then(response => {
                         let chosenMember = this.members.find(m => m.id === id)
                         chosenMember.msg = "Sikeres befizetés!"
                    })
                    .catch(error => {
                         if (error.response.status >= 300) {
                              let chosenMember = this.members.find(m => m.id === id)
                              chosenMember.msg = ''
                              chosenMember.errorMsg = error.response.data.message
                         }
                    })
          },

          resetForm() {
               this.myMember.name = '',
               this.myMember.gender = '',
               this.myMember.birth_date = null
          }
     },

     mounted() {
          this.loadData()
     }
}
</script>
