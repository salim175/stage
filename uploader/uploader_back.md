# Create File Uploader using Node.js and express

**first i used multer**: Multer is a node.js middleware for handling multipart/form-data, which is primarily used for uploading files.

---

### Schema(file.js):
```js
const mongoose = require('mongoose');

const FileSchema = mongoose.Schema({
    _id: {
        type: String,
        required: true
    },
    contentType: {
        type: String,
        required: true
    },
    from: {
        type: String,
        required: true
    },
    date: {
        type: String,
        required: true
    },
    subject: {
        type: String,
        required: true
    },
    to: {
        type: String,
        required: true
    },
    ip_address: {
        type: String,
        required: true
    },
    body: {
        type: String,
        required: true
    },
    uploadedAt: {
        type: Date,
        default: Date.now
    }
})

const File = mongoose.model('filemodel', FileSchema);
module.exports = File

```

MongoDBcompass will creates a collection name in lowercase with an "s" at the end(if i named it in singular, <ins>ex:</ins> filemodel).

---

### upload_Routes.js:
```js
const router = require('express').Router();
const multer = require('multer');
const { convert } = require('html-to-text');
const moment = require('moment-timezone');

const File = require('../models/file');

const storage = multer.memoryStorage();
const upload = multer({ storage: storage });

router.post('/upload', (req, res) => {
    upload.array('files')(req, res, async function (err) {
        try {
            if (!req.files || req.files.length === 0) return res.status(400).json({ message: 'No file uploaded' });

            const options = {
                wordwrap: 130
            }

            // create object for each file
            const fileDocs = req.files.map(file => {
                // the full mail
                let mailAbuse = file.mimetype.startsWith('text/') ? file.buffer.toString('utf-8') : '[Binary file]';

                // extract the importent text from mails (from, date, subject, to, body)
                const paragraphs = mailAbuse.match(/<blockquote[^>]*>(.*?)<\/blockquote>/gs) || [];

                // convert from html format(the important text) to text 
                const convertedMail = paragraphs.map(p => convert(p, options));

                function extractField(convertedMail, field) {
                    const regex = new RegExp(`${field}:\\s*(.*)`, 'i');
                    const match = convertedMail.find(p => regex.test(p))?.match(regex); // the ? to don't throw error if no match
                    return match ? match[1].trim() : 'unknown';
                }

                function extractFromField(convertedMail) {
                    const regex_from = /From:.*?<([^<>]+)>/i;
                    const match_from = convertedMail.find(p => regex_from.test(p))?.match(regex_from);
                    // console.log('froms: ', match_from) // to see the right index
                    return match_from ? match_from[1].trim() : 'unknown';
                }

                function extractIPAddress(subject) {
                    const regex_IP = /\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b/;
                    const match_Ip = subject.match(regex_IP);
                    // console.log('ips: ', match_Ip)
                    return match_Ip ? match_Ip[0] : 'Unknown';
                }

                const fromField = extractFromField(convertedMail);
                const dateField = extractField(convertedMail, 'Date');
                const subjectField = extractField(convertedMail, 'Subject');
                const toField = extractField(convertedMail, 'To');
                const ipField = extractIPAddress(subjectField);

                // convert the date
                const timestamp = moment.tz(dateField, "DD MMM YYYY HH:mm", "Europe/Paris").unix();

                function extractBody(convertedMail) {
                    const fullText = convertedMail.join("\n");

                    const toIndex = fullText.search(/To:.*?\n/i);
                    const ccIndex = fullText.search(/Cc:.*?\n/i);

                    let startIndex;
                    if (ccIndex !== -1 && ccIndex > toIndex) {
                        startIndex = fullText.indexOf('\n', ccIndex) + 1;
                    } else {
                        startIndex = fullText.indexOf('\n', toIndex) + 1;
                    }

                    if (startIndex === -1) {
                        return 'No body found';
                    }

                    // Extract and clean the body
                    return fullText.slice(startIndex)
                        .replace(/>\s*/g, '')
                        .trim() || 'No body found';
                }

                const bodyField = extractBody(convertedMail);

                // the regex is only to remove the last extension (.txt, .html )
                const new_id = file.originalname.replace(/\.[^.]+$/, '');

                return {
                    _id: new_id,
                    contentType: file.mimetype,
                    from: fromField,
                    date: timestamp,
                    subject: subjectField,
                    to: toField,
                    ip_address: ipField,
                    body: bodyField
                }
            });


            try {
                // skip duplicates -> Using { ordered: false }, MongoDB continues inserting even if some _ids already exist.
                await File.insertMany(fileDocs, { ordered: false });
            } catch (error) {
                if (error.code === 11000) {
                    console.log("Duplicate _id detected. Some emails were skipped.");
                } else {
                    throw error;
                }
            }

            res.status(200).json({ message: 'Files uploaded successfuly', files: req.files.length });
        } catch (error) {
            res.status(500).json({ message: 'Server error', error });
        }
    })
})

module.exports = router;
```

