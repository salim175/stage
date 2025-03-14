# 1️⃣ checkIP Function:
```js
require('dotenv').config();
const axios = require('axios');
const BASE_URL = 'https://api.abuseipdb.com/api/v2/check';

const API_KEY = process.env.ABUSEIPDB_API_KEY;

async function checkIP(ipAddress) {
    try {
        const response = await axios.get(BASE_URL, {
            headers: { 'Key': API_KEY, 'Accept': 'application/json' },
            params: { ipAddress, maxAgeInDays: 90, verbose: true },
        });

        const data = response.data.data;

        return {
            ip: ipAddress,
            abuseConfidenceScore: data.abuseConfidenceScore, // Score out of 100
            detected: data.abuseConfidenceScore > 0, // If detected, it's true
            domain: data.domain || 'N/A',
            country: data.countryName || 'Unknown',
            totalReports: data.totalReports || 0,
            reports: data.reports || [] // Array of past reports
        };
    } catch (error) {
        console.error(`Error checking IP ${ipAddress}:`, error);
        return false;
    }
}

module.exports = { checkIP };
```

1. Sends headers:
    - ``Key``: API key for authentication.
    - ``Accept``: application/json: Requests data in JSON format.

2. Sends parameters:
    - ``ipAddress``: The IP address we want to check.
    - ``maxAgeInDays``: 90: Fetches reports from the last 90 days.
    - ``verbose``: true: Returns detailed reports.

3. Processes API Response:
    - ``abuseConfidenceScore``: A score from 0 to 100 (higher = more suspicious).
    - ``detected``: True if abuseConfidenceScore > 0.
    - ``domain``: The domain associated with the IP (if available).
    - ``countryName``: Country of the IP.
    - ``totalReports``: Number of times the IP was reported.
    - ``reports``: List of reports (who reported, when, and why).

--- 

# 2️⃣ getFilesWithIPCheck Function:
```js
const { checkIP } = require('../services/abuseipservices');
const File = require('../models/file');

exports.getFilesWithIPCheck = async (req, res) => {
    try {
        const files = await File.find({}, { body: 0 }).lean();
        // Set() to filter the files from duplicated ips
        const uniqueIps = [...new Set(files.map(file => file.ip_address).filter(ip => ip))];

        const ipResults = {};
        await Promise.all(uniqueIps.map(async (ip) => {
            const ipData = await checkIP(ip);
            console.log(`Checking IP: ${ip}`, ipData);
            ipResults[ip] = ipData || { detected: false };
        }));

        const enrichedFiles = files.map(file => ({
            ...file,
            ip_check: ipResults[file.ip_address] || { detected: false }
        }));

        res.status(200).json(enrichedFiles);
    } catch (error) {
        console.error('Error fetching files with IP check:', error);
        res.status(500).json({ error: 'Internal server error' });
    }
};
```

## 📌 uniqueIps

🔹 map(file => file.ip_address): Extracts all IPs:["45.33.32.156", "192.168.1.1", "45.33.32.156"] 

🔹 new Set(...): Removes duplicates:["45.33.32.156", "192.168.1.1"] 

🔹 filter(ip => ip): Ensures no empty values.


## 📌 Promise

```js 
if (ipData) {
    ipResults[ip] = ipData; // If ipData is valid, store it
} else {
    ipResults[ip] = { detected: false }; // If ipData is false/null/undefined, store default value
}
```

the **checkIp** will always return value, and when the **API Failed** she will return **false** from the tryCatch in checkIP

## 📌 enrichedFiles:
Combines the original file data with IP check results.