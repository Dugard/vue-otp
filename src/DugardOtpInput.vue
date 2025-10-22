<template>
    <div :class="['dugard-otp', { error }]">
        <input
            v-model="hiddenInputValue"
            ref="hiddenInput"
            type="text"
            :inputmode="inputMode"
            autocomplete="one-time-code"
            class="dugard-otp-hidden"
            @keydown.left.prevent="onKeydownLeft"
            @keydown.right.prevent="onKeydownRight"
            @keydown.enter.prevent="onKyeDownEnter"
            @keydown.backspace.prevent="onKeydownBackspace"
            @input="onInput"
            @paste.prevent="onPaste($event)"
            @blur="onHiddenBlur"
        />
        <div
            v-for="(ch, index) in resultingSymbols"
            :key="index"
            :class="[inputClass, { active: activeIndex === index }]"
            @click="onClick(index)"
            role="button"
            tabindex="0"
            @keydown.prevent.stop
        >
            {{ ch }}
        </div>
    </div>
</template>

<script>
export default {
    name: 'DugardOtpInput',
    props: {
        modelValue: { type: String, default: '' },
        numInputs: { type: Number, default: 4 },
        inputClass: { type: String, default: 'dugard-otp-input' },
        inputMode: { type: String, default: 'numeric' }, // 'numeric' | 'text'
        autoPasteFromSms: { type: Boolean, default: false },
        isError: { type: Boolean, default: false },
    },
    emits: ['update:modelValue', 'onFilled'],
    data() {
        return {
            resultingSymbols: Array(this.numInputs).fill(''),
            hiddenInputValue: '',
            activeIndex: 0,
            error: this.isError,
            otpAbortController: null,
        };
    },
    watch: {
        modelValue(newVal) {
            if (newVal === this.resultingSymbols.join('')) return;
            const clean = this.sanitize(newVal).slice(0, this.numInputs);
            const arr = Array(this.numInputs).fill('');
            for (let i = 0; i < clean.length; i++) arr[i] = clean[i];
            this.resultingSymbols = arr;
            this.syncHidden(false);
            this.focus();
        },
        isError(v) {
            this.error = v;
        },
    },
    mounted() {
        this.focusHidden();
        if (this.autoPasteFromSms) this.startOtpListener();
    },
    beforeUnmount() {
        this.otpAbortController?.abort?.();
    },
    methods: {
        focus(index) {
            if (index < 0) {
                this.activeIndex = 0;
            } else if (index >= this.numInputs) {
                this.activeIndex = this.numInputs;
            } else if (!this.resultingSymbols || !this.resultingSymbols[index]) {
                const firstEmpty = this.resultingSymbols.findIndex(ch => ch === '');
                this.activeIndex = firstEmpty !== -1 ? firstEmpty : this.numInputs - 1;
            } else {
                this.activeIndex = index;
            }
            this.focusHidden();
        },
        sanitize(str) {
            if (!str) return '';
            return this.inputMode === 'numeric' ? String(str).replace(/\D/g, '') : String(str);
        },
        syncHidden(emitValue = true) {
            this.hiddenInputValue = this.resultingSymbols.join('');
            if (emitValue) this.emitValue();
        },
        focusHidden() {
            this.$nextTick(() => this.$refs.hiddenInput?.focus());
        },
        emitValue() {
            this.$emit('update:modelValue', this.resultingSymbols.join(''));
            this.focusHidden();
            this.tryEmitFilled();
        },
        tryEmitFilled() {
            if (this.resultingSymbols.every(ch => ch !== '') && this.activeIndex >= this.numInputs - 1) {
                this.$emit('onFilled', this.resultingSymbols.join(''));
                this.activeIndex = this.numInputs;
            }
        },
        removeAt(index) {
            for (let i = index; i < this.resultingSymbols.length - 1; i++) {
                this.resultingSymbols[i] = this.resultingSymbols[i + 1];
            }
            this.resultingSymbols[this.resultingSymbols.length - 1] = '';
        },
        onClick(index) {
            this.error = false;
            this.focus(index);
        },
        onKeydownLeft() {
            this.error = false;
            this.focus(this.activeIndex - 1);
        },
        onKeydownRight() {
            this.error = false;
            this.focus(this.activeIndex + 1);
        },
        onKyeDownEnter() {
            this.tryEmitFilled();
        },
        onKeydownBackspace() {
            this.error = false;
            if (this.resultingSymbols[this.activeIndex] !== '' && typeof this.resultingSymbols[this.activeIndex] !== 'undefined') {
                this.removeAt(this.activeIndex);
            } else if (this.activeIndex > 0) {
                this.focus(this.activeIndex - 1);
                this.removeAt(this.activeIndex);
            }
            this.syncHidden();
        },
        onInput(e) {
            this.error = false;
            if (e.data.length === 1) {
                const char = this.sanitize(e.data).slice(0, 1);
                if (!char) {
                    e.preventDefault();
                    return;
                }
                e.preventDefault();
                this.resultingSymbols[this.activeIndex] = char;
                this.focus(this.activeIndex + 1);
                this.syncHidden();
            }
        },
        onPaste(e) {
            const text = (e.clipboardData || window.clipboardData).getData('text') || '';
            this.applyCode(text);
        },
        startOtpListener() {
            if (!('OTPCredential' in window) || !navigator.credentials) return;
            this.otpAbortController = new AbortController();
            setTimeout(() => this.otpAbortController?.abort(), 180000);
            navigator.credentials
                .get({
                    otp: { transport: ['sms'] },
                    signal: this.otpAbortController.signal,
                })
                .then(content => {
                    if (content?.code) this.applyCode(content.code);
                })
                .catch(() => {});
        },
        applyCode(raw) {
            const clean = this.sanitize(raw).slice(0, this.numInputs);
            const arr = Array(this.numInputs).fill('');
            for (let i = 0; i < clean.length; i++) arr[i] = clean[i];
            this.resultingSymbols = arr;
            this.focus(this.numInputs);
            this.syncHidden();
        },
        onHiddenBlur() {
            const newTarget = document.activeElement;
            const wrapper = this.$el;
            if (!wrapper.contains(newTarget)) {
                this.activeIndex = -1;
                return;
            }
            const isBox = newTarget.closest?.(`.${this.inputClass}`);
            if (isBox) {
                this.focusHidden();
            }
        },
    },
};
</script>

<style scoped>
.dugard-otp {
    display: flex;
    gap: 8px;
}
.dugard-otp-hidden {
    position: absolute;
    opacity: 0;
    pointer-events: none;
    height: 0;
    width: 0;
}
.dugard-otp-input {
    position: relative;
    width: 48px;
    height: 48px;
    border: 3px solid #1c2a3a;
    border-radius: 4px;
    text-align: center;
    font-size: 1.125rem;
    line-height: 48px;
    user-select: none;
    cursor: text;
    transition: border-color 0.2s ease;
}
.dugard-otp-input.active {
    border-color: #70abec;
    box-shadow: 0 0 0 1px #70abec;
}
.dugard-otp-input.active::after {
    content: '';
    position: absolute;
    bottom: 8px;
    left: 50%;
    transform: translateX(-50%);
    width: 18px;
    height: 2px;
    background: #70abec;
    border-radius: 1px;
    animation: otp-caret-blink 1s steps(2, start) infinite;
}
@keyframes otp-caret-blink {
    0%, 49% { opacity: 1; }
    50%, 100% { opacity: 0; }
}
.dugard-otp.error .dugard-otp-input {
    border-color: #FF6A7E;
}
</style>
