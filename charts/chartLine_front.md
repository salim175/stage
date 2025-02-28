# Create a ChartLine with vue:

```js
const fileData = ref<{
  daily: {_id: string; count: number; formattedDate: string}[];
  monthly: { _id: string; count: number }[];
}>({ daily: [], monthly: [] });

const emit = defineEmits(["date-selected"]);

const fetchFileCounts = async () => {
  try {
    const response = await axios.get(import.meta.env.VITE_API_URL + '/counter/file-counts');
    fileData.value = response.data;
  } catch (error) {
    console.error('Error fetching data:', error);
  }
};

// chart 1: Number of Mails/Day
const groupedDailyData = computed(() => {
  const grouped: Record<string, number> = {};

  fileData.value.daily.forEach((item) => {
    const timestamp = Number(item._id);
    console.log(timestamp);
    const formattedDate = new Intl.DateTimeFormat('fr-FR', {
      day: '2-digit',
      month: '2-digit',
      year: 'numeric',
    }).format(new Date(timestamp * 1000));
    console.log(formattedDate);
    console.log(typeof formattedDate);

    if (!grouped[formattedDate]) {
      grouped[formattedDate] = 0;
    }
    grouped[formattedDate] += item.count;
  });
  // Return and Convert the object into an array
  return Object.entries(grouped).map(([date, count]) => ({
    date,
    count,
  }));
});


const areaChart1 = computed(() => ({
  series: [
    {
      name: 'Emails',
      data: groupedDailyData.value.map(item => item.count),
    }
  ]
}));

const chartOptions1 = computed(() => {
  return {
    chart: {
      type: 'area',
      height: 450,
      fontFamily: `inherit`,
      foreColor: 'rgba(var(--v-theme-secondary), var(--v-high-opacity))',
      toolbar: {show:true},
      events: {
        dataPointSelection: (event: Event, chartContext: never, opts: {dataPointIndex: number}) => {
          const selectedIndex = opts.dataPointIndex;
          const selectedData = groupedDailyData.value[selectedIndex];
          const selectedDate = selectedData.date;
          
          console.log('Index selected:', selectedIndex); // Index selected: 19
          console.log('data selected:', selectedData); // data selected: {date: '18/02/2025', count: 12}
          console.log('Date selected:', selectedDate); // Date selected: 18/02/2025
          
          emit("date-selected", selectedDate);
          }
      }
    },
    markers: { size: 3 },
    tooltip: {
    intersect: true,
    shared: false, 
  },
    colors: [primaryColor, darkprimaryColor],
    labels: groupedDailyData.value.map(item => item.date),
    dataLabels: { enabled: false },
    stroke: { curve: 'smooth', width: 2 },
    fill: {
      type: 'gradient',
      gradient: {
        shadeIntensity: 1,
        opacityFrom: 0.7,
        opacityTo: 0.4,
        stops: [0, 100]
      }
    },
    grid: { borderColor: 'rgba(var(--v-theme-borderLight), var(--v-high-opacity))' },
    xaxis: {
      type:'category',
      categories:groupedDailyData.value.map(item => item.date),
      labels: {
        rotate: -45,
        style: {
          fontSize: '12px'
        }
      },
      axisBorder: { show: true, color: 'rgba(var(--v-theme-borderLight), var(--v-high-opacity))' },
      axisTicks: { color: 'rgba(var(--v-theme-borderLight), var(--v-high-opacity))' }
    },
    legend: { show: true }
  };
});

const tab = ref(1);

onMounted(fetchFileCounts);

<template>
  <v-card class="title-card" variant="text">
    <v-card-text class="rounded-md overflow-hidden">
      <v-window v-model="tab">
        <v-window-item value="two">
          <apexchart type="area" height="450" :options="chartOptions1" :series="areaChart1.series"> </apexchart>
        </v-window-item>
      </v-window>
    </v-card-text>
  </v-card>
</template>
```

## 📌 Fetch the data from getDate_back:
The data will be like: 
```js
{
    "daily": [
        {
            "_id": "1738272960", // the date is in timestamp format (string)
            "count": 2
        },
        {
            "_id": "1738273500",
            "count": 2
        },
    ],
    "monthly": [
        {
            "_id": "2025-01",
            "count": 18
        },
        {
            "_id": "2025-02",
            "count": 597
        }
    ]
}
```

## 📌 groupedDailyData:

The grouped object is used to **store** and **sum counts** by formatted date **(DD/MM/YYYY)**.

- ``Record<K, V>`` is a TypeScript utility type.
    - K = string → The keys will be dates ("21/02/2025", "22/02/2025").
    - V = number → The values will be count totals (8, 15).

This means grouped is an object where:
- Each key is a formatted date string ("DD/MM/YYYY").
- Each value is a sum of counts for that date.

Convert ``grouped`` Object into an array:
```js 
return Object.entries(grouped).map(([date, count]) => ({
    date,
    count,
  }));
```
**the returned data will be like:**

![the data](/images/image.png) 

## 📌 dataPointSelection:
- ``dataPointIndex``: return the index of selected point in the chart
- ``selectedData``: is the the data of selected date
- ``emit``: is used to send events from a child component to its parent.

```js
SENDER CHILD (ChartLine.vue):
const emit = defineEmits(["date-selected"]); // ✅ Declare event emitter

emit("date-selected", selectedDate); // ✅ Send selected date to parent

PARENT :
const selectedDate = ref(""); // ✅ Stores the selected date

const handleDateSelected = (date: string) => {
  selectedDate.value = date; // ✅ Updates when event is emitted
};

const resetFilter = () => {
  selectedDate.value = ""; // ✅ Clears the filter when reset is clicked
};

<template>
  <ChartLine @date-selected="handleDateSelected"/>
  <EmailsInfo :selectedDate="selectedDate" @reset-filter="resetFilter"/>
</template>

RECIEVED CHILD (EmailsInfo.vue):
// Receives selectedDate as a prop to filter emails.
// Emits "reset-filter" to clear selectedDate when the reset button is clicked.

const props = defineProps({
  selectedDate: {
    type: String,
    default: "",
  },
});
<v-btn size="small" class="ml-2" color="#F16E00" @click="$emit('reset-filter')">Show today's file</v-btn>



1️⃣ User selects a date in ChartLine.vue
   ⬇
2️⃣ emit("date-selected", selectedDate) → Sends date to Parent.vue
   ⬇
3️⃣ Parent.vue listens (@date-selected="handleDateSelected")
   ⬇
4️⃣ Parent updates `selectedDate` and passes it as a prop to EmailsInfo.vue
   ⬇
5️⃣ EmailsInfo.vue filters data using `selectedDate`
   ⬇
6️⃣ Reset button in EmailsInfo.vue emits ("reset-filter") → Parent clears `selectedDate`
```