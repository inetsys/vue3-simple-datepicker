<template>
    <div class="sdp-datepicker--panel--body">
        <button type="button"
            v-for="(item, index) of items"
            :key="index"
            :class="{selected: item.selected, dimmed: item.dimmed, weekend: item.weekend }"
            @click="select(item.value)"
        >{{ item.label }}</button>
    </div>
</template>

<script>
export default {
    emits: ['select'],
    props: {
        items: {
            type: Array,
            default() {
                return []
            },
        },
    },
    methods: {
        select(value) {
            this.$emit('select', value)
        },
    },
}
</script>
