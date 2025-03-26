# props methode 1:
## 🧱 1. DefaultDash.vue (Parent)
```ts
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'
import ChartIpAddress from './ChartIpAddress.vue'
import WidgetFive from './WidgetFive.vue'

const fileData = ref([])

const fetchIpCounts = async () => {
  try {
    const response = await axios.get(import.meta.env.VITE_API_URL + '/counter/ip-counts')
    fileData.value = response.data
  } catch (error) {
    console.error('Error fetching IPs:', error)
  }
}

onMounted(fetchIpCounts)
</script>

<template>
  <v-col cols="4" md="4" lg="4" no-gutters>
    <ChartIpAddress :file-data="fileData" />
  </v-col>

  <v-col cols="12" md="12" lg="12">
    <WidgetFive :ip-length="fileData.length" />
  </v-col>
</template>
```

## 📈 2. ChartIpAddress.vue
```ts
<script setup lang="ts">
defineProps<{ fileData: IpInfo[] }>()
</script>
```

## 📊 3. WidgetFive.vue
```ts
<script setup lang="ts">
defineProps<{ ipLength: number }>()
</script>

<template>
  <div>Total IPs: {{ ipLength }}</div>
</template>
```