<template>
  <div class="register">
  
    <v-layout>
      <v-flex xs12 sm6 offset-sm3>
  
        <v-card class="mt-5" color="rgba(0, 0, 0, 0.2">
  
          <v-container v-bind="{ [`grid-list-${0}`]: true }" fluid>
            <v-layout justify-center>
              <v-flex v-for="n in 1" :key="n" xs4 text-xs-center>
  
                <!-- card for user registration info -->
                <v-card v-if="!nextInfo" color="rgba(0, 0, 0, 0.0" flat tile>
  
                  <v-text-field dark v-model="email" label="E-mail" required></v-text-field>
                  <v-text-field dark v-model="password" label="Password" :type="'password'" required></v-text-field>
                  <v-text-field dark v-model="name" label="Full name" required></v-text-field>
                  <v-text-field dark v-model="number" label="Phone number" required></v-text-field>
  
                  <!-- scroll list for location -->
                  <v-menu offset-x>
                    <template v-slot:activator="{ on }">
                        <v-btn dark outline v-on="on" > {{ chosenCounty }} <v-icon color="red" right="" >location_on</v-icon> </v-btn>
                  </template>
                  <v-list style="max-height: 300px" class="scroll-y">
                    <v-list-tile v-for="(county, index) in countyArray" :key="index" @click="chooseCounty(index)">
                      <v-list-tile-title>{{ county }}</v-list-tile-title>
                    </v-list-tile>
                  </v-list>
                  </v-menu>

                  <v-btn dark outline @click="next(email, password, name, number)">Next<v-icon color="green" right="" >arrow_forward</v-icon></v-btn>
              
              </v-card>
             
              <!-- Pet information form -->
              <v-card v-if="nextInfo" color="rgba(0, 0, 0, 0.0" flat tile>

                <v-text-field dark v-model="petType" label="Pet type" required></v-text-field>
                <v-text-field dark v-model="petName" label="Pet name" required></v-text-field>
                <v-text-field dark v-model="petAge" label="Pet age" required></v-text-field>
                
                <v-textarea dark v-model="petDescription" label="Describe your pet">Description</v-textarea>

                <v-switch dark v-model="pedigree" label="Pedigree"></v-switch>
                
                <v-switch dark @click="petGender(petMale)" v-model="petMale" label="Male"></v-switch> 
                <v-switch dark @click="petGender(!petMale)" v-model="petFemale" label="Female"></v-switch>

                <form method="post" enctype="multipart/form-data">
                  <input v-on:change="saveImage()" type="file" name="uploadedImage" ref="uploadedImage"/>
                  <!-- <v-btn v-on:change="saveImage()" type="file" name="uploadedImage" ref="uploadedImage">UPLOAD</v-btn> -->
                </form>
                
                <v-btn dark outline @click="back()"> Back <v-icon color="red" right="" >arrow_back</v-icon></v-btn>
                <v-btn dark outline @click="register(petType, petName, petAge, petDescription, pedigree)"> Register <v-icon color="green" right="" >check_circle</v-icon></v-btn>

              </v-card>

              <!-- alert for user to fill out all fields -->
              <v-card dark v-if="emptyFields" flat tile color="rgba(0, 0, 0, 0.0"> 
                 <v-divider></v-divider>
                  <div> Please fill out all the fields. </div>
              </v-card>

            </v-flex>
          </v-layout>
        </v-container>
      </v-card>
    </v-flex>
  </v-layout>

  </div>
</template>


<script>
  import axios from 'axios'
  export default {
    name: 'register',
    data() {
      return {
        countyArray: 
        ['Blekinge',
          'Dalarna',
          'Gotland',
          'Gävleborg',
          'Halland',
          'Jämtland',
          'Jönköping',
          'Kalmar',
          'Kronoberg',
          'Norrbotten',
          'Skåne',
          'Stockholm',
          'Södermanland',
          'Uppsala',
          'Värmland',
          'Västerbotten',
          'Västernorrland',
          'Västmanland',
          'Västra Götaland',
          'Örebro',
          'Östergötland'
        ],
        emptyFields: false,
        email: null,
        name: null,
        password: null,
        chosenCounty: 'County',
        number: null,
        userInfo: null,
        nextInfo: false,
        petName: null,
        petType: null,
        petAge: null,
        petDescription: null,
        pedigree: null,
        gender: null,
        petMale: null,
        petFemale: null,
        petInfo: null,
        petId: null,
        formData: null,
        uploadedFile: ''
      }
    },
    methods: {
      back() {
        this.nextInfo = false
      },
      
      chooseCounty(index) {
        this.chosenCounty = this.countyArray[index]
      },

      // Checker for if all fields have been filled
      fieldChecker(fields) {
        for (var field in fields) {
          if (fields[field] !== null && fields[field] != 'County' ) {
            this.emptyFields = false
          } else if (fields[field] === null || fields[field] == 'County') {
            this.emptyFields = true
          }
        }
        if(!this.emptyFields) {
          this.nextInfo = true
        }
      },

      // Get user information and save to user object
      next(email, password, name, number) {
        this.userInfo = {
          name: name,
          password: password,
          email: email,
          location: this.chosenCounty,
          number: number
        }
        this.fieldChecker(this.userInfo)
      },

      //true for female, false for male
      petGender(chosenGender) {
        if (chosenGender) {
          this.petMale = true
          this.petFemale = false
        } else if (!chosenGender) {
            this.petMale = false
            this.petFemale = true
          }
      },

      register(type, name, age, description, pedigree) {
        this.petInfo = {
          name: name,
          userId: null,
          type: type,
          description: description,
          gender: this.petMale,
          pedigree: pedigree,
          age: age,
          imageName: ''
        }

        this.fieldChecker(this.petInfo)

        if(!this.emptyFields) {
          this.saveUser()
          this.$store.state.userLoggedIn = this.name
          this.$router.push('/Search')
        }
      },

      saveImage() {
        this.formData = new FormData()

        this.uploadedFile = this.$refs.uploadedImage[0].files[0]
        this.formData.append('image', this.uploadedFile)

      },

      saveUser() {
        fetch('/api/users', {
            body: JSON.stringify(this.userInfo),
            headers: {
              'Content-Type': 'application/json'
            },
            method: 'POST'
          })
          .then(response => response.text())
          .then(result => {
            this.petInfo.userId = result
          })
          .catch(error => {
            console.log(errror)
          })
          .then(() => {
            axios.post('/api/file', this.formData, {
            headers: {
                'Content-Type': 'multipart/form-data'
              }
            })
            .then(response => response)
            .then(result => {
              this.petInfo.imageName = result.data.filename 
            })
            .catch(error => {
              console.log(error);
            })
            .then(() => {
              fetch('/api/pets', {
                body: JSON.stringify(this.petInfo),
                headers: {
                  'Content-Type': 'application/json'
                },
                method: 'POST'
              })
              .then(response => response.text())
              .then(result => {
                console.log(result)
              })
              .catch(error => {
                console.log(error)
              })
            })
          }) 
      }
    }
  }
</script>

<style scoped>
  .register {
    background-image: url('../assets/Dog_Background.jpg');
    background-size: cover;
    height: 100%;
    width: 100%;
    align-items: top;
  }
</style>
