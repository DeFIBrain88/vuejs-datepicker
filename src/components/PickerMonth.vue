<template>
  <div :class="[calendarClass, 'vdp-datepicker__calendar']" v-show="showMonthView" :style="calendarStyle" @mousedown.prevent>
    <slot name="beforeCalendarHeader"></slot>
    <header>
      <div class="current-position-header month__year_btn" @click="showYearCalendar" :class="allowedToShowView('year') ? 'up' : ''">{{ pageYearName }}</div>
      <div class="arrow-group">
        <div class="arrow-container prev" :class="{'disabled': isLeftNavDisabled}"  @click="isRtl ? nextYear() : previousYear()">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M9.76757 2.60453C10.1568 2.9938 10.1572 3.62483 9.76834 4.01453L5.89219 7.89914L9.76834 11.7838C10.1572 12.1735 10.1568 12.8045 9.76757 13.1938C9.378 13.5833 8.74638 13.5833 8.3568 13.1938L3.42219 8.25914C3.22337 8.06032 3.22337 7.73797 3.42219 7.53914L8.35681 2.60453C8.74638 2.21495 9.378 2.21495 9.76757 2.60453Z" fill="white"/>
          </svg>
        </div>
        <div class="arrow-container next" @click="isRtl ? previousYear() : nextYear()" :class="{'disabled': isRightNavDisabled}">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M9.76757 2.60453C10.1568 2.9938 10.1572 3.62483 9.76834 4.01453L5.89219 7.89914L9.76834 11.7838C10.1572 12.1735 10.1568 12.8045 9.76757 13.1938C9.378 13.5833 8.74638 13.5833 8.3568 13.1938L3.42219 8.25914C3.22337 8.06032 3.22337 7.73797 3.42219 7.53914L8.35681 2.60453C8.74638 2.21495 9.378 2.21495 9.76757 2.60453Z" fill="white"/>
          </svg>
        </div>
      </div>
    </header>
    <span class="cell month"
      v-for="month in months"
      :key="month.timestamp"
      :class="{'selected': month.isSelected, 'disabled': month.isDisabled}"
      @click.stop="selectMonth(month)">{{ month.month }}</span>
  </div>
</template>
<script>
import { makeDateUtils } from '../utils/DateUtils'
export default {
  props: {
    showMonthView: Boolean,
    selectedDate: Date,
    pageDate: Date,
    pageTimestamp: Number,
    disabledDates: Object,
    calendarClass: [String, Object, Array],
    calendarStyle: Object,
    translation: Object,
    isRtl: Boolean,
    allowedToShowView: Function,
    useUtc: Boolean
  },
  data () {
    const constructedDateUtils = makeDateUtils(this.useUtc)
    return {
      utils: constructedDateUtils
    }
  },
  computed: {
    months () {
      const d = this.pageDate
      let months = []
      // set up a new date object to the beginning of the current 'page'
      let dObj = this.useUtc
        ? new Date(Date.UTC(d.getUTCFullYear(), 0, d.getUTCDate()))
        : new Date(d.getFullYear(), 0, d.getDate(), d.getHours(), d.getMinutes())
      for (let i = 0; i < 12; i++) {
        months.push({
          month: this.utils.getMonthName(i, this.translation.months),
          timestamp: dObj.getTime(),
          isSelected: this.isSelectedMonth(dObj),
          isDisabled: this.isDisabledMonth(dObj)
        })
        this.utils.setMonth(dObj, this.utils.getMonth(dObj) + 1)
      }
      return months
    },
    /**
     * Get year name on current page.
     * @return {String}
     */
    pageYearName () {
      const yearSuffix = this.translation.yearSuffix
      return `${this.utils.getFullYear(this.pageDate)}${yearSuffix}`
    },
    /**
     * Is the left hand navigation disabled
     * @return {Boolean}
     */
    isLeftNavDisabled () {
      return this.isRtl
        ? this.isNextYearDisabled(this.pageTimestamp)
        : this.isPreviousYearDisabled(this.pageTimestamp)
    },
    /**
     * Is the right hand navigation disabled
     * @return {Boolean}
     */
    isRightNavDisabled () {
      return this.isRtl
        ? this.isPreviousYearDisabled(this.pageTimestamp)
        : this.isNextYearDisabled(this.pageTimestamp)
    }
  },
  methods: {
    /**
     * Emits a selectMonth event
     * @param {Object} month
     */
    selectMonth (month) {
      if (month.isDisabled) {
        return false
      }
      this.$emit('selectMonth', month)
    },
    /**
     * Changes the year up or down
     * @param {Number} incrementBy
     */
    changeYear (incrementBy) {
      let date = this.pageDate
      this.utils.setFullYear(date, this.utils.getFullYear(date) + incrementBy)
      this.$emit('changedYear', date)
    },
    /**
     * Decrements the year
     */
    previousYear () {
      if (!this.isPreviousYearDisabled()) {
        this.changeYear(-1)
      }
    },
    /**
     * Checks if the previous year is disabled or not
     * @return {Boolean}
     */
    isPreviousYearDisabled () {
      if (!this.disabledDates || !this.disabledDates.to) {
        return false
      }
      return this.utils.getFullYear(this.disabledDates.to) >= this.utils.getFullYear(this.pageDate)
    },
    /**
     * Increments the year
     */
    nextYear () {
      if (!this.isNextYearDisabled()) {
        this.changeYear(1)
      }
    },
    /**
     * Checks if the next year is disabled or not
     * @return {Boolean}
     */
    isNextYearDisabled () {
      if (!this.disabledDates || !this.disabledDates.from) {
        return false
      }
      return this.utils.getFullYear(this.disabledDates.from) <= this.utils.getFullYear(this.pageDate)
    },
    /**
     * Emits an event that shows the year calendar
     */
    showYearCalendar () {
      this.$emit('showYearCalendar')
    },
    /**
     * Whether the selected date is in this month
     * @param {Date}
     * @return {Boolean}
     */
    isSelectedMonth (date) {
      return (this.selectedDate &&
        this.utils.getFullYear(this.selectedDate) === this.utils.getFullYear(date) &&
        this.utils.getMonth(this.selectedDate) === this.utils.getMonth(date))
    },
    /**
     * Whether a month is disabled
     * @param {Date}
     * @return {Boolean}
     */
    isDisabledMonth (date) {
      let disabledDates = false

      if (typeof this.disabledDates === 'undefined') {
        return false
      }

      if (typeof this.disabledDates.to !== 'undefined' && this.disabledDates.to) {
        if (
          (this.utils.getMonth(date) < this.utils.getMonth(this.disabledDates.to) && this.utils.getFullYear(date) <= this.utils.getFullYear(this.disabledDates.to)) ||
          this.utils.getFullYear(date) < this.utils.getFullYear(this.disabledDates.to)
        ) {
          disabledDates = true
        }
      }
      if (typeof this.disabledDates.from !== 'undefined' && this.disabledDates.from) {
        if (
          (this.utils.getMonth(date) > this.utils.getMonth(this.disabledDates.from) && this.utils.getFullYear(date) >= this.utils.getFullYear(this.disabledDates.from)) ||
          this.utils.getFullYear(date) > this.utils.getFullYear(this.disabledDates.from)
        ) {
          disabledDates = true
        }
      }

      if (typeof this.disabledDates.customPredictor === 'function' && this.disabledDates.customPredictor(date)) {
        disabledDates = true
      }
      return disabledDates
    }
  }
}
// eslint-disable-next-line
;
</script>
