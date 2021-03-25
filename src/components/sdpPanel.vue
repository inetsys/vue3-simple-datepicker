<template>
    <div class="sdp-datepicker--panel--body">
        <button type="button"
            v-for="(item, index) of items"
            :key="index"
            :class="{selected: isEqual(item.value, selected), dimmed: item.dimmed, weekend: item.weekend }"
            @click="select(item.value)"
        >{{ item.label }}</button>
    </div>
</template>

<script>
import { isEqual, isDate } from 'date-fns'

export default {
    emits: ['select'],
    props: {
        items: {
            type: Array,
            default() {
                return []
            },
        },
        selected: {
            type: [String, Number, Date],
            default: null,
        },
    },
    methods: {
        isEqual(valueA, valueB) {
            return isDate(valueA) && isDate(valueB)
                ? isEqual(valueA, valueB)
                : valueA === valueB
        },
        select(value) {
            this.$emit('select', value)
        },
    },
}
</script>