Wrapping ```upload.array('files', 10)``` inside a function lets me catch Multers errors properly.

Because when i put it like this ```router.post('/upload', upload.array('files', 10), async (req, res) => { ... });```, Multer (in this case it's a middleware) processes the request before it reaches your try-catch block (errorMessage: Unexpected end of form )

### 🔹 What Happens When Uploading More Than 10 Files? (If you want to put a limit for files)
- it will look like:
```js
upload.array('files', 10)(req, res, async function (err) {
        try {
            if (err instanceof multer.MulterError && err.code === 'LIMIT_UNEXPECTED_FILE'){
                console.log(err)
                return res.status(400).json({ message: "Maximum upload limit is 10 files." })
            }
            .
            .
            .
```
- If you select **more than 10 files**, Multer will **immediately reject the request** and return an **error(Unexpected field)**, this error have a ``code: 'LIMIT_UNEXPECTED_FILE'``
- **None** of the files will be uploaded or saved to the database.
- The server will respond with:
  ```json
  { "message": "Maximum upload limit is 10 files." }
  ```

---

```.mimetype:``` Checks the type of  file <ins>ex:</ins> text-based (text/plain, text/html, etc.).

A ```Buffer``` in Node.js is a temporary storage for binary data, When you upload a file using Multer, the file is temporarily stored as a binary buffer format in memory instead of being written to disk

Extracts text from the file buffer using ```.toString('utf-8')```, So we can convert the buffer to a string.

file accepted with ```.toString('utf-8')```:  
✅ .txt
✅ .html
✅ .json
✅ .csv
✅ .xml

---

 ```paragraphs``` : Finds all ```blockquote``` HTML elements in the email, and returns an array of matched elements or an empty array ([]) if no matches are found.
```(.*?)```→ Captures everything inside the tag.

it's  return an array because of ```.match()```

---
### 📌 Convert() function

Converts each extracted HTML **blockquote** into plain text using the ```convert()``` function (**html-to-text** package).

it's return an array because of ```.map()```

---

### 📌 `extractField`, `extractFromField`, `extractIPAddress` functions
Extracts a specific email field (`From`, `Date`, `Subject`, `To`, `Ip`,`Body`) from an array of email lines.

- Since `convertedMail` is an **array**, `.find()` **iterates through each element** and checks if it matches the field using regex.
- If a match is found, `.match(regex)` extracts the value after `:` and trims spaces.
- The optional chaining `?.` prevents errors if no match is found.
- Returns `'unknown'` if the field does not exist.

✅ **Because `convertedMail` is an array, `.find()` stops at the first match, making it efficient!** 🚀

---

### 📌 splitMail Explanation
Since ``.search()`` only works on strings, but convertedMail is an array, we use ``.join("\n")`` to convert the array into a single string before applying ```.search()```.

The ``search()`` method returns the index (position) of the first match.

- Handle emails with no Cc field (starting after To)
- Handle emails with a Cc field (starting after Cc)
- Handle emails where Cc might come before To (still works correctly)
- Clean up the body text consistently

---

### 📌 save data in database
1️⃣ MongoDB tries to insert all documents in fileDocs at once using ``insertMany``.

2️⃣ If a document has a duplicate _id, it skips that document but continues inserting others. ✅

3️⃣ Even though some files are inserted successfully, At the end of the insertion process, MongoDB throws error E11000 if it detects duplicates. ❌

4️⃣ The execution jumps to the catch block and logs "Duplicate _id detected. Some emails were skipped."

5️⃣ Final result:

- New (unique) files are inserted into the database.
- Duplicate files are skipped (not inserted).
- Error 11000 is thrown, and the catch block handles it.