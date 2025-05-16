# 🔨 thsi page is for some info

### 🔁 ``ref():`` its a function in Vue.js
- when i changes the value of a ref, Vue detects the change
- components that use this value are automatically updated.

- In ``<script setup>``:
    - i must use ``.value`` to get or set the actual value inside a ref.

- In ``<template>``:
    - Vue automatically unwraps ``.value``, so i just use the variable directly.

### 🔁 ``computed(): ``  

- when any of the dependencies inside the computed change, Vue automatically updates the result

- In ``<script setup>:``
    - i use it like a function: ``const result = computed(() => something)``
    - Access the value with ``.value`` (ex: ``result.valu``e)

- In ``<template>:``
    - Vue unwraps ``.value`` automatically → just use ``{{ result }}``

### 🔁 ``onMounted():``

- runs once, right after the component is mounted in the DOM
- In ``<script setup>:``
    - i call ``onMounted(() => { ... })`` and put the code to run once inside the function

### 🔁 ``watch():``
- listens to changes in a reactive value
- runs a callback every time that value changes

- In ``<script setup>:``
    - i use it ``like: watch(source, (newVal, oldVal) => { ... })``
    - Great for running side-effects like API calls, logging, etc.

### 🔁 ``Spread operator (...):``
```js
let array1 = ['h', 'e', 'l', 'l', 'o'];
let array2 = [...array1];
console.log(array2);
['h', 'e', 'l', 'l', 'o'] // output
```

The ``array2`` has the elements of ``array1`` copied into it. Any changes made to array1 will not be reflected in array2 and vice versa.

If the simple assignment operator had been used, then array2 would have been assigned a reference to array1. The changes made in one array would be reflected in the other array, which in most cases is undesirable.


# ❌ ERRORs:
You have this setup:
```ts
interface Upload {
  _id: string
  uploadedAt: number
}
const lastUpload = ref<Upload | null>(null)
```

## ❗️ TypeScript Errors You Might See:
### 1. ``Type 'string | undefined' is not assignable to type 'string'``
Example:
```ts
fivecards.value[3].text = lastUpload.value?._id // ❌
```
🔍 **Why**:
``lastUpload.value`` might be ``null`` or ``undefined``,  and the ``.text`` property is strictly typed as ``string``, so use the fallback to ensure a ``string``.

✅ Fix:
```ts
fivecards.value[3].text = lastUpload.value?._id || 'N/A'
```
### 2. ``Object is possibly 'null'``

Example:
```ts
const id = lastUpload.value._id // ❌
```
🔍 **Why**:
because this line assumes lastIpload.value is always not null but lastUpload can be ``null(ref<Upload | null>(null))``

✅ Fix:
```ts
if (lastUpload.value) {
  const id = lastUpload.value._id
}
```

### 3. ``Property '_id' does not exist on type 'unknown'``
Example:
```ts
const lastUpload = ref() // ❌ inferred as Ref<unknown>
lastUpload.value._id
```
✅ Fix:
```ts
const lastUpload = ref<Upload | null>(null)
```
### 4. ``Type 'null' is not assignable to type 'Upload'``
Example:
```ts
const lastUpload = ref<Upload>(null) // ❌
```
🔍 **Why**:
You told TS it must always be ``Upload``, but it's starting as ``null``.

✅ Fix:
```ts
const lastUpload = ref<Upload | null>(null)
```
### 5. ``Cannot read properties of null (reading '_id') (Runtime, not TS)``
Example:
```html
<span>{{ lastUpload.value._id }}</span> <!-- ❌ -->
```
✅ Fix:
```html
<span v-if="lastUpload?.value">{{ lastUpload.value._id }}</span>
```