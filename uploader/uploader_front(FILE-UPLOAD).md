# Create file Uploader using ``<v-file-upload/>`` (vuetify)

### MailUploader.vue (Script):

```js
<script setup lang="ts">
import { ref, watch } from 'vue'
import axios from 'axios'
import { mdiDelete, mdiCloudUpload } from '@mdi/js';

const rawFiles = ref<File[]>([])
const files = ref<File[]>([])     

const showSuccessAlert = ref(false)
const showErrorAlert = ref(false)
const errorMessage = ref('')
const isUploading = ref(false)

const allowedExtensions = ['txt', 'html']

watch(rawFiles, (newFiles) => {
  if (!newFiles) {
    files.value = []
    return
  }

  const valid = newFiles.filter(file => {
    const ext = file.name.split('.').pop()?.toLowerCase()
    return allowedExtensions.includes(ext ?? '')
  })

  files.value = valid
})

const uploadFile = async () => {
  const formData = new FormData()
  files.value.forEach(file => {
    formData.append('files', file)
  })

  isUploading.value = true
  errorMessage.value = ''
  showSuccessAlert.value = false
  showErrorAlert.value = false

  try {
    const res = await axios.post(
      import.meta.env.VITE_API_URL + '/uploader/upload',
      formData,
      {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }
    )

    if (res.status === 200) {
      showSuccessAlert.value = true
      rawFiles.value = []
      files.value = []
    }
  } catch (error) {
    errorMessage.value =
      error instanceof Error ? error.message : 'Upload failed. Please try again.'
    showErrorAlert.value = true
  } finally {
    isUploading.value = false
  }
}

const removeFile = (index: number) => {
  files.value.splice(index, 1)
  rawFiles.value.splice(index, 1)
}
</script>
```

### 🔹 rawFiles – User's full selection (All the files the user picked — valid or not.)
```ts
rawFiles.value = [
  File { name: 'notes.txt', type: 'text/plain' },
  File { name: 'image.png', type: 'image/png' },
  File { name: 'page.html', type: 'text/html' }
]
```
User picked 3 files — including one invalid (.png).

---

### 🔹 files – Only accepted files (A cleaned-up list from rawFiles — only .txt and .html.)
```ts
files.value = [
  File { name: 'notes.txt', type: 'text/plain' },
  File { name: 'page.html', type: 'text/html' }
]
```
After validation, only .txt and .html are kept.

---

### 🔹 formData – What gets uploaded (A special object that holds the validated files (files) in the format needed for sending to your backend via axios.)
```ts
const formData = new FormData()
files.value.forEach(file => formData.append('files', file))
```
Now formData contains:

```makefile
files: notes.txt  
files: page.html
```
When submitted with Axios, this is what the server receives.

---

✅ That’s the data flow: rawFiles ➜ filtered ➜ files ➜ wrapped ➜ formData ➜ uploaded

### Template:

```ts
<template>
  <v-container>
    <v-row>
      <v-col>
        <v-btn to="/" color="primary" class="mt-4">
          Back to homePage
        </v-btn>
      </v-col>
    </v-row>
      
    <v-row>
      <v-col cols="12" lg="12" md="12">
        <v-file-upload
          v-model="rawFiles"
          :show-list="false"
          multiple
          accept=".txt,.html"
          color="#edcaaf"
          >
          <template v-slot:icon="">
            <v-icon>{{ mdiCloudUpload  }}</v-icon>
          </template>
          <template #item /> <-- disable the list of selected items -->
        </v-file-upload>
      </v-col>
    </v-row>

    <v-row>
      <v-col>
        <v-btn
          color="#F16E00"
          class="mt-4"
          :loading="isUploading"
          :disabled="!rawFiles.length"
          @click="uploadFile"
        >
          Upload
        </v-btn>
      </v-col>
    </v-row>

    <v-alert
      v-if="showSuccessAlert"
      type="success"
      class="mt-4"
      closable
      @click:close="showSuccessAlert = false"
    >
      File uploaded successfully!
    </v-alert>

    <v-alert
      v-if="showErrorAlert"
      type="error"
      class="mt-4"
      closable
      @click:close="showErrorAlert = false"
    >
      {{ errorMessage }}
    </v-alert>

    <v-row>
      <v-col cols="12">
        <div v-if="files.length">
          <h3 class="text-h6 mb-2">Files ready to upload:</h3>
          <v-list density="compact" class="bg-grey-lighten-4 rounded" max-height="400" width="700">
            <v-list-item
              v-for="(file, index) in files"
              :key="index"
            >
              <v-list-item-title>
                {{ file.name }} — {{ (file.size / 1024).toFixed(1) }} KB ({{ file.type || 'unknown' }})
              </v-list-item-title>

              <template #append>
                <v-btn
                  size="small"
                  variant="text"
                  @click="removeFile(index)"
                ><v-icon color="red" :size="25">{{ mdiDelete }}</v-icon></v-btn>
              </template>
            </v-list-item>
          </v-list>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>
```
