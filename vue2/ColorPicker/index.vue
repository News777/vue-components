<template>
  <div :class="['color-picker', disabled ? 'is-disabled' : '']">
    <div
      class="color-inner"
      :style="{
        background: value
      }"
      @click.stop.prevent="handleTrigger"
    >
      <svg viewBox="0 0 1024 1024" xmlns="http://www.w3.org/2000/svg" width="16" height="16">
        <path
          v-if="!value"
          d="M556.736893 512.149956L1014.402812 54.184126c12.396368-12.396368 12.396368-32.490481 0-44.88685-12.396368-12.396368-32.490481-12.396368-44.88685 0L511.850044 467.263107 54.184126 9.597188C41.787757-2.79918 21.693644-2.79918 9.297276 9.597188c-12.396368 12.396368-12.396368 32.490481 0 44.88685l457.665918 457.665918L9.297276 969.815874c-12.396368 12.396368-12.396368 32.490481 0 44.88685 6.198184 6.198184 14.295812 9.297276 22.39344 9.297276s16.195255-3.099092 22.393439-9.297276L511.850044 557.036806l457.665918 457.665918c6.198184 6.198184 14.295812 9.297276 22.39344 9.297276 8.097628 0 16.195255-3.099092 22.393439-9.297276 12.396368-12.396368 12.396368-32.490481 0-44.88685L556.736893 512.149956z"
          fill="#b0b0b0"
          p-id="981"
        ></path>
        <path
          v-else
          d="M512 784c-8.5 0-16.6-3.4-22.6-9.4l-480-480c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0L512 706.7l457.4-457.4c12.5-12.5 32.8-12.5 45.3 0s12.5 32.8 0 45.3l-480 480c-6.1 6-14.2 9.4-22.7 9.4z"
          fill="#b0b0b0"
          p-id="1279"
        ></path>
      </svg>
      <!-- <i :class="value ? 'el-icon-arrow-down' : 'el-icon-close'"></i> -->
    </div>
    <picker-dropdown
      ref="dropdown"
      :class="['color-picker-panel', popperClass || '']"
      :style="pos"
      v-model="showPicker"
      @pick="confirmValue"
      @clearColor="clearValue"
      :value="value"
    >
    </picker-dropdown>
  </div>
</template>

<script>
import pickerDropdown from './components/picker-dropdown.vue'
export default {
  name: 'ColorPicker',
  components: {
    pickerDropdown
  },
  props: {
    value: String,
    disabled: Boolean,
    colorFormat: String,
    popperClass: String,
    format: String
  },
  model: {
    event: 'change',
    prop: 'value'
  },
  data() {
    return {
      showPicker: false,
      parentEle: null,
      resizeObserver: null,
      pos: {}
    }
  },
  // computed: {
  //   pos() {
  //     const rect = this.parentEle?.getBoundingClientRect()
  //     let tabWidth
  //     ;(async () => {
  //       await this.$nextTick()
  //       tabWidth = this.$refs?.dropdown?.$el?.clientWidth
  //     })()
  //     console.log(tabWidth)
  //     return rect
  //       ? {
  //           left: rect.left - rect.width - tabWidth + 'px',
  //           top: rect.top + rect.height + 'px'
  //         }
  //       : {}
  //   }
  // },
  mounted() {
    document.addEventListener('click', this.handler, true)
    this.resizeObserver = new ResizeObserver((entries) => {
      for (const entry of entries) {
        const rect = this.parentEle?.getBoundingClientRect()
        if (entry.contentRect && rect) {
          // Firefox implements `contentBoxSize` as a single content rect, rather than an array
          this.pos = {
            left: rect.left - rect.width - entry.contentRect.width + 'px',
            top: rect.top - entry.contentRect.height + 'px'
          }
        }
      }
    })
    this.resizeObserver.observe(this.$refs.dropdown.$el)
  },
  beforeMount() {
    document.removeEventListener('click', this.handler, true)
    if (this.resizeObserver) {
      this.resizeObserver.unobserve(this.$refs.dropdown.$el)
    }
  },
  methods: {
    handler(e) {
      if (
        this.showPicker &&
        !this.findAncestorWithClass(e.target, 'color-picker-panel') &&
        !this.findAncestorWithClass(e.target, 'color-picker')
      ) {
        this.showPicker = false
      }
    },
    findAncestorWithClass(element, className) {
      while (element && !element?.classList?.contains(className)) {
        // 当element为null时停止循环（即已经到达了DOM树的顶部）
        // 当element的classList包含className时停止循环
        element = element.parentElement
      }
      // 返回具有所需类的元素或null（如果未找到）
      return element ? true : false
    },
    handleTrigger($event) {
      if (this.disabled) return
      this.showPicker = !this.showPicker
      this.parentEle = $event.srcElement
    },
    confirmValue(selection) {
      const value = selection
      this.$emit('change', value)
      this.showPicker = false
    },
    clearValue() {
      this.$emit('change', '')
      this.showPicker = false
      // this.resetColor()
    }
    // resetColor() {
    //   this.$nextTick((_) => {
    //     if (this.value) {
    //       this.color.fromString(this.value)
    //     }
    //   })
    // }
  }
}
</script>

<style lang="scss" scoped>
.color-picker {
  width: 36px;
  height: 36px;
  background: #ffffff;
  border: 1px solid #e6e6e6;
  border-radius: 4px;
  padding: 4px;
  .color-inner {
    border: 1px solid #999999;
    width: 26px;
    height: 26px;
    border-radius: 2px;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    i {
      color: #b0b0b0;
    }
  }
  .color-picker-panel {
    position: fixed;
    z-index: 10;
    padding: 6px;
    line-height: 1 !important;
    font-size: inherit;
    z-index: 9999;
    box-sizing: content-box;
    background-color: #ffffff;
    border: 1px solid #ebeef5;
    border-radius: 4px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  }
}
.is-disabled {
  cursor: not-allowed;
}
</style>
