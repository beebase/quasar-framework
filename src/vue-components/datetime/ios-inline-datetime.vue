<template>
  <div class="quasar-datetime" :class="['type-' + type, disabled ? 'disabled' : '']">
    <slot></slot>

    <div class="quasar-datetime-content non-selectable">
      <div class="quasar-datetime-inner full-height flex justify-center">
        <div
          class="quasar-datetime-col quasar-datetime-col-month"
          v-if="type === 'date' || type === 'datetime'"
          @touchstart="__dragStart($event, 'month')"
          @touchmove="__dragMove($event, 'month')"
          @touchend="__dragStop($event, 'month')"
        >
          <div v-el:month class="quasar-datetime-col-wrapper" :style="__monthStyle">
            <div
              v-for="monthName in monthsList"
              class="quasar-datetime-item"
              @click="setMonth($index + 1)"
            >
              {{ monthName }}
            </div>
          </div>
        </div>

        <div
          class="quasar-datetime-col quasar-datetime-col-day"
          v-if="type === 'date' || type === 'datetime'"
          @touchstart="__dragStart($event, 'date')"
          @touchmove="__dragMove($event, 'date')"
          @touchend="__dragStop($event, 'date')"
        >
          <div v-el:date class="quasar-datetime-col-wrapper" :style="__dayStyle">
            <div
              v-for="monthDay in daysInMonth"
              class="quasar-datetime-item"
              @click="setDay(monthDay + 1)"
            >
              {{ monthDay + 1 }}
            </div>
          </div>
        </div>

        <div
          class="quasar-datetime-col quasar-datetime-col-year"
          v-if="type === 'date' || type === 'datetime'"
          @touchstart="__dragStart($event, 'year')"
          @touchmove="__dragMove($event, 'year')"
          @touchend="__dragStop($event, 'year')"
        >
          <div v-el:year class="quasar-datetime-col-wrapper" :style="__yearStyle">
            <div
              v-for="n in 101"
              class="quasar-datetime-item"
              @click="setYear(n + 1950)"
            >
              {{ n + 1950 }}
            </div>
          </div>
        </div>

        <div
          v-el
          class="quasar-datetime-col quasar-datetime-col-hour"
          v-if="type === 'time' || type === 'datetime'"
          @touchstart="__dragStart($event, 'hour')"
          @touchmove="__dragMove($event, 'hour')"
          @touchend="__dragStop($event, 'hour')"
        >
          <div v-el:hour class="quasar-datetime-col-wrapper" :style="__hourStyle">
            <div
              v-for="n in 24"
              class="quasar-datetime-item"
              @click="setHour(n)"
            >
              {{ n }}
            </div>
          </div>
        </div>

        <div
          class="quasar-datetime-col-divider"
          v-if="type === 'time' || type === 'datetime'"
        >
          <div class="quasar-datetime-col-wrapper full-height row items-center justify-center">
            <div>:</div>
          </div>
        </div>

        <div
          class="quasar-datetime-col quasar-datetime-col-minute"
          v-if="type === 'time' || type === 'datetime'"
          @touchstart="__dragStart($event, 'minute')"
          @touchmove="__dragMove($event, 'minute')"
          @touchend="__dragStop($event, 'minute')"
        >
          <div v-el:minute class="quasar-datetime-col-wrapper" :style="__minuteStyle">
            <div
              v-for="n in 60"
              class="quasar-datetime-item"
              @click="setMinute(n)"
            >
              {{ __pad(n) }}
            </div>
          </div>
        </div>

        <div class="quasar-datetime-highlight"></div>
      </div>

      <div class="quasar-datetime-mask"></div>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import Utils from '../../utils'

