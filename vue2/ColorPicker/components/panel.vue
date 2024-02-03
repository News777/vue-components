<template>
  <div
    class="panel"
    :style="{
      backgroundColor: background
    }"
  >
    <div class="panel__white"></div>
    <div class="panel__black"></div>
    <div
      class="panel__cursor"
      :style="{
        top: cursorTop + 'px',
        left: cursorLeft + 'px'
      }"
    >
      <div></div>
    </div>
  </div>
</template>

<script>
import draggable from '../utils/draggable'

export default {
  name: 'panel',

  props: {
    color: {
      required: true
    }
  },

  computed: {
    colorValue() {
      const hue = this.color.get('hue')
      const value = this.color.get('value')
      return { hue, value }
    }
  },

  watch: {
    colorValue() {
      this.update()
    }
  },

  methods: {
    update() {
      const saturation = this.color.get('saturation')
      const value = this.color.get('value')

      const el = this.$el
      let { clientWidth: width, clientHeight: height } = el

      this.cursorLeft = (saturation * width) / 100
      this.cursorTop = ((100 - value) * height) / 100

      this.background = 'hsl(' + this.color.get('hue') + ', 100%, 50%)'
    },

    handleDrag(event) {
      const el = this.$el
      const rect = el.getBoundingClientRect()

      let left = event.clientX - rect.left
      let top = event.clientY - rect.top
      left = Math.max(0, left)
      left = Math.min(left, rect.width)

      top = Math.max(0, top)
      top = Math.min(top, rect.height)

      this.cursorLeft = left
      this.cursorTop = top
      this.color.set({
        saturation: (left / rect.width) * 100,
        value: 100 - (top / rect.height) * 100
      })
    }
  },

  mounted() {
    draggable(this.$el, {
      drag: (event) => {
        this.handleDrag(event)
      },
      end: (event) => {
        this.handleDrag(event)
      }
    })

    this.update()
  },

  data() {
    return {
      cursorTop: 0,
      cursorLeft: 0,
      background: 'hsl(0, 100%, 50%)'
    }
  }
}
</script>
<style lang="scss" scoped>
.panel {
  position: relative;
  width: calc(100% - 20px);
  height: 180px;
  cursor: pointer;
  &__white {
    position: absolute;
    inset: 0;
    background: linear-gradient(to right, #fff, rgba(255, 255, 255, 0));
  }
  &__black {
    position: absolute;
    inset: 0;
    background: linear-gradient(to top, #000, rgba(0, 0, 0, 0));
  }
  &__cursor {
    position: absolute;

    > div {
      cursor: head;
      width: 4px;
      height: 4px;
      box-shadow: 0 0 0 1.5px #fff, inset 0 0 1px 1px rgba(0, 0, 0, 0.3),
        0 0 1px 2px rgba(0, 0, 0, 0.4);
      border-radius: 50%;
      transform: translate(-2px, -2px);
    }
  }
}
</style>
