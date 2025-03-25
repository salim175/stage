# the Store of the chartline:
```js
... code ... 

export const useChartStore = defineStore('chart', () => {
    const fileData = ref<{
        daily: { _id: number; count: number; }[];
        monthly: { _id: string; count: number }[];
    }>({ daily: [], monthly: [] });

    const currentWeekStart = ref(dayjs().isoWeekday(1)); // start from monday
    const earliestTimestamp = ref<number | null>(null);
    const isDataFetched = ref(false); // Prevent multiple fetches

    const fetchChartData = async () => {
        if (isDataFetched.value) return; // Prevent re-fetching
        try {
            const res = await axios.get(import.meta.env.VITE_API_URL + '/counter/file-counts');
            fileData.value = res.data;
            isDataFetched.value = true;

            if (fileData.value.daily.length > 0) {
                earliestTimestamp.value = Math.min(...fileData.value.daily.map((item) => item._id));
            }
        } catch (error) {
            console.error('Error fetching chart data:', error);
        }
    };

    // Compute weekly filtered data for the chart
    const groupedWeeklyData = computed(() => {
        const startOfWeek = currentWeekStart.value.startOf('day').unix();
        // console.log('startOfWeek', startOfWeek);
        const endOfWeek = currentWeekStart.value.add(6, 'day').endOf('day').unix();

        const grouped: Record<string, number> = {};
        fileData.value.daily.forEach((item) => {
            const timestamp = item._id;
            if (timestamp >= startOfWeek && timestamp <= endOfWeek) {
                const formattedDate = dayjs.unix(timestamp).format('DD/MM/YYYY');
                grouped[formattedDate] = (grouped[formattedDate] || 0) + item.count;
                // console.log('grouped[formattedDate]', grouped)
            }
        });


        return Object.entries(grouped).map(([date, count]) => ({ date, count }));
    });

    // Compute Yearly Data
    const groupedYearlyData = computed(() => {
        return fileData.value.monthly.map((item) => ({
            month: item._id, // Example: "03-2025"
            count: item.count
        }));
    });

    ... code of Navigation in weeks ... 

    return {
        fileData,
        currentWeekStart,
        earliestTimestamp,
        groupedWeeklyData,
        groupedYearlyData,
        fetchChartData,
        changeWeek,
        disableNextWeek,
        disablePrevWeek,
        isDataFetched,
    };
});
```
## 📌groupedWeeklyData Function():

### **1️⃣ Why Do We Group Emails?**
- The API returns **timestamps** (Unix format) with exact times (`DD/MM/YYYY HH:mm:ss`).
- We **only care about the date** (`DD/MM/YYYY`), so we **group emails from the same day**.

### **2️⃣ How We Convert & Group Dates**
#### **🔹 Original Data (Example)**
| **Timestamp** | **Original Date (DD/MM/YYYY HH:mm:ss)** | **Converted Date (DD/MM/YYYY)** |
|--------------|----------------------|----------------|
| `1742218080` | `17/03/2025 14:48:00` | `17/03/2025` |
| `1742304480` | `18/03/2025 14:48:00` | `18/03/2025` |
| `1742304480` | `18/03/2025 21:30:00` | `18/03/2025` |

#### **🔹 Code to Convert & Group**
```ts
const grouped: Record<string, number> = {}; 

fileData.value.daily.forEach((item) => {
    const timestamp = item._id; // Email's timestamp in seconds
    const formattedDate = dayjs.unix(timestamp).format('DD/MM/YYYY'); // Convert to "DD/MM/YYYY"

    // If the date already exists in grouped, add the count
    if (grouped[formattedDate]) {
        grouped[formattedDate] += item.count;
    } 
    // Otherwise, create a new entry for this date
    else {
        grouped[formattedDate] = item.count;
    }
});
```

### **3️⃣ Final Grouped Data**
After running the code, the grouped object looks like this:
```json
{
  "17/03/2025": 5,
  "18/03/2025": 12 // (8 + 4 combined)
}
```
✅ Emails from the same date are combined into a single entry!

