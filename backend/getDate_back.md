# Convert the date from timestamp to human readable:

```js
app.get('/api/file-counts', async (req, res) => {
    try {
        const dailyCounts = await File.aggregate([
            {
                $match: { date: { $exists: true, $ne: null } },
            },
            {
                $group: {
                    // keep the date in timestamp format and convert it to number with $toLong
                    _id: { $toLong: "$date" },
                    count: { $sum: 1 }
                },
            },
            { $sort: { _id: 1 } }
        ]);

        const monthlyCounts = await File.aggregate([
            {
                $match: { date: { $exists: true, $ne: null } },
            },
            {// Convert to Date to human readAble
                $addFields: { date: { $toDate: { $multiply: [{ $toLong: "$date" }, 1000] } } },
            },
            {
                $group: {
                    _id: { $dateToString: { format: "%m-%Y", date: "$date", timezone: "Europe/Paris" } },
                    count: { $sum: 1 },
                },
            },
            { $sort: { _id: 1 } }
        ]);

        res.json({
            daily: dailyCounts,
            monthly: monthlyCounts
        });
    } catch (error) {
        console.error('Error fetching file counts:', error);
        res.status(500).json({ error: 'Internal Server Error' });
    }
});
```

## 📌 $match:
✅ Purpose:

- Removes documents where the date field is missing ($exists: true).
- Removes null values ($ne: null).

🔥 Why is this important?
If some documents do not have a date field, they would cause errors in the next steps.

## 📌 $addFields:
✅ Purpose:

- Converts the date field from a string to a number using $toLong: "$date".
- Multiplies by 1000 to convert seconds → milliseconds (MongoDB stores dates in milliseconds).
- Uses $toDate to convert the final number into a proper Date object.

🔥 Example Conversion:

| MongoDB Original     | **Step 1:** `$toLong` | **Step 2:** `$multiply` | **Step 3:** `$toDate`           |
|----------------------|----------------------|-------------------------|---------------------------------|
| `"1738346520"` (String) | `1738346520` (Number) | `1738346520000` (Milliseconds) | `ISODate("2025-02-01T10:42:00Z")` |

## 📌 $group:
✅ Purpose:

- Converts full Date object into just YYYY-MM-DD ($dateToString).
- Groups files uploaded on the same day together.
- Counts occurrences per day ($sum: 1).

## 📌 $sort:
✅ Purpose:

- Sorts the grouped data in chronological order (1 = oldest to newest).
- Ensures the chart displays dates correctly (not shuffled).

# 📌 Summary (with example):

| **File ID** | **Original Date (CET/CEST - Paris Time)** | **Stored UNIX Timestamp (UTC)** | **Converted UTC Time** | **MongoDB Default (UTC Grouping - Wrong)** | **MongoDB Grouping with `Europe/Paris` (Correct)** |
|------------|----------------------------------|------------------------|----------------------|---------------------------------|--------------------------------|
| **1️⃣** | **Feb 28, 2025 - 23:45 CET** | `1743463500` | **Feb 28, 2025 - 22:45 UTC** | **Grouped as February** ❌ | **Grouped as February** ✅ |
| **2️⃣** | **Mar 1, 2025 - 00:46 CET** | `1743475600` | **Feb 28, 2025 - 23:46 UTC** | **Grouped as February** ❌ | **Grouped as March** ✅ |
