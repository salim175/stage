# Create the store for Pie chart

```js
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import axios from 'axios';

export interface From {
    _id: string,
    count: number,
    attackers: { ip: string; count: number }[]
}

export const usePieStore = defineStore('pie', () => {
    const fromData = ref<From[]>([]);
    const chartData = ref<{ name: string; value: number }[]>([]);

    // Fetch function
    const fetchFromCounts = async () => {
        try {
            const response = await axios.get(import.meta.env.VITE_API_URL + '/counter/from-counts')
            fromData.value = response.data
            chartData.value = transformGroupedChartData(response.data)
        } catch (error) {
            console.error('Error fetching data:', error)
        }
    }

    // convert the data from the api into chart format (name, value)
    function transformGroupedChartData(raw: From[]): { name: string; value: number }[] {
        const groupMap = new Map<number, string[]>()

        raw.forEach(item => {
            const key = item.count
            if (!groupMap.has(key)) {
                groupMap.set(key, [])
            }
            groupMap.get(key)!.push(item._id)
        })

        console.log(groupMap)
        // console.log([...groupMap.entries()])
        return Array.from(groupMap.entries()).map(([count, ids]) => ({
            name: ids.join(', '),
            value: Number(count),
        }))
    }

    return {
        fromData,
        chartData,
        fetchFromCounts
    }
})
```

## 📌 The grouped function:
```js
if (!groupMap.has(key)) {
    groupMap.set(key, [])
}
groupMap.get(key)!.push(item._id)
```
| Line                               | Description                                                                 |
|------------------------------------|-----------------------------------------------------------------------------|
| `if (!groupMap.has(key))`         | Checks if the Map already has an entry for the current key (which is `item.count`). |
| `groupMap.set(key, [])`           | If it doesn’t, it initializes a new empty array for that key.              |
| `groupMap.get(key)!.push(item._id)` | Pushes the current `item._id` into the array for that key. The `!` tells TypeScript we are sure the key exists (non-null assertion). |

the returned data will be like:
```js
Map {
  1 => ['idA', 'idB'],
  2 => ['idC']
}
```
---
## 📌 The final result:

```ts
[
  { name: "id1, id2", value: 3 },
  { name: "id3", value: 1 }
]
```
