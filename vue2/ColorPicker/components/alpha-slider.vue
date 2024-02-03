<template>
  <div class="color-alpha-slider" :class="{ 'is-vertical': vertical }">
    <div
      class="color-alpha-slider__bar"
      @click="handleClick"
      ref="bar"
      :style="{
        background: background
      }"
    ></div>
    <div
      class="color-alpha-slider__thumb"
      ref="thumb"
      :style="{
        left: thumbLeft + 'px',
        top: thumbTop + 'px'
      }"
    ></div>
  </div>
</template>

<script>
import draggable from '../utils/draggable'

export default {
  name: 'ColorAlphaSlider',

  props: {
    color: {
      required: true
    },
    vertical: Boolean
  },

  watch: {
    'color._alpha'() {
      this.update()
    },

    'color.value'() {
      this.update()
    }
  },

  methods: {
    handleClick(event) {
      const thumb = this.$refs.thumb
      const target = event.target
      if (target !== thumb) {
        this.handleDrag(event)
      }
    },

    handleDrag(event) {
      const rect = this.$el.getBoundingClientRect()
      console.log(rect)
      const { thumb } = this.$refs
      if (!this.vertical) {
        let left = event.clientX - rect.left
        left = Math.max(thumb.offsetWidth / 2, left)
        left = Math.min(left, rect.width - thumb.offsetWidth / 2)
        this.color.set(
          'alpha',
          Math.round(((left - thumb.offsetWidth / 2) / (rect.width - thumb.offsetWidth)) * 100)
        )
      } else {
        let top = event.clientY - rect.top
        top = Math.max(thumb.offsetHeight / 2, top)
        top = Math.min(top, rect.height - thumb.offsetHeight / 2)

        this.color.set(
          'alpha',
          Math.round(((top - thumb.offsetHeight / 2) / (rect.height - thumb.offsetHeight)) * 100)
        )
      }
    },

    getThumbLeft() {
      if (this.vertical) return 0
      const el = this.$el
      const alpha = this.color._alpha

      if (!el) return 0
      const thumb = this.$refs.thumb
      return Math.round((alpha * (el.offsetWidth - thumb.offsetWidth / 2)) / 100)
    },

    getThumbTop() {
      if (!this.vertical) return 0
      const el = this.$el
      const alpha = this.color._alpha

      if (!el) return 0
      const thumb = this.$refs.thumb
      return Math.round((alpha * (el.offsetHeight - thumb.offsetHeight / 2)) / 100)
    },

    getBackground() {
      if (this.color && this.color.value) {
        const { r, g, b } = this.color.toRgb()
        return `linear-gradient(to right, rgba(${r}, ${g}, ${b}, 0) 0%, rgba(${r}, ${g}, ${b}, 1) 100%)`
      }
      return null
    },

    update() {
      this.thumbLeft = this.getThumbLeft()
      this.thumbTop = this.getThumbTop()
      this.background = this.getBackground()
    }
  },

  data() {
    return {
      thumbLeft: 0,
      thumbTop: 0,
      background: null
    }
  },

  mounted() {
    const { bar, thumb } = this.$refs

    const dragConfig = {
      drag: (event) => {
        this.handleDrag(event)
      },
      end: (event) => {
        this.handleDrag(event)
      }
    }

    draggable(bar, dragConfig)
    draggable(thumb, dragConfig)
    this.update()
  }
}
</script>
<style lang="scss" scoped>
.color-alpha-slider {
  position: relative;
  box-sizing: border-box;
  width: calc(100% - 20px);
  height: 12px;
  cursor: pointer;
  background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAwAAAAMCAIAAADZF8uwAAAAGUlEQVQYV2M4gwH+YwCGIasIUwhT25BVBADtzYNYrHvv4gAAAABJRU5ErkJggg==);
  &__bar {
    position: relative;
    background: linear-gradient(to right, rgba(255, 255, 255, 0) 0%, rgba(255, 255, 255, 1) 100%);
    height: 100%;
  }
  &__thumb {
    position: absolute;
    cursor: pointer;
    box-sizing: border-box;
    left: 0;
    top: 0;
    width: 4px;
    height: 100%;
    border-radius: 1px;
    background: #fff;
    border: 1px solid #f0f0f0;
    box-shadow: 0 0 2px rgba(0, 0, 0, 0.6);
    z-index: 1;
  }
}
.is_vertical {
  width: 20px;
  height: 180px;

  .color-alpha-slider__bar {
    background: linear-gradient(to bottom, rgba(255, 255, 255, 0) 0%, rgba(255, 255, 255, 1) 100%);
  }

  .color-alpha-slider__thumb {
    left: 0;
    top: 0;
    width: 100%;
    height: 4px;
  }
}
</style>
