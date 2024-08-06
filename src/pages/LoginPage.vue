<template>
  <q-layout>
    <q-page-container>
      <q-page class="flex flex-center">
        <q-card>
          <q-card-section class="image row justify-center items-center">
            <q-img src="logo.svg" />
          </q-card-section>

          <q-card-section>
            <q-form @submit.prevent="login" class="column items-center">
              <q-input rounded standout v-model="username" type="text" color="white" label="Username" />
              <div class="validation">
                <small v-if="validationError.username">{{ validationError.username[0] }}</small>
              </div>
              <q-input rounded standout v-model="password" type="password" color="white" label="Password" />
              <div class="validation">
                <small v-if="validationError.password">{{ validationError.password[0] }}</small>
              </div>
              <q-btn unelevated rounded color="primary" type="submit" label="LOGIN" />
            </q-form>
          </q-card-section>
        </q-card>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script>
import { Notify } from "quasar";
import { api } from "src/boot/axios";
import { defineComponent } from "vue";

export default defineComponent({
  name: "LoginPage",

  data() {
    return {
      username: "",
      password: "",
      validationError: ''
    };
  },

  methods: {
    login() {
      api
        .post("http://127.0.0.1:8000/api/login", {
          username: this.username,
          password: this.password
        })
        .then((response) => {
          Notify.create({
            type: 'positive',
            message: 'Login berhasil!',
            position: 'top-right',
            timeout: 2500
          })
          this.$router.push('/dashboard');
          console.log(response.data.message);
        })
        .catch((error) => {
          Notify.create({
            type: 'negative',
            message: error.response.data.message,
            position: 'top-right',
            timeout: 2500
          })
          this.validationError = error.response.data.error || '';
          console.log(error);
        });
    },
  },

});
</script>

<style scoped>
.q-layout {
  background: #0561ce;
}

.q-card {
  width: 600px;
  padding: 16px;
  border-radius: 16px;
}

.image {
  height: 100px;
}

.q-img {
  object-fit: cover;
  width: 250px;
  height: 100px;
}

.q-input {
  width: 100%;
}

.validation {
  width: 100%;
  height: 24px;
  margin-top: 4px;
  margin-bottom: 16px;
  padding-left: 8px;
}

small {
  text-align: left;
  color: red;
  font-weight: 500;
}

.q-btn {
  width: 350px;
  height: 56px;
  margin-top: 16px;
}
</style>
