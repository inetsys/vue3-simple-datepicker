<template>
    <div class="sdp-datepicker" :class="{ disabled }">
        <div class="sdp-datepicker--header">
            <slot name="header" />
        </div>

        <div class="sdp-datepicker--panel" v-if="activePanel === 'days'">
            <div class="sdp-datepicker--panel--top">
                <button type="button" class="sdp-datepicker--btn--previous-month" @click="previousMonth">◀</button>
                <button type="button" class="sdp-datepicker--btn--month" @click="showMonthsPanel">{{ currentMonth }} {{ currentYear }}</button>
                <button type="button" class="sdp-datepicker--btn--next-month" @click="nextMonth">▶</button>
            </div>
            <div class="sdp-datepicker--week-days">
                <span v-for="(item, index) of weekDays" :key="index" :class="{ weekend: item.weekend }">{{ item.value }}</span>
            </div>
            <sdp-panel class="sdp-datepicker--panel--days" :items="days" @select="selectDate" />
        </div>

        <div class="sdp-datepicker--panel" v-if="activePanel === 'months'">
            <div class="sdp-datepicker--panel--top">
                <button type="button" class="sdp-datepicker--btn--previous-year" @click="previousYear">◀</button>
                <button type="button" class="sdp-datepicker--btn--year" @click="showYearsPanel">{{ currentYear }}</button>
                <button type="button" class="sdp-datepicker--btn--next-year" @click="nextYear">▶</button>
            </div>
            <sdp-panel class="sdp-datepicker--panel--months" :items="months" @select="selectMonth" />
        </div>

        <div class="sdp-datepicker--panel" v-if="activePanel === 'years'">
            <div class="sdp-datepicker--panel--top">
                <button type="button" class="sdp-datepicker--btn--previous-years" @click="previousYears">▲</button>
            </div>
            <sdp-panel class="sdp-datepicker--panel--years" :items="years" @select="selectYear" />
            <div class="sdp-datepicker--panel--top">
                <button type="button" class="sdp-datepicker--btn--next-years" @click="nextYears">▼</button>
            </div>
        </div>

        <div class="sdp-datepicker--footer">
            <slot name="footer" />
        </div>
    </div>
</template>

<script>
import {
    format,
    isEqual,
    isSameMonth,
    isWeekend,
    isValid,
    eachDayOfInterval,
    endOfWeek,
    endOfMonth,
    startOfWeek,
    startOfMonth,
    addMonths,
    subMonths,
    addYears,
    subYears,
    getDate,
    getMonth,
    getYear,
    setDate,
    setMonth,
    setYear,
} from 'date-fns'
import { enGB as defaultLocale } from 'date-fns/locale'

import sdpPanel from './sdpPanel.vue'

