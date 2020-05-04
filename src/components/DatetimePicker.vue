<template>
    <v-dialog v-model="display" :width="dialogWidth">
        <template v-slot:activator="{ on }">
            <v-text-field
                v-bind="textFieldProps"
                :disabled="disabled"
                :loading="loading"
                :label="label"
                :value="formattedDatetime"
                v-on="on"
                readonly
            >
                <template v-slot:progress>
                    <slot name="progress">
                        <v-progress-linear color="primary" indeterminate absolute height="2"></v-progress-linear>
                    </slot>
                </template>
            </v-text-field>
        </template>

        <v-card>
            <v-card-text class="px-0 py-0">
                <v-tabs fixed-tabs v-model="activeTab">
                    <v-tab key="calendar">
                        <slot name="dateIcon">
                            <v-icon>event</v-icon>
                        </slot>
                    </v-tab>
                    <v-tab key="timer" :disabled="dateSelected">
                        <slot name="timeIcon">
                            <v-icon>access_time</v-icon>
                        </slot>
                    </v-tab>
                    <v-tab-item key="calendar">
                        <v-date-picker
                            v-model="date"
                            v-bind="datePickerProps"
                            :min="minDate"
                            :max="maxDate"
                            @input="showTimePicker"
                            full-width
                        />
                    </v-tab-item>
                    <v-tab-item key="timer">
                        <v-time-picker
                            ref="timer"
                            class="v-time-picker-custom"
                            v-model="time"
                            v-bind="timePickerProps"
                            :min="minTime"
                            :max="maxTime"
                            full-width
                        />
                    </v-tab-item>
                </v-tabs>
            </v-card-text>
            <v-card-actions>
                <v-spacer></v-spacer>
                <slot name="actions" :parent="this">
                    <v-btn color="grey lighten-1" text @click.native="clearHandler">{{ clearText }}</v-btn>
                    <v-btn color="green darken-1" text @click="okHandler">{{ okText }}</v-btn>
                </slot>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
    import { format, parse, formatISO } from 'date-fns'
    const DEFAULT_DATE = ''
    const DEFAULT_TIME = ''
    const DEFAULT_DATE_FORMAT = 'yyyy-MM-dd'
    const DEFAULT_TIME_FORMAT = 'HH:mm:ss'
    const DEFAULT_DIALOG_WIDTH = 340
    const DEFAULT_CLEAR_TEXT = 'CLEAR'
    const DEFAULT_OK_TEXT = 'OK'
    export default {
        name: 'v-datetime-picker',
        model: {
            prop: 'datetime',
            event: 'input'
        },
        props: {
            datetime: {
                type: [Date, String],
                default: null
            },
            disabled: {
                type: Boolean
            },
            loading: {
                type: Boolean
            },
            label: {
                type: String,
                default: ''
            },
            dialogWidth: {
                type: Number,
                default: DEFAULT_DIALOG_WIDTH
            },
            dateFormat: {
                type: String,
                default: DEFAULT_DATE_FORMAT
            },
            timeFormat: {
                type: String,
                default: DEFAULT_TIME_FORMAT
            },
            clearText: {
                type: String,
                default: DEFAULT_CLEAR_TEXT
            },
            okText: {
                type: String,
                default: DEFAULT_OK_TEXT
            },
            textFieldProps: {
                type: Object
            },
            datePickerProps: {
                type: Object
            },
            timePickerProps: {
                type: Object
            },
            datetimeMax: {
                type: [Date, String],
                default: null,
            },
            datetimeMin: {
                type: [Date, String],
                default: null,
            },
        },
        data() {
            return {
                display: false,
                activeTab: 0,
                date: DEFAULT_DATE,
                time: DEFAULT_TIME
            }
        },
        mounted() {
            this.init()
        },
        computed: {
            dateTimeFormat() {
                return this.dateFormat + ' ' + this.timeFormat
            },
            formattedDatetime() {
                return this.selectedDatetime ? format(this.selectedDatetime, this.dateTimeFormat) : ''
            },
            selectedDatetime() {
                if (this.date && this.time) {
                    let datetimeString = this.date + ' ' + this.time
                    if (this.time.length === 5) {
                        datetimeString += ':00'
                    }
                    return new Date(datetimeString)
                } else {
                    return null
                }
            },
            dateSelected() {
                return !this.date
            },
            dtMax() {
                if(!!this.datetimeMax) {
                    return (this.datetimeMax instanceof Date) ? formatISO(this.datetimeMax) : new Date(this.datetimeMax).toISOString();
                }
                return null;
            },
            dtMin() {
                if(!!this.datetimeMin){
                    return (this.datetimeMin instanceof Date) ? formatISO(this.datetimeMin) : new Date(this.datetimeMin).toISOString();
                }
                return null;
            },
            maxDate() {
                if (!!this.dtMax) {
                    return this.dtMax.substr(0, 10);
                }
                return null;
            },
            minDate() {
                if (!!this.dtMin) {
                    return this.dtMin.substr(0, 10);
                }
                return null;
            },
            maxTime() {
                if (!this.dtMax || !this.date) return null;
                let eqDay = formatISO(new Date(this.date)).substr(0, 10) === this.maxDate;
                return eqDay ? this.dtMax.substr(11, 8) : null;
            },
            minTime() {
                if (!this.dtMin || !this.date) return null;
                let eqDay = formatISO(new Date(this.date)).substr(0, 10) === this.minDate;
                return eqDay ? this.dtMin.substr(11, 8) : null;
            },
        },
        methods: {
            init() {
                if (!this.datetime) {
                    this.date = DEFAULT_DATE
                    this.time = DEFAULT_TIME
                    return
                }
                let initDateTime
                if (this.datetime instanceof Date) {
                    initDateTime = this.datetime
                } else if (typeof this.datetime === 'string' || this.datetime instanceof String) {
                    // see https://stackoverflow.com/a/9436948
                    initDateTime = new Date(this.datetime)
                }
                this.date = format(initDateTime, DEFAULT_DATE_FORMAT)
                this.time = format(initDateTime, DEFAULT_TIME_FORMAT)
            },
            okHandler() {
                this.resetPicker()
                this.$emit('input', formatISO(this.selectedDatetime))
            },
            clearHandler() {
                this.resetPicker()
                this.date = DEFAULT_DATE
                this.time = DEFAULT_TIME
                this.$emit('input', null)
            },
            resetPicker() {
                this.display = false
                this.activeTab = 0
                if (this.$refs.timer) {
                    this.$refs.timer.selectingHour = true
                }
            },
            showTimePicker() {
                this.activeTab = 1
            }
        },
        watch: {
            datetime: function() {
                this.init()
            },
            date: function (val, oldVal) {
                if(!this.dateSelected) this.time = this.minTime || this.maxTime;
            }
        }
    }
</script>

<style scoped>
    .v-picker.v-card{
        box-shadow: none;
        border-radius: 0;
    }
    .v-picker ::v-deep .v-time-picker-title__time .v-picker__title__btn,
    .v-picker ::v-deep .v-time-picker-title__time span{
        height: 56px;
    }
    .v-picker ::v-deep .v-time-picker-title__time{
        margin: auto;
    }
</style>
