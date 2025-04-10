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