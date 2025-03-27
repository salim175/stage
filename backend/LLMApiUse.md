# Use the LLM Api Url:

## 🔸  Option 1 (CommonJS / require) :
```js
const axios = require('axios');
const https = require("https");

const agent = new https.Agent({ rejectUnauthorized: false });

async function extractIPAddress(subject, body) {
    const regex_IP = /\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b/;
    const match_Ip = subject.match(regex_IP);
    // console.log('IP in subject: ', match_Ip) => x.x.x.x

    if (match_Ip) return match_Ip[0];

    try {
        const res = await axios({
            method: 'POST',
            url: 'https://gpttools.apps.prd1.c1.paas.tech.orange/getIPS',
            data: { query: body },
            httpsAgent: agent,
        });

        // console.log('result of LLM API :', res.data.attacker_ip); => x.x.x.x
        if (regex_IP.test(res.data.attacker_ip)) {
            return res.data.attacker_ip
        } else {
            console.warn('the LLM returned an invalid LLM', res.data.attacker_ip);
            return 'Unknown'
        }

    } catch (error) {
        console.error('Error with the LLM API:', error)
        return 'Unknown';
    }
}
```

## 🔸 Option 2 (ES modules / import) :

```js
import axios from 'axios';
import https from 'https';

const agent = new https.Agent({ rejectUnauthorized: false });

export async function extractIPAddress(subject, body) {
    const regex_IP = /\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b/;
    const match_Ip = subject.match(regex_IP);
    // console.log('IP in subject:', match_Ip); => x.x.x.x

    if (match_Ip) return match_Ip[0];

    try {
        const res = await axios.post('xxxxxxxxxxxx', { query: body }, { httpsAgent: agent });

        // console.log('result of LLM API :', res.data.attacker_ip); => x.x.x.x
        if (regex_IP.test(res.data.attacker_ip)) {
            return res.data.attacker_ip
        } else {
            console.warn('the LLM returned an invalid LLM', res.data.attacker_ip);
            return 'Unknown'
        }
    } catch (error) {
        console.error('Axios Error with the LLM API:', error.message);
        return 'Unknown';
    }
}
```

