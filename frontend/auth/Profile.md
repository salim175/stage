# the code complete:
```ts
<script setup lang="ts">
import { ref, watch } from 'vue'
import axios from 'axios'
import { useRoute, useRouter } from 'vue-router'
import { useUserStore } from '@/stores/userStore'
import { mdiPencil } from '@mdi/js';

const route = useRoute()
const router = useRouter()

const userStore = useUserStore()
const form = ref({ ...userStore.user })

const saving = ref(false)
const editing = ref(false)
const success = ref(false)
const errorMsg = ref('')

const mode = ref<'view' | 'edit'>(route.query.mode === 'edit' ? 'edit' : 'view')

const updateProfile = async () => {
  saving.value = true
  errorMsg.value = ''
  success.value = false

  try {
    const id = userStore.user?.id
    const token = localStorage.getItem('Auth')

    const res = await axios.put(import.meta.env.VITE_API_URL + '/user/-1', form.value, {
      headers: {
        Authorization: `Bearer ${token}`
      }
    })

    if (res.status === 200) {
      userStore.user = res.data
      editing.value = false
      form.value.oldPassword = ''
      form.value.password= ''
      success.value = true
    }
  } catch (err: any) {
    errorMsg.value = err.response?.data?.message || 'Update failed'
  } finally {
    saving.value = false
  }
}

const formatDateHuman = (timestamp: string | number | undefined) => {
    if (!timestamp) return 'N/A';
    return new Intl.DateTimeFormat("fr-FR", {
        day: "2-digit",
        month: "short",
        year: "numeric",
        hour: "2-digit",
        minute: "2-digit",
        second: "2-digit"
    }).format(new Date(Number(timestamp) * 1000));
};

watch(() => route.query.mode, (val) => {
  mode.value = val === 'edit' ? 'edit' : 'view'
})

</script>

<template>
  <v-container>
    <v-card elevation="2" class="pa-4">
      <v-card-title class="d-flex justify-space-between align-center">
        <span class="text-h5">My Profile</span>
        <v-btn 
            v-if="mode === 'view'" 
            @click="router.push({path:'/profile', query:{mode:'edit'}})"
            color="primary"
            :prepend-icon="mdiPencil"
            >
            Edit
        </v-btn>
      </v-card-title>

      <v-divider></v-divider>

      <v-card-text>
        <v-form v-if="mode === 'edit'" @submit.prevent="updateProfile">
          <v-row dense>
            <v-col cols="12" md="6">
              <v-text-field v-model="form.firstname" label="First Name" density="comfortable" />
            </v-col>
            <v-col cols="12" md="6">
              <v-text-field v-model="form.lastname" label="Last Name" density="comfortable"/>
            </v-col>
            <v-col cols="12" md="6">
              <v-text-field v-model="form.email" label="Email" density="comfortable"/>
            </v-col>
            <v-col cols="12" md="6">
              <v-text-field v-model="form.pseudo" label="Pseudo" density="comfortable"/>
            </v-col>
            <v-col cols="12" md="6">
              <v-text-field v-model="form.oldPassword" label="Old Password" type="password" density="comfortable"/>
            </v-col>
            <v-col cols="12" md="6">
              <v-text-field v-model="form.password" label="New Password" type="password" density="comfortable"/>
            </v-col>
          </v-row>

          <v-alert type="error" v-if="errorMsg" class="my-2">{{ errorMsg }}</v-alert>
          <v-alert type="success" v-if="success" class="my-2">Profile updated successfully.</v-alert>

          <v-btn color="primary" type="submit" class="me-2" :loading="saving">Save</v-btn>
          <v-btn color="secondary" variant="outlined" 
            @click="router.push({path:'/profile', query:{ mode: 'view' }})"
            >
            Cancel
        </v-btn>
        </v-form>

        <div v-if="mode === 'view'" class="text-body-1">
          <v-row dense>
            <v-col cols="12" md="6"><strong>First Name:</strong> {{ form.firstname }}</v-col>
            <v-col cols="12" md="6"><strong>Last Name:</strong> {{ form.lastname }}</v-col>
            <v-col cols="12" md="6"><strong>Email:</strong> {{ form.email }}</v-col>
            <v-col cols="12" md="6"><strong>Pseudo:</strong> {{ form.pseudo }}</v-col>
            <v-col cols="12" md="6"><strong>Role:</strong> {{ form.role }}</v-col>
            <v-col cols="12" md="6"><strong>Last Login:</strong> {{ formatDateHuman(form.last_logins?.[8]?.date) || '' }}</v-col>
          </v-row>
        </div>
      </v-card-text>
    </v-card>
  </v-container>
</template>
```