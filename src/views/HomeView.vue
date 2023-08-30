<template>
  <v-container >
    <v-card max-width="768" class="mx-auto" elevation="1" :color="bg_color" dark>
      <v-card-text>
        <v-row no-gutters>
          <v-col cols="2" align-self="center">
            <v-img :src="i_img"></v-img>
          </v-col>
          <v-col cols="8">
            <v-form ref="tnform">
              <h3>{{ ticket.message }}</h3>
              <v-text-field
                placeholder="Ticket Type"
                v-model="ticket.ticket_type"
                disabled
                dense
                hide-details
                outlined
                class="pa-0"
                prepend-inner-icon="mdi-account"
              ></v-text-field>
              <v-text-field
                placeholder="Phone No."
                v-model="ticket.phone_number"
                disabled
                dense
                hide-details
                outlined
                class="pa-0"
                prepend-inner-icon="mdi-phone"
              ></v-text-field>
              <v-text-field
              v-model="ticket_number"
              :rules="[(v) => !!v || 'This field is required']"
              outlined
              dense
              placeholder="Enter ticket number"
              prepend-inner-icon="mdi-numeric"
              ></v-text-field>
            </v-form>
          </v-col>
          <v-col cols="2" align-self="center" class="mb-7 ">
            <v-btn @click="validateFo" color="blue" fab>
              <v-icon>mdi-scanner</v-icon>
            </v-btn>
          </v-col>
        </v-row>
          
        <qrcode-stream @decode="onDecode"></qrcode-stream>
      </v-card-text>
    </v-card>
    <v-dialog v-model="processing" max-width="200" persistent>
      <v-card>
        <v-card-text class="text-center py-10">
          <v-progress-circular indeterminate color="blue" size="100" class="mx-auto">
              Loading...
          </v-progress-circular>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import { QrcodeStream } from 'vue-qrcode-reader'
// import axios from 'axios'

  export default {
    name: 'HomeViwe',

    components: {
      QrcodeStream
    },
    data: () => ({
      ticket_number: '',
      ticket:{
        ticket_type: '',
        phone_number:'',
        message: 'Scan'
      },
      processing: false,

      // sound file path
      success_beep : "./sounds/Barcode-scanner-beep-sound.mp3",
      expired_beep : "./sounds/Beep-beep-sound-effect.mp3",
      error_beep : "./sounds/Error-sound.mp3",

      // icon file path
      success_img : "../img/success.png",
      expired_img : "../img/expired.png",
      error_img : "../img/invalid.png",
      next_img : "../img/next.png",

      // colors
      colors:['red', 'warning', 'success', 'grey'],
      bg_color: 'grey',
      i_img: "../img/favicon.png",
    }),

    methods : {
        onDecode(ticket){
            this.processing = true
            this.ticket_number = ticket
            this.axios(
                {
                  method:'get',
                  url:'https://buses.pridezm.com/api/'+ticket+'/scan',
                  headers: { 'Cache-Control': 'no-cache' },
                }
              ).then((res) => {
              if (res.request.readyState == 4 && res.request.status == 200) {
                this.ui(res.data);
              }else if(res.request.readyState == 4 && res.request.status == 404){
                this.ui('{"status":"failed"}');
              }else {
                this.ui('{"status":"failed"}');
              }

            }).catch( (err => {
              alert(err)
              this.processing =false
            })
            )
        },
        validateFo(){
            if (this.$refs.tnform.validate()) {
              this.onDecode(this.ticket_number)
            }
        },
        ui(ticket){
          if(ticket.status == 'success'){
            this.beepControl(this.success_beep)
            this.bg_color = 'success'
            this.ticket = ticket
            this.i_img = this.success_img
          }else if(ticket.status == 'already scanned'){
            this.beepControl(this.expired_beep)
            this.bg_color = 'warning'
            this.ticket = ticket
            this.i_img = this.expired_img
          }else{
            this.beepControl(this.error_beep)
            this.bg_color = 'red'
            this.ticket = {}
            this.i_img = this.error_img
          }

          this.processing = false //hide loading indicator
        },

        resetUi(){
          this.bg_color = 'grey'
          this.i_img = "../img/favicon.png"
        },
        beepControl(sound) {
            const beep = new Audio(sound);
            beep.play();
        }
    }
  }
</script>
