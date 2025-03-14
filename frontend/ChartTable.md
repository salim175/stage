# Create Table with expand:

### ChartTable.vue(Script):
```ts
// Interfaces for type safety ``Mail`` represents an individual email.
interface Mail {
  _id?: string | number;
  from: string ;
  date: string ;
  ip_address: string;
  total?: string;
  details?: Mail[] ;
}

// define the headers for the table
interface TableHeader{
  title: string,
  key: keyof Mail,
  align?: "start" | "center" | "end",
  sortable?: boolean
}

// Reactive state for fetched data
const fileData = ref<Mail[]>([]);
const filteredData = ref<Mail[]>([]);

// prop teh selected date from chartLine
const props = defineProps({
  selectedDate: {
    type: String,
    default: ''
  }
});

// Fetch function
const fetchFileData = async () => {
  try {
    const res = await axios.get(import.meta.env.VITE_API_URL + '/info/file-data');
    fileData.value = res.data;
    filterDataByDate();
  } catch (error) {
    console.log('Error fetching data:', error);
  }
};

// Headers for the table
const headers = computed<TableHeader[]>(() => [
  { title: `${filteredData.value.length} FROM`, key: "from", align: "start", sortable: false, width: "60%" },
  { title: "DATE", key: "date", align: "center", sortable: true, width: "10%"   },
  { title: "IP ADDRESS", key: "ip_address", align: "center", sortable: false, width: "10%"   },
  { title: "TOTAL", key: "total", align: "center", sortable: false, width: "10%" },
  { title: "DETAILS", key: "details", align: "center", sortable: false, width: "10%"   },
])


// Function to format timestamp to dd/mm/yyyy
const formatTimestampToDate = (timestamp: number) => {
  return new Intl.DateTimeFormat("fr-FR", {
    day: "2-digit",
    month: "2-digit",
    year: "numeric",
  }).format(new Date(timestamp * 1000));
};

// Filter data by date
const filterDataByDate = () => {
  console.log('Filtering by date:', props.selectedDate);

  // Get today's date in dd/mm/yyyy Format
  const todayDate = formatTimestampToDate(Date.now() / 1000);

  // If selectedDate is set, it filters emails that match that date, else it shows today's emails.
  filteredData.value = fileData.value.filter(file => {
    const fileDate = formatTimestampToDate(Number(file.date));
    return props.selectedDate ? fileDate === props.selectedDate : fileDate === todayDate;
  });

  // console.log(filteredData.value)
  console.log('Filtered files count:', filteredData.value.length);
};

// **Group data only for duplicated IPs**
const groupedData = computed((): (Mail )[] => {
  const ipCount: Record<string, number> = {}; // will be like "192.168.1.1": 1 strin with number
  const groups: Record<string, Mail[]> = {}; // "192.168.1.2":[ { _id: 2, from: "", date: "",....} ] string with array
  const uniqueData: Mail[] = [];

  // Count occurrences of each IP
  filteredData.value.forEach((file) => {
    ipCount[file.ip_address] = (ipCount[file.ip_address] || 0) + 1;
    console.log(file.ip_address)
    console.log('test',ipCount[file.ip_address])
    console.log('TESTTTTTT',ipCount)
  });

  // Separate unique and duplicate IPs
  filteredData.value.forEach((file) => {
    if (ipCount[file.ip_address] > 1) {
      if (!groups[file.ip_address]) {
        groups[file.ip_address] = [];
      }
      groups[file.ip_address].push(file);
    } else {
      uniqueData.push(file); // Keep unique emails with _id
    }
  });

  // Convert grouped IPs into a structured array
  const groupedArray: Mail[] = Object.keys(groups).map((ip) => ({
    ip_address: ip,
    from: groups[ip][0].from,
    date: groups[ip][0].date,
    total: groups[ip].length.toString(),
    details: [...groups[ip]],  // This is an array of duplicates, Ensure it's always Mail[]
  }));

  console.log(groupedArray);

  return [...uniqueData, ...groupedArray]; // Combine unique and grouped data
});

// Expanded row state
const expanded = ref<string | null>(null)

// Computed for expanded item
const expandedItem = computed(() => 
  groupedData.value.find(item => item.ip_address === expanded.value)
)

// Toggle expand method
const toggleExpand = (item: Mail) => {
  expanded.value = expanded.value === item.ip_address ? null : item.ip_address
}

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

onMounted(fetchFileData);
```

``filedata``: Holds all fetched emails

``filteredData``: Holds emails filtered by selected Date

``Fetch``: calls ``filterDataByDate`` to apply date filtering

``selectedDate``: is passed from ``DefaultDashboard`` (Parent)

## filterDataByDate Function():
- Convert the date from timestamp to human Readable
- If ``selectedDate`` is set, it filters emails that match that date. (passed the date to ``filteredData.value``)
- If ``selectedDate`` is not set, it shows today's emails. (passed the date to ``filteredData.value``)

## GroupedData Function():
- ``ipCount``: Stores how many times each IP address appears.
- ``groups``: Stores emails grouped by IP address.
- ``uniqueData``: Stores emails that have a unique IP.

```js
// ipCount will be like "192.168.1.1": 1 string with number
filteredData.value.forEach((file) => {
  ipCount[file.ip_address] = (ipCount[file.ip_address] || 0) + 1;
});
```

1. ``file.ip_address`` is used as a key in ``ipCount`` (an object tracking occurrences of each IP).
2. If ``file.ip_address`` already exists in ``ipCount``:
    - It retrieves the current count and adds ``1`` to it.
3. If ``file.ip_address`` does not exist in ``ipCount``:
    - ``ipCount[file.ip_address]`` is ``undefined``, so it defaults to **0**, then adds **1**.

        ---

```js
filteredData.value.forEach((file) => {
  if (ipCount[file.ip_address] > 1) {
    if (!groups[file.ip_address]) {
      groups[file.ip_address] = [];
    }
    groups[file.ip_address].push(file);
  } else {
    uniqueData.push(file);
  }
});
```
- If an IP appears more than once, it is added to ``groups``.
- If an IP appears only once, it is added to ``uniqueData``.

    ---

```js
const groupedArray: GroupedMail[] = Object.keys(groups).map((ip) => ({
  ip_address: ip,
  from: groups[ip][0].from,
  date: groups[ip][0].date,
  total: groups[ip].length.toString(),
  details: [...groups[ip]], // Ensure it's always an array
}));
```

- Each grouped IP becomes a GroupedMail object.
- total field stores the number of emails.
- details field stores all emails under that IP.

## Expanding:
```js
const expandedItem = computed(() => 
  groupedData.value.find(item => item.ip_address === expanded.value)
);
```
- Finds the expanded row by matching ip_address.
- If expanded.value contains an IP address, it searches groupedData for an item with the same ip_address and returns it.
- If no row is expanded (expanded = null), it returns undefined.

```js
const toggleExpand = (item: Mail | GroupedMail) => {
  expanded.value = expanded.value === item.ip_address ? null : item.ip_address;
};
```
- If the row is already expanded, clicking it closes it (sets expanded = null).
- If another row is clicked, it updates expanded.value to the clicked item's ip_address