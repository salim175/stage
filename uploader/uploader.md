# Create File Uploader using Node.js and express

**first i used multer**: Multer is a node.js middleware for handling multipart/form-data, which is primarily used for uploading files.

---

### Schema(file.js):
```js
const mongoose = require('mongoose');

const FileSchema = mongoose.Schema({
    name: {
        type: String,
        required: true
    },
    contentType: {
        type: String,
        required: true
    },
    text: {
        type: String,
        required: true
    },
    uploadedAt: {
        type: Date,
        default: Date.now
    }
})

const File = mongoose.model('filemodels', FileSchema);
module.exports = File
```

MongoDBcompass will creates a collection name in lowercase with an "s" at the end(if i named it in singular, <ins>ex:</ins> filemodels), displaying(name, contentType, text and uploadedAt) 

---

### upload_Routes.js:
```js
const router = require('express').Router();
const multer = require('multer');
const File = require('../models/file');

const storage = multer.memoryStorage();
const upload = multer({ storage: storage });

router.post('/upload', (req, res) => {
    upload.array('files', 10)(req, res, async function (err) {
        try {
            console.log(err)
            if (err instanceof multer.MulterError && err.code === 'LIMIT_UNEXPECTED_FILE')
                return res.status(400).json({ message: "Maximum upload limit is 10 files." })

            if (!req.files || req.files.length === 0) return res.status(400).json({ message: 'No file uploaded' });

            // create object for each file
            const fileDocs = req.files.map(file => {
                let extractedText = file.mimetype.startsWith('text/') ? file.buffer.toString('utf-8') : '[Binary file]';
                // let extractedText = '';
                // if (file.mimetype.startsWith('text/'))
                //     extractedText = file.buffer.toString('utf-8');
                // else
                //     extractedText = '[Binary file: Content not extracted]';

                return {
                    name: file.originalname,
                    contentType: file.mimetype,
                    text: extractedText
                }
            });

            // save to mongoDB
            await File.insertMany(fileDocs);

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

### 🔹 What Happens When Uploading More Than 10 Files?
- If you select **more than 10 files**, Multer will **immediately reject the request** and return an **error(Unexpected field)**, this error have a ``code: 'LIMIT_UNEXPECTED_FILE'``
- **None** of the files will be uploaded or saved to the database.
- The server will respond with:
  ```json
  { "message": "Maximum upload limit is 10 files." }
  ```

```mime.type:``` Checks the type of  file <ins>ex:</ins> text-based (text/plain, text/html, etc.).

A ```Buffer``` in Node.js is a temporary storage for binary data, When you upload a file using Multer, the file is temporarily stored as a binary buffer format in memory instead of being written to disk

Extracts text from the file buffer using ```.toString('utf-8')```, So we can convert the buffer to a string.

file accepted with ```.toString('utf-8')```:  
✅ .txt
✅ .html
✅ .json
✅ .csv
✅ .xml
