<template>
    <div :class="['dugard-otp', { error }]">
        <input
            v-for="(digit, index) in digits"
            :key="index"
            ref="inputs"
            :type="inputType"
            :inputmode="inputMode"
            maxlength="1"
            :class="inputClass"
            :value="digit"
            @input="onInput($event, index)"
            @keydown.backspace.prevent="onBackspace(index)"
            @paste.prevent="onPaste($event)"
        >
    </div>
</template>

<script>
export default {
    name: 'DugardOtpInput',
    props: {
        modelValue: {
            type: String,
            default: '',
        },
        numInputs: {
            type: Number,
            default: 4,
        },
        inputClass: {
            type: String,
            default: 'dugard-otp-input',
        },
        inputMode: {
            type: String,
            default: 'numeric', // 'numeric' | 'text'
        },
        autoPasteFromSms: {
            type: Boolean,
            default: false,
        },
        isError: {
            type: Boolean,
            default: false,
        },
    },
    emits: ['update:modelValue', 'onFilled'],
    data() {
        return {
            digits: Array(this.numInputs).fill(''),
            inputType: this.inputMode === 'numeric' ? 'number' : 'text',
            error: this.isError,
            otpAbortController: null,
        };
    },
    watch: {
        modelValue(newVal) {
            if (newVal !== this.digits.join('')) {
                const arr = newVal.split('').slice(0, this.numInputs);
                while (arr.length < this.numInputs) arr.push('');
                this.digits = arr;
            }
        },
        isError(newVal) {
            this.error = newVal;
        },
    },
    mounted() {
        this.focus(0);
        if (this.autoPasteFromSms) this.startOtpListener();
    },
    beforeUnmount() {
        if (this.otpAbortController) {
            this.otpAbortController.abort();
        }
    },
    methods: {
        focus(index) {
            const inputs = this.$refs.inputs;
            if (!inputs || !inputs[index]) {
                const firstEmpty = this.digits.findIndex(ch => ch === '');
                index = firstEmpty !== -1 ? firstEmpty : this.numInputs - 1;
            }
            if (inputs && inputs[index]) inputs[index].focus();
        },
        onInput(e, index) {
            let value = e.target.value;
            if (this.inputMode === 'numeric') value = value.replace(/\D/g, '');
            if (!value) {
                e.target.value = '';
                this.setItem(index, '');
                return;
            }
            value = value.charAt(0);
            e.target.value = value;
            this.setItem(index, value);
            if (index < this.numInputs - 1) {
                this.focus(index + 1);
            } else {
                e.target.blur();
            }
            this.error = false;
        },
        onBackspace(index) {
            if (this.digits[index]) {
                this.setItem(index, '');
            } else if (index > 0) {
                this.focus(index - 1);
                this.setItem(index - 1, '');
            }
        },
        onPaste(e) {
            e.preventDefault();
            const text = (e.clipboardData || window.clipboardData).getData('text') || '';
            this.applyCode(text);
        },
        setItem(index, value) {
            this.digits[index] = value;
            this.emitValue();
            if (this.digits.every(ch => ch !== '')) {
                this.emitFilled();
            }
        },
        emitValue() {
            const val = this.digits.join('');
            this.$emit('update:modelValue', val);
        },
        emitFilled() {
            const val = this.digits.join('');
            this.$emit('onFilled', val);
        },
        startOtpListener() {
            if (!('OTPCredential' in window) || !navigator.credentials) return;
            this.otpAbortController = new AbortController();
            setTimeout(() => {
                if (this.otpAbortController) this.otpAbortController.abort();
            }, 60000);
            navigator.credentials
                .get({
                    otp: { transport: ['sms'] },
                    signal: this.otpAbortController.signal,
                })
                .then(content => {
                    if (content && content.code) {
                        this.applyCode(content.code);
                    }
                });
        },
        applyCode(code) {
            if (!code) return;
            let clean = code.trim();
            if (this.inputMode === 'numeric') {
                clean = clean.replace(/\D/g, '');
            }
            const chars = clean.slice(0, this.numInputs).split('');
            chars.forEach((char, i) => this.setItem(i, char));
            const nextIndex = Math.min(chars.length, this.numInputs - 1);
            this.focus(nextIndex);
        },
    },
};
</script>

<style scoped>
.dugard-otp-input {
    width: 48px;
    height: 48px;
    padding: 12px;
    margin: 0 6px;
    font-size: 1.125rem;
    line-height: 1;
    border-style: solid;
    border-radius: 4px;
    border-width: 3px;
    border-color: #1c2a3a;
    text-align: center;
}
.dugard-otp-input:focus, .dugard-otp-input:focus-visible {
    outline: none;
    border-color: #70abec;
    box-shadow: 0 0 0 1px #70abec;
}
.dugard-otp.error .dugard-otp-input {
    border-color: #FF6A7E;
}
</style>
