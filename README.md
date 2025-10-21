# @dugard/vue-otp

A simple and lightweight OTP input component for Vue 3.  
Supports paste, auto-focus navigation between fields, and automatic SMS code detection via Web OTP API.

---

## Installation

```bash
npm install @dugard/vue-otp
````

or

```bash
yarn add @dugard/vue-otp
```

---

## Usage Example

```vue
<template>
  <PPOtpInput
    v-model="code"
    :num-inputs="6"
    :auto-paste-from-sms="true"
    @onFilled="onFilled"
  />
</template>

<script setup>
import { ref } from 'vue'
import { DugardOtpInput } from '@dugard/vue-otp'

const code = ref('')

function onFilled(value) {
  console.log('Code entered completely:', value)
}
</script>
```

---

## Props

| Prop               | Type      | Default     | Description                              |
| ------------------ | --------- | ----------- | ---------------------------------------- |
| `v-model`          | `String`  | `''`        | Code value                               |
| `numInputs`        | `Number`  | `4`         | Number of input fields                   |
| `inputMode`        | `String`  | `'numeric'` | `'numeric'` or `'text'`                  |
| `autoPasteFromSms` | `Boolean` | `false`     | Enables auto-fill from SMS (Web OTP API) |
| `isError`          | `Boolean` | `false`     | Applies red border if true               |

---

## Events

| Event      | Payload  | Description                      |
|------------| -------- | -------------------------------- |
| `onFilled` | `String` | Fires when all inputs are filled |


---

## License

[MIT](./LICENSE) © 2025 Ivan Shkuta (Dugard)

```
