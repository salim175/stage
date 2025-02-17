# Create file Uploader using Vue.js (vuetify)

### FileUploader.vue (Template):
```js
<template>
  <div>
    <v-row>
      <v-col cols="12">
        <v-file-input
          v-model="files"
          :show-size="1000"
          color="orange-accent-4"
          label="File input"
          placeholder="Select your files"
          prepend-icon="mdi-paperclip"
          variant="outlined"
          counter
          multiple
          :error="!!errorMessage"
          :error-messages="errorMessage"
          @on-update:model-value="handleFileUpload"
        >
          <template #selection="{ fileNames }">
            <template v-for="(fileName, index) in fileNames" :key="fileName">
              <v-chip
                v-if="index < 2"
                class="me-2"
                color="orange-accent-4"
                size="small"
                label
              >
                {{ fileName }}
              </v-chip>
              
              <span
                v-else-if="index === 2"
                class="text-overline text-grey-darken-3 mx-2"
              >
                +{{ files.length - 2 }} File(s)
              </span>
            </template>
          </template>
        </v-file-input>
      </v-col>
      
      <v-col cols="12">
        <v-btn 
          color="primary" 
          class="mt-4" 
          :loading="isUploading"
          :disabled="!files.length"
          @click="uploadFile"
        >
          Upload
        </v-btn>
      </v-col>
    </v-row>
    
    <v-alert
      v-if="uploadSuccess"
      type="success"
      class="mt-4"
      closable
    >
      File uploaded successfully!
    </v-alert>
    
    <v-alert
      v-if="errorMessage"
      type="error"
      class="mt-4"
      closable
    >
      {{ errorMessage }}
    </v-alert>
  </div>
</template>

```

### 📌 v-file-input:
- ```<v-file-input>``` is a Vuetify file input component.
- ```v-model="files"``` binds the selected files to the ```files``` state.
- ```:show-size="1000"``` shows file sizes in KB (divides by ```1000```).
- ```color="orange-accent-4"``` styles the input field.
- ``label="File input"`` and ``placeholder="Select your files"`` provide UI text.
- ``prepend-icon="mdi-paperclip"`` adds a paperclip icon.
- ``variant="outlined"`` gives an outlined style.
- ``counter`` displays the number of selected files.
- ``multiple`` allows multiple file selection.
- ``:error="!!errorMessage"`` enables error styling if ``errorMessage`` is not empty.
  - <u>ex:</u> errorMessage='value' (she have a value, but we need to return a boolean) → So !errorMessage=false → !!errorMessage = true
    - When **errorMessage** is empty, ``:error="false"`` → No error styling.
    - When **errorMessage** has a value, ``:error="true"`` → Error styling is applied.
- ``:error-messages="errorMessage"`` displays error messages.
- ``@on-update:model-value="handleFileUpload"`` calls ``handleFileUpload()`` when a file is selected.

``selection slot:`` fileNames is automatically provided by Vuetify in the #selection slot. [more info](https://vuetifyjs.com/en/api/v-file-input/#:~:text=to%20the%20input.-,selection,-%7B%0A%20%20fileNames%3A)

---
### Script:

```js

<script setup lang="ts">
import axios from 'axios';
import { ref } from 'vue';

const files = ref<File[]>([]);
const uploadSuccess = ref(false);
const errorMessage = ref('');
const isUploading = ref(false);

const handleFileUpload = (uploadedFiles: File[]) => {
  if (!uploadedFiles) {
    files.value = [];
    return;
  }
  
  files.value = uploadedFiles;
  errorMessage.value = '';
  uploadSuccess.value = false;
};

const uploadFile = async () => {
  const formData = new FormData();
  files.value.forEach((file) => {
    formData.append('files', file); // Changed to use consistent key 'files'
  });

  isUploading.value = true;
  errorMessage.value = '';
  
  try {
    const res = await axios.post('http://localhost:5000/api/uploader/upload', formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    });

    if (res.status === 200) {
      uploadSuccess.value = true;
      files.value = [];
    }
  } catch (error) {
    errorMessage.value = error instanceof Error 
      ? error.message 
      : 'Upload failed. Please try again.';
  } finally {
    isUploading.value = false;
  }
};
</script>
```

``FormData():`` is used to send files and other data in an HTTP request. It allows sending binary data (files) instead of JSON.

``files.value`` is an array of selected files.
``.forEach((file) => { ... })`` loops through each file.

``finally:`` Runs whether the upload succeeds or fails.