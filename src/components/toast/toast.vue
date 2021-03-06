<template>
  <div
    class="quasar-toast-container"
    :class="{active: active}"
  >
    <div
      class="quasar-toast row no-wrap items-center non-selectable"
      v-if="stack[0]"
      :style="{color: stack[0].color, background: stack[0].bgColor}"
    >
      <div>
        <i v-if="stack[0].icon">{{ stack[0].icon }}</i>
        <img v-if="stack[0].image" :src="stack[0].image">
      </div>
      <div class="quasar-toast-message auto">
        {{{ stack[0].html }}}
      </div>
      <div class="quasar-toast-button" v-if="stack[0].button && stack[0].button.label">
        <a
          @click="dismiss(stack[0].button.handler)"
          :style="{color: stack[0].button.color}"
        >
          {{ stack[0].button.label }}
        </a>
      </div>
    </div>
  </div>
</template>

<script>
import Events from '../../events'
import Utils from '../../utils'

let
  transitionDuration = 300, // in ms
  displayDuration = 2500 // in ms

function parseOptions (opts, defaults) {
  if (!opts) {
    throw new Error('Missing toast options.')
  }

  let options = Utils.extend(
    true,
    {},
    defaults,
    typeof opts === 'string' ? {html: opts} : opts
  )

  if (!options.html) {
    throw new Error('Missing toast content/HTML.')
  }

  return options
}

export default {
  data () {
    return {
      active: false,
      inTransition: false,
      stack: [],
      timer: null,
      defaults: {
        color: 'white',
        bgColor: '#323232',
        button: {
          color: 'yellow'
        },
        onDismiss () {}
      }
    }
  },
  methods: {
    create (options) {
      this.stack.push(parseOptions(options, this.defaults))

      if (this.active || this.inTransition) {
        return
      }

      this.active = true
      this.inTransition = true

      this.__show()
    },
    __show () {
      Events.trigger('app:toast', this.stack[0].html)

      this.timer = setTimeout(() => {
        if (this.stack.length > 0) {
          this.dismiss()
        }
        else {
          this.inTransition = false
        }
      }, transitionDuration + displayDuration)
    },
    dismiss (done) {
      this.active = false

      if (this.timer) {
        clearTimeout(this.timer)
        this.timer = null
      }

      setTimeout(() => {
        this.stack.shift()

        done && done()

        if (this.stack.length > 0) {
          this.active = true
          this.__show()
          return
        }

        this.inTransition = false
      }, transitionDuration + 50)
    },
    setDefaults (opts) {
      Utils.extend(true, this.defaults, opts)
    }
  }
}
</script>
