# Create table to display data:
### EmailsInfo.vue (Script):
```js
interface FileData {
  _id: string | number;
  from: string | number;
  date: string ;
  ip_address: string;
  total?: string;
  details: '';
}

const fileData = ref<FileData[]>([]);
const filteredData = ref<FileData[]>([]);

const props = defineProps({
  selectedDate: {
    type: String,
    default: ''
  }
});

interface TableHeader{
  title: string,
  key: keyof FileData,
  align?: "start" | "center" | "end",
  sortable?: boolean
}

const headers = computed<TableHeader[]>(() => [
  { title: `${filteredData.value.length} FROM`, key: "from", align: "start", sortable: true, width: "50%" },
  { title: "DATE", key: "date", align: "center", sortable: true, width: "16%"  },
  { title: "IP ADDRESS", key: "ip_address", align: "center", sortable: false, width: "16%"  },
  { title: "DETAILS", key: "details", align: "center", sortable: false, width: "16%"  },
]);


// Fetch the data
const fetchFileData = async ()=> {
  try{
    const res = await axios.get(import.meta.env.VITE_API_URL + '/info/file-data')
    fileData.value = res.data;
    filterDataByDate();
  }catch(error){
    console.log(error);
  }
}

// Function to format timestamp to dd/mm/yyyy
const formatTimestampToDate = (timestamp: number) => {
  return new Intl.DateTimeFormat("fr-FR", {
    day: "2-digit",
    month: "2-digit",
    year: "numeric",
  }).format(new Date(timestamp * 1000));
};

// filter data by date
const filterDataByDate = () => {
  console.log('Filtering by date:', props.selectedDate);

  // get today's date in dd/mm/yyyy Format
  const todayDate = formatTimestampToDate(Date.now() / 1000);

  if (!props.selectedDate) {
    // If no date is selected, show Today's files
    filteredData.value = fileData.value.filter(file => {
      const fileDate = formatTimestampToDate(Number(file.date));
      return fileDate === todayDate;
    });

    console.log("Showing today's files:", filteredData.value.length);
  } else {    
    // Filter files by selected date
    filteredData.value = fileData.value.filter(file => {
      // convert the date of file from fileData array  to number using Number
      // convert the date to compare it with the selected date(the date will still be in timestamp)
      const fileDate = formatTimestampToDate(Number(file.date));
      return fileDate === props.selectedDate;
    });
    
    console.log('Filtered files count:', filteredData.value.length);
  }
};

// format to convert the date in the table from timestamp to human readable
const formatDate = (timestamp: string | number) => {
  return new Intl.DateTimeFormat("fr-FR", {
    day: "2-digit",
    month: "2-digit",
    year: "numeric",
    hour: "2-digit",
    minute: "2-digit",
    second: "2-digit"
  }).format(new Date(Number(timestamp) * 1000));
};

// Watch for changes in selectedDate prop
watch(() => props.selectedDate, () => {
  filterDataByDate();
});

onMounted(()=>{
  fetchFileData();
})
```

### Template:
```js
<v-data-table
      :headers="headers"
      :items="filteredData"
      :items-per-page="15"
      class="bordered-table"
      hover
      fixed-header
      density="comfortable"
      :height="filteredData.length > 0 ? '500px' : '100px'"
      :sort-by="[{ key: 'date', order: 'asc' }]"
    >

  <template v-slot:no-data>
    <div class="empty-table">
      There is no file uploaded Today
    </div>
  </template>

  <template v-slot:item.date="{item}">
    {{ formatDate(item.date) }}
  </template>

  <template v-slot:item.details="{ item }">
    <v-btn color="#F16E00" :to="`/details/${item._id}`">
      Plus D'infos
    </v-btn>
  </template>

  </v-data-table>
```

### headers and TableHeader:


---

**the data in the console will be like:**

![alt text](/images/image1.png)

### (1) When You Filter Data (filterDataByDate()):

    - You convert date to DD/MM/YYYY for comparison only, but you don't modify item.date itself.
    - The original item.date (timestamp) remains unchanged inside filteredData.

### (2) When Displaying Data in the Table
    - filteredData still contains timestamps (item.date).
    - In <td>, formatDate(item.date) is used to convert the timestamp for display.


## 📌 filterDataByDate:
### 1. Logging the Selected Date
- It logs the selected date (`props.selectedDate`) for debugging.

### 2. Getting Today's Date in `dd/mm/yyyy` Format
- It uses `formatTimestampToDate(Date.now() / 1000)` to get today’s date.

### 3. If No Date is Selected (`props.selectedDate` is `null` or `undefined`)
- It filters `fileData.value` to **only include emails from today**.
- Converts `file.date` (timestamp) to a **formatted date**.
- Compares the formatted date with today’s date.
- Stores the filtered data in `filteredData.value`.

### 4. If a Date is Selected (`props.selectedDate` exists)
- It filters `fileData.value` to **only include emails from the selected date**.
- Converts `file.date` (timestamp) to a **formatted date**.
- Compares it with `props.selectedDate`.
- Stores the filtered data in `filteredData.value`.


## 📌 Some notes:

- ``keyof FileData`` creates a union of the keys in ``FileData`` (``"_id" | "from" | "date" | ...``).
This ensures that ``key`` in ``TableHeader`` must be one of the actual keys in ``FileData``.

- if fileData is a ref(Vue Ref), that's mean it is an object that holds the actual data inside value

```ts
// In script `(<script setup>)`, fileData is a ref(), so you must access .value:
console.log(fileData.value?.to); // ✅ Correct in script
```

```html
<!-- However, in Vue's template `<template>`, Vue automatically unwraps ref(), so you can omit .value: -->
<v-list-item-title> {{ fileData?.to }} </v-list-item-title>
```

---

- <u>Note:</u> for the reset-filter, the explanation is in the ```chartline_front```