export default {
    components: { sdpPanel },
    props: {
        disabled: {
            type: Boolean,
            default: false,
        },
        modelValue: {
            type: Date,
            default: null,
        },
        locale: {
            type: Object,
            default: defaultLocale,
        },
    },
    emits: ['update:modelValue'],
    data() {
        const months = []
        for (let index = 0; index < 12; index++) {
            months.push({
                label: this.locale.localize.month(index, {}),
                value: index,
            })
        }
        const weekDays = []
        for (let index = 0; index < 7; index++) {
            const dayCode = (index + this.locale.options.weekStartsOn) % 7
            weekDays.push({
                value: this.locale.localize.day(dayCode, { width: 'narrow' }),
                weekend: dayCode === 0 || dayCode === 6,
            })
        }

        return {
            activePanel: 'days',
            currentDate: new Date(),
            days: [],
            months,
            weekDays,
            years: [],
        }
    },
    computed: {
        isModelValueValid() {
            return this.modelValue ? isValid(this.modelValue) : true
        },
        currentYear() {
            return getYear(this.currentDate)
        },
        currentMonth() {
            return format(this.currentDate, 'MMMM', { locale: this.locale })
        },
        currentDay() {
            return getDate(this.currentDate)
        },
    },
    methods: {
        showCalendar() {
            this.showDaysPanel()
        },

        selectYear(year) {
            this.currentDate = setYear(this.currentDate, year)
            this.showMonthsPanel()
        },
        selectMonth(month) {
            this.currentDate = setMonth(this.currentDate, month)
            this.showDaysPanel()
        },
        selectDate(date) {
            this.currentDate = setDate(date, 15)
            if (!this.disabled) {
                this.$emit('update:modelValue', date)
            }
        },

        showDaysPanel() {
            const vm = this
            this.activePanel = 'days'

            this.days = eachDayOfInterval({
                start: startOfWeek(startOfMonth(this.currentDate), {
                    weekStartsOn: this.locale.options.weekStartsOn,
                }),
                end: endOfWeek(endOfMonth(this.currentDate), {
                    weekStartsOn: this.locale.options.weekStartsOn,
                }),
            }).map(day => {
                return {
                    label: format(day, 'd'),
                    value: day,
                    selected: vm.modelValue && isEqual(day, vm.modelValue),
                    dimmed: !isSameMonth(day, vm.currentDate),
                    weekend: isWeekend(day),
                }
            })
        },
        showMonthsPanel() {
            this.activePanel = 'months'

            this.months.forEach(month => {
                month.selected = this.modelValue
                    ? getYear(this.modelValue) === getYear(this.currentDate) && getMonth(this.modelValue) === month.value
                    : false
            })
        },
        showYearsPanel() {
            this.activePanel = 'years'
            const years = []
            for (let year = this.currentYear - 4; year <= this.currentYear + 4; year++) {
                years.push({
                    label: year,
                    value: year,
                    selected: this.modelValue ? getYear(this.modelValue) === year : false,
                })
            }
            this.years = years
        },

        previousMonth() {
            this.currentDate = subMonths(this.currentDate, 1)
            this.showDaysPanel()
        },
        nextMonth() {
            this.currentDate = addMonths(this.currentDate, 1)
            this.showDaysPanel()
        },

        previousYear() {
            this.currentDate = subYears(this.currentDate, 1)
            this.showMonthsPanel()
        },
        nextYear() {
            this.currentDate = addYears(this.currentDate, 1)
            this.showMonthsPanel()
        },

        previousYears() {
            this.currentDate = subYears(this.currentDate, 9)
            this.showYearsPanel()
        },
        nextYears() {
            this.currentDate = addYears(this.currentDate, 9)
            this.showYearsPanel()
        },
    },
    watch: {
        modelValue(newValue, oldValue) {
            this.currentDate = setDate(this.modelValue && this.isModelValueValid ? this.modelValue : new Date(), 15)
            this.showCalendar()
        },
    },
    created() {
        this.currentDate = setDate(this.modelValue && this.isModelValueValid ? this.modelValue : new Date(), 15)
        this.showCalendar()
    },
}
</script>

<style lang="less">
@datepicker-width: 280px;
@datepicker-padding: 3px;

.sdp-datepicker {
    background: white;
    box-shadow: 0 0 3px rgba(0, 0, 0, .6);
    width: @datepicker-width;
    max-width: 100%;
    padding: @datepicker-padding;

    &, & * {
        box-sizing: border-box;
    }

    button {
        background-color: transparent;
        border: none;
        box-shadow: none;
        outline: none;
        line-height: 34px;
        border-radius: 3px;
        cursor: pointer;
        transition: background-color .3s, color .3s;
        padding: 0;
        margin: 0;
        color: #333;

        .disabled& {
            cursor: default;
        }

        &:hover {
            background-color: ghostwhite;
        }

        &:focus {
            box-shadow: 0 0 2px silver;
        }

        &.dimmed {
            opacity: .7;
        }
        &.weekend {
            color: darkslategrey;
        }
        &.selected {
            &, &:hover {
                background-color: darkgreen;
                color: white;
            }
        }
    }

    &--panel {
        &--top {
            display: flex;
            flex-flow: row wrap;

            button {
                flex: 1;
                min-width: 34px;
            }
        }

        &--body {
            display: flex;
            flex-flow: row wrap;

            button {
                flex: 1 1;
            }
        }

        &--days {
            justify-content: space-between;
            width: 235px;
            margin: 0 auto;

            button {
                flex: 0 0 30px;
                border-radius: 50%;
                line-height: 30px;
            }
        }
        &--months {
            button {
                flex: 0 1 ((@datepicker-width - 8) / 3);
            }
        }
        &--years {
            button {
                flex: 0 1 ((@datepicker-width - 8) / 3);
            }
        }
    }

    &--week-days {
        display: flex;
        flex-flow: row wrap;
        justify-content: space-between;
        width: 235px;
        margin: 5px auto;
        font-weight: 700;

        span {
            flex: 0 0 30px;
            text-align: center;
            text-transform: uppercase;
        }
    }

    &--header {
        margin: 0 0 5px;

        &:empty {
            display: none;
        }
    }

    &--footer {
        margin: 5px 0 0;

        &:empty {
            display: none;
        }
    }

    button&--btn {
        &--previous-month,
        &--next-month,
        &--previous-year,
        &--next-year {
            flex: 0 0 20px;
        }

        &--month,
        &--year {
            font-weight: 700;
            text-transform: uppercase;
            font-size: 1em;
        }
    }
}
</style>
