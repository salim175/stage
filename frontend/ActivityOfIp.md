# Create a structure to detect the IP activity using store(pieStore):
### Script:

```js
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { mdiMagnify } from '@mdi/js';
import { usePieStore } from '@/stores/chartPieStore';
import type { From } from '@/stores/chartPieStore';

const store = usePieStore()
const search = ref<string>('');
const selectedCompany = ref<From | null>(null);
const selectedAttackerIP = ref<string | null>(null);

console.log('CHART DATA:',store.chartData)
console.log('FROM DATA:',store.fromData)

// define the headers for the table
interface TableHeader {
  title: string,
  key: keyof From,
  align?: "start" | "center" | "end",
  sortable?: boolean
}

const headers = computed<TableHeader[]>(() => [
  { title: "Company", key: "_id", align: "start", sortable: false },
  { title: `Number of report`, key: "count", align: "center", sortable: true }
])

// 1
function handleRowClick(row: any) {
  const company = row?.item // the company is in the item field in the selected row
  console.log('row:', row)
  console.log('selected row:', company)
  console.log('attackers:', company?.attackers)
  selectedCompany.value = company
}

// console.log('FROM DATA:', store.fromData);
// 2
const attackersList = computed(() => selectedCompany.value?.attackers ?? [])

// 3
const attackerActivity = computed(()=>{
  if(!selectedAttackerIP.value) return [];

  return store.fromData.filter(company =>
    company.attackers.some(attacker => attacker.ip === selectedAttackerIP.value)
  )
})

onMounted(() => {
  store.fetchFromCounts();
})
</script>
```

## the data of the selected company is in **item**:
the value of selected company will becomes -> **selected row**

![alt text](/images/image4.png)

# 📌 Complete Code:
```js
<template>
  <v-container>
    <v-row>
      <!-- Victim Company Table -->
      <v-col cols="6">
        <v-card title="Victim Companies">
            <template v-slot:text>
              <v-text-field
              v-model="search"
              label="Search"
              :prepend-inner-icon="mdiMagnify" 
              hide-details
              single-line
              placeholder="Search for an IP..."
              persistent-placeholder
              ></v-text-field>
            </template>
            <v-data-table
              :headers="headers"
              :items="store.fromData"
              :search="search"
              :height=600 
              dense
              @click:row="(_: any, row: any) => handleRowClick(row)"
            />
        </v-card>
      </v-col>

      <!-- Attcker List -->
      <v-col cols="6">
        <v-card :title="selectedCompany ? `IP attaquant - ${selectedCompany._id} - ${selectedCompany.attackers.length} IPs` : 'IP attaquant'" >
          
          <template v-slot:text>
            <v-text-field
            v-model="search"
            label="Search"
            :prepend-inner-icon="mdiMagnify" 
            hide-details
            single-line
            placeholder="Search for an IP..."
            persistent-placeholder
            ></v-text-field>
          </template>
          <!-- <v-list height="662" v-if="selectedCompany && selectedCompany.attackers?.length"> -->
          <v-list height="662" v-if="attackersList && attackersList.length > 0">
            <v-list-item
              v-for="(attacker, index) in selectedCompany?.attackers"
              :key="index"
              class="cursor-pointer"
              @click="selectedAttackerIP = attacker.ip"
            >
              <v-list-item-title>
                {{ attacker.ip }} — {{ attacker.count }} time<span v-if="attacker.count > 1">s</span>
              </v-list-item-title>
            </v-list-item>
          </v-list>

          <v-card-text v-else>
            {{ selectedCompany ? 'No attackers found for this company.' : 'Select a company to view attacker IPs' }}
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- Attacker History -->
    <v-row v-if="selectedAttackerIP">
      <v-col cols="12">
        <v-card>
          <v-card-title>
            Activity of IP: {{ selectedAttackerIP }}
          </v-card-title>
          <v-card-text>
            <v-list>
              <v-list-item
                v-for="(company, i) in attackerActivity"
                :key="i"
              >
                <v-list-item-title>
                  <!-- {{ company._id }} — {{ company.count }} reports -->
                  {{ company._id }} 
                </v-list-item-title>
              </v-list-item>
            </v-list>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>
```