export default {
  props: {
    model: {
      type: String,
      twoWay: true,
      required: true
    },
    type: {
      type: String,
      default: 'date',
      validator (value) {
        return ['date', 'time', 'datetime'].includes(value)
      }
    },
    disabled: {
      type: Boolean,
      default: false,
      coerce: Boolean
    }
  },
  data () {
    return {
      date: moment(this.model),
      monthDragOffset: 0,
      dateDragOffset: 0,
      yearDragOffset: 0,
      hourDragOffset: 0,
      minuteDragOffset: 0,
      monthsList: ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December']
    }
  },
  watch: {
    model (value) {
      this.date = moment(value)
    }
  },
  computed: {
    year () {
      let value = this.date.year()
      this.__updatePositions('year', value)
      this.__updatePositions('date', this.date.date())
      return value
    },
    month () {
      let value = this.date.month()
      this.__updatePositions('month', value)
      this.__updatePositions('date', this.date.date())
      return value + 1
    },
    day () {
      let value = this.date.date()
      this.__updatePositions('date', value)
      return value
    },
    daysInMonth () {
      return this.date.daysInMonth()
    },
    hour () {
      let value = this.date.hour()
      this.__updatePositions('hour', value)
      return value
    },
    minute () {
      let value = this.date.minute()
      this.__updatePositions('minute', value)
      return value
    },

    __monthStyle () {
      return this.__colStyle(82 - (this.month - 1 + this.monthDragOffset) * 36)
    },
    __dayStyle () {
      return this.__colStyle(82 - (this.day + this.dateDragOffset) * 36)
    },
    __yearStyle () {
      return this.__colStyle(82 - (this.year + this.yearDragOffset) * 36)
    },
    __hourStyle () {
      return this.__colStyle(82 - (this.hour + this.hourDragOffset) * 36)
    },
    __minuteStyle () {
      return this.__colStyle(82 - (this.minute + this.minuteDragOffset) * 36)
    }
  },
  methods: {
    /* date */
    setYear (value) {
      if (this.disabled) {
        return
      }
      this.date.year(this.__parseTypeValue('year', value))
      this.__updateModel()
    },
    setMonth (value) {
      if (this.disabled) {
        return
      }
      this.date.month(this.__parseTypeValue('month', value) - 1)
      this.__updateModel()
    },
    setDay (value) {
      if (this.disabled) {
        return
      }
      this.date.date(this.__parseTypeValue('date', value))
      this.__updateModel()
    },

    /* time */
    setHour (value) {
      if (this.disabled) {
        return
      }
      this.date.hour(this.__parseTypeValue('hour', value))
      this.__updateModel()
    },
    setMinute (value) {
      if (this.disabled) {
        return
      }
      this.date.minute(this.__parseTypeValue('minute', value))
      this.__updateModel()
    },

    /* helpers */
    __pad (unit, filler) {
      return (unit < 10 ? filler || '0' : '') + unit
    },
    __parseTypeValue (type, value) {
      if (type === 'month') {
        return Math.max(1, Math.min(12, value))
      }
      if (type === 'date') {
        return Math.max(1, Math.min(this.daysInMonth, value))
      }
      if (type === 'year') {
        return Math.max(1950, Math.min(2050, value))
      }
      if (type === 'hour') {
        return Math.max(0, Math.min(23, value))
      }
      if (type === 'minute') {
        return Math.max(0, Math.min(59, value))
      }
    },
    __updatePositions (type, value) {
      let root = this.$els[type]

      if (!root) {
        return
      }

      let delta = -value + (type === 'year' ? 1950 : (type === 'date' ? 1 : 0))

      ;[].slice.call(root.children).forEach(item => {
        Utils.dom.css(item, this.__itemStyle(value * 36, Math.max(-180, Math.min(180, delta * -18))))
        delta++
      })
    },
    __colStyle (value) {
      return {
        '-webkit-transform': 'translate3d(0,' + value + 'px,0)',
        '-ms-transform': 'translate3d(0,' + value + 'px,0)',
        'transform': 'translate3d(0,' + value + 'px,0)'
      }
    },
    __itemStyle (translation, rotation) {
      return {
        '-webkit-transform': 'translate3d(0, ' + translation + 'px, 0) rotateX(' + rotation + 'deg)',
        '-ms-transform': 'translate3d(0, ' + translation + 'px, 0) rotateX(' + rotation + 'deg)',
        'transform': 'translate3d(0, ' + translation + 'px, 0) rotateX(' + rotation + 'deg)'
      }
    },

    /* common */
    __updateModel () {
      this.model = this.date.toISOString()
    },
    __dragStart (ev, type) {
      if (this.disabled) {
        return
      }

      this.__dragCleanup()
      ;['month', 'date', 'year', 'hour', 'minute'].forEach(type => {
        this[type + 'DragOffset'] = 0
      })
      ev.stopPropagation()
      ev.preventDefault()

      this.dragging = type
      this.__dragPosition = Utils.event.position(ev).top
    },
    __dragMove (ev, type) {
      if (this.dragging !== type || this.disabled) {
        return
      }
      ev.stopPropagation()
      ev.preventDefault()
      this[type + 'DragOffset'] = (this.__dragPosition - Utils.event.position(ev).top) / 36
      this.__updatePositions(type, this.date[type]() + this[type + 'DragOffset'])
    },
    __dragStop (ev, type) {
      if (this.dragging !== type || this.disabled) {
        return
      }
      ev.stopPropagation()
      ev.preventDefault()
      this.dragging = false

      let
        offset = Math.round(this[type + 'DragOffset']),
        newValue = this.__parseTypeValue(type, this[type === 'date' ? 'day' : type] + offset),
        actualType = type === 'date' ? 'day' : type

      if (newValue !== this[actualType]) {
        this['set' + actualType.charAt(0).toUpperCase() + actualType.slice(1)](newValue)
        this[type + 'DragOffset'] = 0
      }
      else {
        this.__updatePositions(type, this.date[type]())
        this.timeout = setTimeout(() => {
          this[type + 'DragOffset'] = 0
        }, 150)
      }
    },
    __dragCleanup () {
      if (this.timeout) {
        clearTimeout(this.timeout)
        this.timeout = null
      }
    }
  },
  compiled () {
    if (this.type === 'date' || this.type === 'datetime') {
      this.__updatePositions('month', this.date.month())
      this.__updatePositions('date', this.date.date())
      this.__updatePositions('year', this.date.year())
    }
    if (this.type === 'time' || this.type === 'datetime') {
      this.__updatePositions('hour', this.date.hour())
      this.__updatePositions('minute', this.date.minute())
    }
  },
  beforeDestroy () {
    this.__dragCleanup()
  }
}
</script>