### 4️⃣ **The return**

we need an array like the <u>after</u> for the chart:
```js
Before:                     After:
const grouped = {           [
  "17/03/2025": 5,            { date: "17/03/2025", count: 5 },
  "18/03/2025": 12,           { date: "18/03/2025", count: 12 },
  "19/03/2025": 7             { date: "19/03/2025", count: 7 }
};                          ]
```

``Object.entries(grouped)``: 
```js
[
  ["17/03/2025", 5],
  ["18/03/2025", 12],
  ["19/03/2025", 7]
]
```
``.map``: takes each [date, count] pair and converts it into an object { date, count }. 

# 📌The complete code : 
```js
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import axios from 'axios';
import dayjs from 'dayjs';
import isoWeek from 'dayjs/plugin/isoWeek';

dayjs.extend(isoWeek);

export const useChartStore = defineStore('chart', () => {
    const fileData = ref<{
        daily: { _id: number; count: number; }[];
        monthly: { _id: string; count: number }[];
    }>({ daily: [], monthly: [] });

    const currentWeekStart = ref(dayjs().isoWeekday(1));
    const earliestTimestamp = ref<number | null>(null);
    const isDataFetched = ref(false); // Prevent multiple fetches

    const fetchChartData = async () => {
        if (isDataFetched.value) return; // Prevent re-fetching
        try {
            const res = await axios.get(import.meta.env.VITE_API_URL + '/counter/file-counts');
            fileData.value = res.data;
            isDataFetched.value = true;

            if (fileData.value.daily.length > 0) {
                earliestTimestamp.value = Math.min(...fileData.value.daily.map((item) => item._id));
            }
        } catch (error) {
            console.error('Error fetching chart data:', error);
        }
    };

    // Compute weekly filtered data for the chart
    const groupedWeeklyData = computed(() => {
        const startOfWeek = currentWeekStart.value.startOf('day').unix();
        // console.log('startOfWeek', startOfWeek);
        const endOfWeek = currentWeekStart.value.add(6, 'day').endOf('day').unix();

        const grouped: Record<string, number> = {};
        fileData.value.daily.forEach((item) => {
            const timestamp = item._id;
            if (timestamp >= startOfWeek && timestamp <= endOfWeek) {
                const formattedDate = dayjs.unix(timestamp).format('DD/MM/YYYY');
                grouped[formattedDate] = (grouped[formattedDate] || 0) + item.count;
                // console.log('grouped[formattedDate]', grouped)
            }
        });


        return Object.entries(grouped).map(([date, count]) => ({ date, count }));
    });

    // Compute Yearly Data
    const groupedYearlyData = computed(() => {
        return fileData.value.monthly.map((item) => ({
            month: item._id, // Example: "03-2025"
            count: item.count
        }));
    });

    // Function to navigate weeks
    const changeWeek = (direction: 'prev' | 'next') => {
        currentWeekStart.value = currentWeekStart.value.add(direction === 'prev' ? -7 : 7, 'day').isoWeekday(1);
    };

    // Disable next week button
    const disableNextWeek = computed(() => {
        const startOfDisplayedWeek = currentWeekStart.value.startOf('week');
        const endOfDisplayedWeek = currentWeekStart.value.endOf('week');
        return dayjs().isBetween(startOfDisplayedWeek, endOfDisplayedWeek, 'day', '[]');
    });

    // Disable previous week button
    const disablePrevWeek = computed(() => {
        if (!earliestTimestamp.value) return true;
        const startOfDisplayedWeek = currentWeekStart.value.unix();
        return startOfDisplayedWeek <= earliestTimestamp.value;
    });

    return {
        fileData,
        currentWeekStart,
        earliestTimestamp,
        groupedWeeklyData,
        groupedYearlyData,
        fetchChartData,
        changeWeek,
        disableNextWeek,
        disablePrevWeek,
        isDataFetched,
    };
});
```