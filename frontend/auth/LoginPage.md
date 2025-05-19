# the code complete:
```ts
<script lang="ts" setup>
import "../../assets/auth.css"
import "../../assets/abuse.png"
import "../../layouts/dashboard/logo/logo.svg"
import { ref } from 'vue';
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/userStore';
import type { User } from '@/stores/userStore';
import { mdiEyeOff, mdiEye } from '@mdi/js';

const router = useRouter()

let loading = false;
const snackbar = ref<boolean>(false);
const email = ref('');
const visible = ref(false);
const password = ref('');
const errorMsg = ref('errorMessage');
const userStore = useUserStore();

const rememberMe = ref(false) 
    
const handleLogin = async () => {
    loading = true;
    let user: User = { email: email.value, password: password.value }
    let res = await userStore.logIn(user, rememberMe.value)


    if (res?.status === true) {
        // Redirect to dashboard or perform other actions after successful login
        loading = false;
        router.push({ path: "/" })
    }else{
        if(res?.apiStatus == 401){
            errorMsg.value = "incorrectLogin";
        }if(res?.apiStatus == 403){
            errorMsg.value = "blockedLogin";
        } 
    snackbar.value = true;
    loading = false;
    }
};
</script>

<template>
  <v-container fluid class="d-flex align-center justify-center min-h-screen login-bg">
    <v-sheet class="auth-wrapper elevation-10">
      <div class="text-center mb-6">
        <img height="80" src="../../assets/abuse.png" alt="logo" /><strong>X</strong> &nbsp;
        <img height="80" src="../../layouts/dashboard/logo/logo.svg" alt="logo" />
      </div>

      <v-card-title class="text-h5 text-center mb-1">Bienvenue</v-card-title>
      <v-card-text class="text-subtitle-1 text-center mb-4">
        Veuillez vous connecter pour accéder à votre compte
      </v-card-text>

      <v-form validate-on="submit lazy" @submit.prevent="handleLogin">
        <v-text-field
          v-model="email"
          label="Email"
          type="email"
          density="comfortable"
          class="mb-3"
        />
        <v-text-field
          v-model="password"
          label="Mot de passe"
          :append-inner-icon="visible ? mdiEyeOff : mdiEye"
          :type="visible ? 'text' : 'password'"
          @click:append-inner="visible = !visible"
          density="comfortable"
          class="mb-3"
        />
        <v-checkbox
          v-model="rememberMe"
          label="Se souvenir de moi"
          hide-details
          class="mb-4"
        />

        <v-btn
          :loading="loading"
          type="submit"
          color="primary"
          class="mb-2"
          block
        >
          Se connecter
        </v-btn>

        <v-snackbar v-model="snackbar" :timeout="3000" color="red" class="mb-2">
          {{ errorMsg }}
          <template #actions>
            <v-btn variant="text" @click="snackbar = false">Fermer</v-btn>
          </template>
        </v-snackbar>
      </v-form>
    </v-sheet>

    <v-footer class="auth-footer" app>
      <v-col class="text-center">
        © Orange {{ new Date().getFullYear() }} - 1.1.0
      </v-col>
    </v-footer>
  </v-container>
</template>
```