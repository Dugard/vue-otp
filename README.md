# @dugard/vue-otp

A simple and lightweight OTP input component for Vue 3.  
Supports paste, auto-focus navigation between fields, and automatic SMS code detection via Web OTP API.

---

## Installation

```bash
npm i @dugard/vue-otp
````

or

```bash
yarn add @dugard/vue-otp
```

---

## Usage Example

```vue
<template>
  <DugardOtpInput
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

## Web OTP API

[Chrome for Developers says:](https://developer.chrome.com/docs/identity/web-apis/web-otp#format)

The message must adhere to the following formatting:

* The message begins with human-readable text that contains a four to ten character alphanumeric string with at least one number leaving the last line for the URL and the OTP.
* The domain part of the URL of the website that invoked the API must be preceded by @.
* The URL must contain a pound sign ('#') followed by the OTP.

For example:

```
Your OTP is: 123456.

@www.example.com #123456
```

---

## License

[MIT](./LICENSE) © 2025 Ivan Shkuta (Dugard)
