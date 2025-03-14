# Connect to AbuseIpdb using Store Pinia:

## IpdbDetailsPage.vue:
```js
<script setup lang="ts">
import { useRoute } from "vue-router";
import { onMounted, computed, watch } from "vue";
import { useDashboardStore } from "@/stores/dashStores";

const route = useRoute();
const store = useDashboardStore();

console.log("params from url:", route.params);

const ipAddress = computed(() => {
  return Array.isArray(route.params.id) ? route.params.id[0] : route.params.id;
});

onMounted(() => {
  console.log("Ip:", ipAddress.value);

  if (store.fileData.length === 0) {
    store.fetchFileData().then(() => {
      store.fetchIpReport(ipAddress.value as string);
    });
  } else {
    store.fetchIpReport(ipAddress.value as string);
  }
});
</script>

<template>
  <div>
    <h2>IP Report for {{ ipAddress }}</h2>

    <div v-if="store.selectedIpReport">
      <p><strong>Domain:</strong> {{ store.selectedIpReport.domain }}</p>
      <p><strong>Abuse Score:</strong> {{ store.selectedIpReport.abuseConfidenceScore }}</p>
      <p><strong>Country:</strong> {{ store.selectedIpReport.country }}</p>
      <p><strong>Total Reports:</strong> {{ store.selectedIpReport.totalReports }}</p>

      <h3>Reports:</h3>
      <ul>
        <li v-for="report in store.selectedIpReport.reports" :key="report.reporterId">
          <p><strong>Reported At:</strong> {{ report.reportedAt }}</p>
          <p><strong>Comment:</strong> {{ report.comment }}</p>
          <p><strong>Reporter Country:</strong> {{ report.reporterCountryName }}</p>
        </li>
      </ul>
    </div>

    <p v-else>You should select an Ip</p>

    <router-link to="/" class="back-button">Back to Dashboard</router-link>
  </div>
</template>
```

## Store(dashStores.js):

```js
const fetchFileData = async () => {
        try {
            const res = await axios.get(import.meta.env.VITE_API_URL + '/files/with-ip-check');
            fileData.value = res.data;
        } catch (error) {
            console.error('Error fetching data:', error);
        }
    };

const fetchIpReport = (ip: string) => {
        console.log("IP:", ip);
        console.log("fileData:", fileData.value);

        let foundReport = fileData.value.find((item) => item.ip_address === ip)?.ip_check;

        if (!foundReport) {
            for (const mail of fileData.value) {
                if (mail.details) {
                    const detailReport = mail.details.find((detail) => detail.ip_address === ip)?.ip_check;
                    if (detailReport) {
                        foundReport = detailReport;
                        break;
                    }
                }
            }
        }
        console.log("foundReport:", foundReport);

        if (foundReport) {
            selectedIpReport.value = foundReport;
        } else {
            selectedIpReport.value = null;
        }
        console.log("report:", selectedIpReport.value);
    };
```

## fileData fetched will look Like:
![alt text](image2.png)

## the data will be proped like that:
![alt text](image3.png)

