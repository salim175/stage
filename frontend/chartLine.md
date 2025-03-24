# Create a ChartLine and use Store:

### Script:
```js
... code ...
import { useChartStore } from '@/stores/chartLineStore';
import { useDashboardStore } from '@/stores/dashStores';
import dayjs from 'dayjs';
import isBetween from 'dayjs/plugin/isBetween';

dayjs.extend(isBetween); // Enable the `isBetween` plugin

... code ...

const chartStore = useChartStore();
const store = useDashboardStore();

const areaChart1 = computed(() => ({
  series: [
    {
      name: 'Emails',
      data: chartStore.groupedWeeklyData.map((item) => item.count),
    }
  ]
}));

const chartOptions1 = computed(() => {
  return {
    chart: {
      ... code ...
      events: {
        dataPointSelection: (event: Event, chartContext: never, opts: { dataPointIndex: number }) => {
          const selectedIndex = opts.dataPointIndex;
          const selectedData = chartStore.groupedWeeklyData[opts.dataPointIndex];
          
          if (selectedData) {
            store.selectedDate = selectedData.date;
            console.log('Date selected:', store.selectedDate);
          }
          // console.log('Index selected:', selectedIndex); // Index selected: 19
          // console.log('data selected:', selectedData); // data selected: {date: '18/02/2025', count: 12}
          // console.log('Date selected:', store.selectedDate); // Date selected: 18/02/2025
        },
      },
    },
   ... code ...
    labels: chartStore.groupedWeeklyData.map((item) => item.date),
    ... code ...
    xaxis: {
      type:'category',
      categories: chartStore.groupedWeeklyData.map((item) => item.date),
      labels: {
        rotate: -45,
        style: {
          fontSize: '12px'
        }
      },
      axisBorder: { show: true, color: 'rgba(var(--v-theme-borderLight), var(--v-high-opacity))' },
      axisTicks: { color: 'rgba(var(--v-theme-borderLight), var(--v-high-opacity))' }
    },
    l... code ...
  };
});

onMounted(chartStore.fetchChartData);
```
### Template:
```
<v-window-item value="one">
    <apexchart type="area" height="450" :options="chartOptions2" :series="areaChart2.series"> </apexchart>
</v-window-item>
```


## stores:
the useDashboardStore is to send the selected date to chartTable ``store.selectedDate = selectedData.date;``
the useChartStore is to get the functions and the grouped data