<template>
  <div class="content">
    
    <section class="h-100 gradient-form" style="background-color: #eee;">
      <div class="container py-5 h-100">
        <div class="row d-flex justify-content-center align-items-center h-100">
          <div class="col-xl-10">
            <div class="card rounded-3 text-black">
              <div class="row g-0">
                <div class="col-lg-6" style="background-color:#FFFFFF">
                  <div class="card-body p-md-5 mx-md-4">

                    <div class="text-center">
                      <img src="/img/logo-atrabajar.png"
                        style="width: 185px;" alt="logo">
                      <h4 class="mt-1 mb-5 pb-1">We are The Solucion Afore Team</h4>
                    </div>

                    <form>
                      <p>Ingresa la información de tu cuenta</p>

                      <div :class="'form-outline mb-4' + ($v.user.email.$invalid && submited?' has-error':'') ">
                        <input type="email" id="form2Example11" class="form-control" v-model="user.email" placeholder="Correo Electrónico" />
                        <label :class="'form-label' + ($v.user.email.$invalid && submited?' text-danger':'') " for="form2Example11">Correo Electrónico</label>
                      </div>

                      <div :class="'form-outline mb-4' + (($v.user.cntr.$invalid) && submited?' has-error':'') ">
                        <input type="password" id="form2Example22" class="form-control" v-model="user.cntr" placeholder="Contraseña"/>
                        <label :class="'form-label' + ($v.user.cntr.$invalid && submited?' text-danger':'') " for="form2Example22">Contraseña</label>
                      </div>

                      <p class="alert alert-danger control-label" v-if="$v.user.$invalid && submited">Existen campos pendientes de llenar en el formulario</p>

                      <p class="alert alert-danger control-label" v-if="mensajeUsuario != ''" v-html="mensajeUsuario"></p>

                      <div class="text-center pt-1 mb-5 pb-1">
                        <button @click="login" class="btn btn-primary btn-block fa-lg gradient-custom-2 mb-3" type="button">Ingresar</button>
                        <a class="text-muted" href="#" @click="olvidaste()">Olvidaste tu Contraseña?</a>
                      </div>

                      <div class="d-flex align-items-center justify-content-between pb-4">
                        <p class="mb-0 me-2">Tienes cuenta de acceso?</p>
                        <button type="button" class="btn btn-outline-danger" @click="crear()">Crear</button>
                      </div>

                    </form>

                  </div>
                </div>
                <div class="col-lg-6 d-flex align-items-center gradient-custom-2" style="background-color:#0090CC">
                  <div class="text-white px-3 py-4 p-md-5 mx-md-4" >
                    <h4 class="mb-4">We are more than just a company</h4>
                    <p class="small mb-0">Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
                      tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud
                      exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.</p>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>


  </div>
</template>
<script>
  import Vue from 'vue'
  import axios from 'axios'
  import store from '@/store'
  import { validationMixin } from 'vuelidate'
  import { required, email } from 'vuelidate/lib/validators'

  export default {
    props: {
      backgroundColor: {
        type: String,
        default: 'black',
        validator: (value) => {
          let acceptedValues = ['', 'blue', 'azure', 'green', 'orange', 'red', 'purple', 'black']
          return acceptedValues.indexOf(value) !== -1
        }
      },
    },
    data () {
      return {
        user: {
          email: '',
          cntr: '' 
        },
        submited: false,
        mensajeUsuario: '' 
      }
    },
    mixins: [validationMixin],
    validations() { 
        return { 
          user: this.rules
        } 
    },
    computed: { 
        rules () {
            return {
              email: { required, email },
              cntr: { required }
            }
        }
    },
    methods: {
      login(){
        var self = this
        self.submited = true
        this.mensajeUsuario = '';
        if(self.$v.user.$invalid) return
        //e.preventDefault();
        Vue.showLoader();
        store.dispatch('AUTH_REQUEST', self.user )
            .then(() => {
              Vue.hideLoader();
              console.log('entra al sistema')
              this.$router.push("/")
                /*
                if( store.getters.correoConfirmado == '0' ){
                    this.$router.push('/confirmaCorreo/')
                }else if( store.getters.cntrTmpr == '1' ){
                    this.$router.push('/contraseniaTemporal/')
                }else if( store.getters.roles.length == 0 ){
                    //console.log('sin roles')
                    Vue.alert({ 'title': 'Aviso', 'html': 'El Usuario No cuenta con ningun rol asignado, por favor contacta al administrador del Sistema para que te sea asignado tu correspondiente Rol (Liga,Equipo,Arbitro,etc)',
                        'buttons': [
                            {
                                title: 'Aceptar',
                                handler: () => {  }
                            }
                        ]
                    })
                }else if ( store.getters.idRol != 0 ){
                    this.$router.push('/'); this.$router.go(0)
                    //this.$router.push('/roles/')
                }else{
                    this.$router.push('/roles/')
                }
                */
            }).catch(error => {
              Vue.hideLoader();
              console.log('error');
              console.log(error);
              // error.code == "ERR_NETWORK" Error de conexion
              if( error == 'Usuario invalido'){
                self.errorLogin()
              }else if ( error == 'Usuario inhabilitado'){
                self.errorInhabilitado()
              }else{
                self.errorComunicacion()
              }
              //$.toast({ heading: 'Aviso', text: 'Usuario/ contraseña incorrecto', hideAfter: 5000, position: 'top-center' });
        })
      },
      errorLogin(){
        this.mensajeUsuario = 'Usuario o Contraseña incorrectas';
        Vue.alert({ 'title': 'Aviso', 'html': 'Verifica tus credenciales, las cuales son incorrectas.',
            'buttons': [ { title: 'Aceptar', handler: () => { } } ]
        })
      },
      errorInhabilitado(){
        this.mensajeUsuario = 'Usuario inhabilitado';
        Vue.alert({ 'title': 'Aviso', 'html': 'Es necesario que se comunique con el adminsitrador del sistema para que confirme su acceso al sistema.',
            'buttons': [ { title: 'Aceptar', handler: () => { } } ]
        })
      },
      errorComunicacion(){
        this.mensajeUsuario = 'No se pudo establecer conexión al servidor';
      },
      olvidaste(){
        Vue.showLoader();
        Vue.hideLoader();
        this.$router.push("/recoverPw/")
      },
      crear(){
        Vue.showLoader();
        Vue.hideLoader();
        this.$router.push("/registro/")
      }
    },
  }

</script>
<style>

</style>
