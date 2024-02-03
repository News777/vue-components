<template>
  <div class="color-hue-slider" :class="{ 'is-vertical': vertical }">
    <div class="color-hue-slider__bar" @click="handleClick" ref="bar"></div>
    <div
      class="color-hue-slider__thumb"
      :style="{
        left: thumbLeft + 'px',
        top: thumbTop + 'px'
      }"
      ref="thumb"
    ></div>
  </div>
</template>

<script>
import draggable from '../utils/draggable'

export default {
  name: 'ColorHueSlider',

  props: {
    color: {
      required: true
    },

    vertical: Boolean
  },

  data() {
    return {
      thumbLeft: 0,
      thumbTop: 0
    }
  },

  computed: {
    hueValue() {
      const hue = this.color.get('hue')
      return hue
    }
  },

  watch: {
    hueValue() {
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
      const { thumb } = this.$refs
      let hue

      if (!this.vertical) {
        let left = event.clientX - rect.left
        left = Math.min(left, rect.width - thumb.offsetWidth / 2)
        left = Math.max(thumb.offsetWidth / 2, left)

        hue = Math.round(((left - thumb.offsetWidth / 2) / (rect.width - thumb.offsetWidth)) * 360)
      } else {
        let top = event.clientY - rect.top
        top = Math.min(top, rect.height - thumb.offsetHeight / 2)
        top = Math.max(thumb.offsetHeight / 2, top)

        hue = Math.round(
          ((top - thumb.offsetHeight / 2) / (rect.height - thumb.offsetHeight)) * 360
        )
      }

      this.color.set('hue', hue)
    },

    getThumbLeft() {
      if (this.vertical) return 0
      const el = this.$el
      const hue = this.color.get('hue')

      if (!el) return 0
      const thumb = this.$refs.thumb
      return Math.round((hue * (el.offsetWidth - thumb.offsetWidth / 2)) / 360)
    },

    getThumbTop() {
      if (!this.vertical) return 0
      const el = this.$el
      const hue = this.color.get('hue')

      if (!el) return 0
      const thumb = this.$refs.thumb
      return Math.round((hue * (el.offsetHeight - thumb.offsetHeight / 2)) / 360)
    },

    update() {
      this.thumbLeft = this.getThumbLeft()
      this.thumbTop = this.getThumbTop()
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
.color-hue-slider {
  position: relative;
  box-sizing: border-box;
  width: calc(100% - 20px);
  height: 12px;
  background-color: #f00;
  padding: 0 2px;
  cursor: pointer;
  &__bar {
    position: relative;
    background: linear-gradient(
      to right,
      #f00 0%,
      #ff0 17%,
      #0f0 33%,
      #0ff 50%,
      #00f 67%,
      #f0f 83%,
      #f00 100%
    );
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
.is-vertical {
  width: 12px;
  height: 180px;
  padding: 2px 0;

  .color-hue-slider__bar {
    background: linear-gradient(
      to bottom,
      #f00 0%,
      #ff0 17%,
      #0f0 33%,
      #0ff 50%,
      #00f 67%,
      #f0f 83%,
      #f00 100%
    );
  }

  .color-hue-slider__thumb {
    left: 0;
    top: 0;
    width: 100%;
    height: 4px;
  }
}
</style>
