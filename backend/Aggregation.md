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

##  📌 Summary (with example):

| **File ID** | **Original Date (CET/CEST - Paris Time)** | **Stored UNIX Timestamp (UTC)** | **Converted UTC Time** | **MongoDB Default (UTC Grouping - Wrong)** | **MongoDB Grouping with `Europe/Paris` (Correct)** |
|------------|----------------------------------|------------------------|----------------------|---------------------------------|--------------------------------|
| **1️⃣** | **Feb 28, 2025 - 23:45 CET** | `1743463500` | **Feb 28, 2025 - 22:45 UTC** | **Grouped as February** ❌ | **Grouped as February** ✅ |
| **2️⃣** | **Mar 1, 2025 - 00:46 CET** | `1743475600` | **Feb 28, 2025 - 23:46 UTC** | **Grouped as February** ❌ | **Grouped as March** ✅ |

---

# How Write One:

```js
// group Company aggreagate
exports.getCompanyCount = async (req, res) => {
    try {
        const result = await File.aggregate([
            {
                $match: { // scan the db to only allow the files with company and date exist (not null)
                    company: { $exists: true, $ne: null },
                    date: { $exists: true, $ne: null }
                }
            },
            {
                $project: { // to clean the data returned by the api(only extract date and company)
                    company: 1,
                date: {                                                                                             ^ end
                        $dateToString: { /* changing the format of ISO date to yyyy-mm-dd */                        |
                            format: "%Y-%m-%d",                                                                     |
                            date: {                                                                                 |
                                $toDate: { /* convert the ts from ms number to ISO date*/                           |    
                                    $multiply: [ /* x 1000, because mongodb are in ms, and $toDate expect ms*/      |
                                        { $toLong: "$date" }, /*string to number*/                                  | start
                                        1000
                                    ]
                                }
                            }
                        }
                    },
                }
            },
            {
                $group: { // group the output by same company name with same date toghether
                    _id: { company: "$company", date: "$date" }, // why i can't put id instead of _id
                    count: { $sum: 1 } // make the sum of how many time the caompany with the same date was repeated
                }
            },
            {
                $project: { // add a second project to organize and control the returning output 
                    company: "$_id.company",
                    date: "$_id.date",
                    count: 1,
                    _id: 0
                }
            },
            {
                $sort: { company: 1, date: 1 } // alphabetic for company and ascending for date
            }
        ]);

        res.json(result);
    } catch (err) {
        console.log(err)
        res.status(500).json({ error: 'Failed to fetch company chart data.' });
    }
}
```

---

# Another One:

```js
// get the after @ in email (Name)
exports.getFromCount = async (req, res) => {
    try {
        const mailCounts = await File.aggregate([
            {
                $match: {
                    from: { $exists: true, $ne: null },
                    ip_address: { $exists: true, $ne: null }
                },
            },
            {
                $project: {
                    senderName: {
                        $arrayElemAt: [
                            { $split: ["$from", "@"] },
                            1
                        ]
                    },
                    ip_address: 1
                }
            },
            {
                $group: {
                    _id: { sender: "$senderName", ip: "$ip_address" },
                    ip_count: { $sum: 1 },
                    // attackers: { $addToSet: "$ip_address" }
                },
            },
            {
                $group: {
                    _id: "$_id.sender",
                    count: { $sum: "$ip_count" },
                    attackers: {
                        $push: {
                            ip: "$_id.ip",
                            count: "$ip_count"
                        }
                    }
                }
            },
            { $sort: { count: -1 } }
        ]);

        res.status(200).json(mailCounts)
    } catch (error) {
        console.error('Error fetching from counts:', error);
        res.status(500).json({ error: 'Internal server error' });
    }

}

```
| **Pipeline Stage** | **Expression** | **Purpose / What it Does** |
|--------------------|----------------|-----------------------------|
| `$project` | `{ senderName: { $arrayElemAt: [ { $split: ["$from", "@"] }, 1 ] }, ip_address: 1 }` | Extracts the domain from the `from` email field (e.g., `example.com` from `user@example.com`) and includes the IP address. |
| `$group` | `{ _id: { sender: "$senderName", ip: "$ip_address" }, ip_count: { $sum: 1 } }` | Groups by **both domain and IP**, counting how many emails came from each domain-IP pair. |
| `$group` | `{ _id: "$_id.sender", count: { $sum: "$ip_count" }, attackers: { $push: { ip: "$_id.ip", count: "$ip_count" } } }` | Regroups by **domain only**, sums up all IP-specific counts to get total per domain, and gathers attacker IPs + their counts into an array. |
| `$sort` | `{ count: -1 }` | Sorts the result in descending order of total email count per domain. |
