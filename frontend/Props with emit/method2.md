# props methode 2:
## 🧩 1. ChartIpAddress.vue (child sender)
```ts
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

const emit = defineEmits(['ip-length'])

const fileData = ref([])

const fetchIpCounts = async () => {
  try {
    const response = await axios.get(import.meta.env.VITE_API_URL + '/counter/ip-counts')
    fileData.value = response.data

    emit('ip-length', fileData.value.length) // ✅ send to parent
  } catch (error) {
    console.error('Error fetching IPs:', error)
  }
}

onMounted(fetchIpCounts)
</script>
```

## 🧩 2. DefaultDash.vue (parent coordinator)
```ts
<script setup lang="ts">
import { ref } from 'vue'
import ChartIpAddress from './ChartIpAddress.vue'
import WidgetFive from './WidgetFive.vue'

const ipLength = ref(0)

const handleIpLength = (length: number) => {
  ipLength.value = length
}
</script>

<template>
  <v-col cols="4" md="4" lg="4" no-gutters>
    <ChartIpAddress @ip-length="handleIpLength" />
  </v-col>

  <v-col cols="12" md="12" lg="12">
    <WidgetFive :ip-length="ipLength" />
  </v-col>
</template>
```

## 🧩 3. WidgetFive.vue (child receiver)
```ts
<script setup lang="ts">
const props = defineProps<{ ipLength: number }>()
</script>

<template>
  <div>Total IPs: {{ ipLength }}</div>
</template>
```

| Syntax | Meaning                             | Usage Example                                |
|--------|-------------------------------------|----------------------------------------------|
| `:`    | Bind a prop from parent → child     | `<WidgetFive :ip-length="ipLength" />`       |
| `@`    | Listen to an event from a child     | `<ChartIpAddress @ip-length="handleFn" />`   |