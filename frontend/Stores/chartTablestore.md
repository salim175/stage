# create a store for the Table

```js
export const useDashboardStore = defineStore('dashboard', () => {
    const fileData = ref<Mail[]>([]);
    const selectedDate = ref('');
    const expandedItem = ref<string | null>(null);
    // const selectedIpReport = ref<Report | null>(null);

    const fetchFileData = async () => {
        try {
            // const res = await axios.get(import.meta.env.VITE_API_URL + '/files/with-ip-check');
            const res = await axios.get(import.meta.env.VITE_API_URL + '/info/mail-data');
            fileData.value = res.data;
        } catch (error) {
            console.error('Error fetching data:', error);
        }
    };

    // convert date from timestamp to dd/mm/yyyyy
    const formatTimestampToDate = (timestamp: string | number) => {
        return new Intl.DateTimeFormat("fr-FR", {
            day: "2-digit",
            month: "2-digit",
            year: "numeric",
        }).format(new Date(Number(timestamp) * 1000));
    };

    // Get today date in dd/mm/yyyy format
    const todayDate = formatTimestampToDate(Date.now() / 1000);

    // Filter emails based on selected date
    const filteredData = computed(() => {
        return fileData.value.filter(file => {
            const fileDate = formatTimestampToDate(Number(file.date));
            return selectedDate.value ? fileDate === selectedDate.value : fileDate === todayDate;
        });
    });

    console.log("Filtered Data", filteredData)

    // Group data only for duplicated IPs
    const groupedData = computed((): (Mail | GroupedMail)[] => {
        const ipCount: Record<string, number> = {}; // will be like "192.168.1.1": 1 / strin with number
        const groups: Record<string, Mail[]> = {};  // "192.168.1.2":[ { _id: 2, from: "", date: "",....} ] / string with array
        const uniqueData: Mail[] = [];

        filteredData.value.forEach((file) => {
            ipCount[file.ip_address] = (ipCount[file.ip_address] || 0) + 1;
        });

        // Separate unique and duplicate IPs
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

        // Convert grouped IPs into a structured array
        const groupedArray: GroupedMail[] = Object.keys(groups).map((ip) => ({
            ip_address: ip,
            from: groups[ip][0].from,
            date: groups[ip][0].date,
            total: groups[ip].length.toString(),
            details: [...groups[ip]],
        }));

        return [...uniqueData, ...groupedArray];
    });

    const formatHostnames = (hostnames: string[] | undefined) => {
        return hostnames && hostnames.length > 0 ? hostnames.join(", ") : "N/A";
    };

    const formatDate = (date: string | number) => {
        return new Intl.DateTimeFormat("fr-FR", {
            day: "2-digit",
            month: "2-digit",
            year: "numeric",
            hour: "2-digit",
            minute: "2-digit",
            second: "2-digit"
        }).format(new Date(date));
    };

    return {
        fileData,
        selectedDate,
        expandedItem,
        filteredData,
        groupedData,
        todayDate,
        fetchFileData,
        formatDate,
        formatHostnames
    };
});
```

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

### example:
```js
const ipCount = {}; // start empty

// file 1
ipCount['A'] = (undefined || 0) + 1 // => 1

// file 2
ipCount['A'] = (1 || 0) + 1         // => 2

// file 3
ipCount['B'] = (undefined || 0) + 1 // => 1

Result:
ipCount = {
  A: 2,
  B: 1
}
```

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