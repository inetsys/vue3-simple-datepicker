<template>
    <h2>1.- Simple use</h2>
    <datepicker v-model="example01.value" :disabled="example01.disabled" />
    <div>
        <label>
            <input type="checkbox" v-model="example01.disabled">
            Disable
        </label>
    </div>
    <pre>Value: {{ example01Formatted }}</pre>

    <h2>2.- Pop-up</h2>
    <button @click="example02Toggle">Open calendar</button>
    <div class="backdrop" v-if="example02.visible" @click="example02Close" />
    <datepicker
        id="example03"
        v-if="example02.visible"
        :model-value="example02.value"
        @update:modelValue="example02OnSelectDate"
    >
        <template #footer>
            <button type="button" class="btn-clear" @click="example02Clear">Clear</button>
            <button type="button" class="btn-today" @click="example02SetToday">Today</button>
            <button type="button" class="btn-close" @click="example02Close">Close</button>
        </template>
    </datepicker>
    <pre>Value: {{ example02Formatted }}</pre>

    <h2>3.- Spanish</h2>
    <datepicker v-model="example03.value" :locale="example03.locale" />
    <pre>Value: {{ example03Formatted }}</pre>
</template>

<script>
import { format } from 'date-fns'
import { es as spanishLocale } from 'date-fns/locale'
import Datepicker from '@/components/Datepicker.vue'

export default {
    name: 'Demo',
    components: {
        Datepicker,
    },
    data() {
        return {
            example01: {
                value: null,
                disabled: false,
            },
            example02: {
                value: null,
                visible: false,
            },
            example03: {
                value: null,
                locale: spanishLocale,
            },
        }
    },
    computed: {
        example01Formatted() {
            return this.example01.value ? format(this.example01.value, 'dd MMM yyyy') : ''
        },
        example02Formatted() {
            return this.example02.value ? format(this.example02.value, 'dd MMM yyyy') : ''
        },
        example03Formatted() {
            return this.example03.value ? format(this.example03.value, 'dd MMM yyyy', { locale: this.example03.locale }) : ''
        },
    },
    methods: {
        example02Toggle() {
            this.example02.visible = !this.example02.visible
        },
        example02OnSelectDate(value) {
            this.example02.value = value
            this.example02Close()
        },
        example02Clear() {
            this.example02.value = null
            this.example02Close()
        },
        example02SetToday() {
            this.example02.value = new Date()
            this.example02Close()
        },
        example02Close() {
            this.example02.visible = false
        },
    },
}
</script>

<style lang="less" scoped>
button {
    padding: 0 10px;
    border-radius: 5px;
}

.btn-clear {
    &:hover {
        background: darkred;
        color: white;
    }
}
.btn-today {
    &:hover {
        background: dodgerblue;
        color: white;
    }
}
.btn-close {
    float: right;

    &:hover {
        background: orange;
        color: white;
    }
}

.backdrop {
    position: fixed;
    z-index: 5;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
}
#example03 {
    position: absolute;
    z-index: 10;
}
</style>
