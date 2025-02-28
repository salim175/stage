# Create table to display data:
### EmailsInfo.vue (Script):
```js
interface FileData {
  _id: string | number;
  from: string | number;
  date: string ;
  ip_address: string;
  total: string;
}

const props = defineProps({
  selectedDate: {
    type: String,
    default: ''
  }
});

const fileData = ref<FileData[]>([]);
const filteredData = ref<FileData[]>([]);

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

// filter data by date
const filterDataByDate = () => {
  console.log('Filtering by date:', props.selectedDate);

  // get today's date in dd/mm/yyyy Format
  const todayDate = new Intl.DateTimeFormat("fr-FR", {
      day: "2-digit",
      month: "2-digit",
      year: "numeric",
    }).format(new Date());
  
  if (!props.selectedDate) {
    // If no date is selected, show Today's files
    filteredData.value = fileData.value.filter(file => {
      const timestampDate = Number(file.date);
      const formattedDate = new Intl.DateTimeFormat("fr-FR", {
        day: "2-digit",
        month: "2-digit",
        year: "numeric",
      }).format(new Date(timestampDate * 1000));

      return formattedDate === todayDate;
    });

    console.log("Showing today's files:", filteredData.value.length);
  } else {    
    // Filter files by selected date
    filteredData.value = fileData.value.filter(file => {
      // convert the date of file from fileData array  to number 
      const timestampDate = Number(file.date)
      // convert the date to compare it with the selected date(the date will still be in timestamp)
      const formattedDate = new Intl.DateTimeFormat("fr-Fr").format(new Date(timestampDate*1000));
      return formattedDate === props.selectedDate;
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
</script>
```

**the data in the console will be like:**

![alt text](/images/image1.png)

### (1) When You Filter Data (filterDataByDate()):

    - You convert date to DD/MM/YYYY for comparison only, but you don't modify item.date itself.
    - The original item.date (timestamp) remains unchanged inside filteredData.

### (2) When Displaying Data in the Table
    - filteredData still contains timestamps (item.date).
    - In <td>, formatDate(item.date) is used to convert the timestamp for display.

## Script:
if fileData is a ref(Vue Ref), that's mean it is an object that holds the actual data inside value

📌 Explanation
In script `(<script setup>)`, fileData is a ref(), so you must access .value:

```ts
console.log(fileData.value?.to); // ✅ Correct in script
```
However, in Vue's template `<template>`, Vue automatically unwraps ref(), so you can omit .value:

```html
<v-list-item-title> {{ fileData?.to }} </v-list-item-title>
```