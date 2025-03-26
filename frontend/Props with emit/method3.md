# props methode 3:
## 🧱 1. ChartIpAddress.vue (the Provider)
```ts
<script setup lang="ts">
import { ref, provide, onMounted } from 'vue'
import axios from 'axios'

const fileData = ref([])

const fetchIpCounts = async () => {
  try {
    const response = await axios.get(import.meta.env.VITE_API_URL + '/counter/ip-counts')
    fileData.value = response.data
    provide('ipLength', fileData.value.length) // ✅ provide number
  } catch (error) {
    console.error('Error fetching IPs:', error)
  }
}

onMounted(fetchIpCounts)
</script>
```

## 📊 2. WidgetFive.vue (the Injector)
```ts
<script setup lang="ts">
import { inject, ref } from 'vue'

const ipLength = inject<number>('ipLength', 0) // default = 0
</script>

<template>
  <div>Total IPs: {{ ipLength }}</div>
</template>
```

## 🧩 3. DefaultDash.vue (the Parent)
### No props needed anymore 👇
```ts
<script setup lang="ts">
import ChartIpAddress from './ChartIpAddress.vue'
import WidgetFive from './WidgetFive.vue'
</script>

<template>
  <v-col cols="4" md="4" lg="4" no-gutters>
    <ChartIpAddress />
  </v-col>

  <v-col cols="12" md="12" lg="12">
    <WidgetFive />
  </v-col>
</template>
```