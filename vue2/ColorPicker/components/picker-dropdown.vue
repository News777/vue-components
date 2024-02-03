<template>
  <transition name="zoom-in-top">
    <div class="color-dropdown" v-show="showPopper">
      <div class="color-dropdown__menu">
        <div :class="{ active: activeTab === 'tab1' }" @click="activeTab = 'tab1'">普通颜色</div>
        <div :class="{ active: activeTab === 'tab2' }" @click="activeTab = 'tab2'">线性颜色</div>
        <div :class="{ active: activeTab === 'tab3' }" @click="activeTab = 'tab3'">直接输入</div>
      </div>
      <template v-if="activeTab === 'tab2'">
        <div class="linear-color-config">
          <span class="linear-title">{{ `linear-gradient：(` }}</span>
          <div
            class="update-btn"
            :class="{ 'active-btn': updateName === 'deg' }"
            @click="handleLinearColor('deg')"
          >
            {{ linearColor.deg + 'deg' }}
          </div>
          <span>{{ `，` }}</span>
          <div class="color-group" v-for="(stepColor, step) in linearColor.group" :key="step">
            <div
              class="update-btn"
              :class="{ 'active-btn': updateName === 'color' && updateIndex === step }"
              @click="handleLinearColor('color', step, stepColor.color)"
            >
              {{ stepColor.color }}
            </div>
            <div
              class="update-btn ml-xs"
              :class="{ 'active-btn': updateName === 'groupDeg' && updateIndex === step }"
              @click="handleLinearColor('groupDeg', step, stepColor.deg)"
            >
              {{ `${stepColor.deg[0]}% ${stepColor.deg[1]}%` }}
            </div>
            <span v-show="step != linearColor.group.length - 1">{{ `，` }}</span>
          </div>
          <span>{{ `) ` }}</span>
        </div>
        <div class="linear-color-value" v-if="['deg', 'groupDeg'].includes(updateName)">
          <div class="value-row" v-if="updateName === 'deg'">
            <div class="row-title">渐变轴：</div>
            <input type="range" v-model="linearColor.deg" min="0" max="360" />
            <span class="row-unit">deg</span>
          </div>
          <div class="quick-value-btns" v-if="updateName === 'deg'">
            <button
              class="custom-btn"
              v-for="item in quickValueBtns"
              :key="item.value"
              :title="item.value"
              @click="linearColor.deg = item.value"
            >
              {{ item.label }}
            </button>
          </div>
          <template v-if="updateName === 'groupDeg'">
            <div
              class="value-row"
              v-for="(deg, index) in linearColor.group[updateIndex].deg"
              :key="index"
            >
              <div class="row-title">渐变轴：</div>
              <input
                type="range"
                v-model="linearColor.group[updateIndex].deg[index]"
                min="0"
                max="100"
              />
              <span class="row-unit">deg</span>
            </div>
          </template>
        </div>
      </template>
      <template v-if="activeTab === 'tab1' || updateName === 'color'">
        <div class="color-dropdown__main-wrapper">
          <hue-slider ref="hue" :color="color" vertical style="float: right"></hue-slider>
          <panel ref="sl" :color="color"></panel>
        </div>
        <alpha-slider ref="alpha" :color="color"></alpha-slider>
        <div class="color-info">
          <input v-model="color.value" class="input-style" type="text" @blur="handleConfirm" />
        </div>
      </template>
      <template v-if="activeTab === 'tab3'">
        <input v-model="customColor" class="input-style custom-color-input" type="text" />
      </template>
      <div class="control">
        <div class="control-btn" @click="clear">清除</div>
        <div class="control-btn primary" @click="confirm">确认</div>
      </div>
    </div>
  </transition>
</template>

<script>
import Panel from './panel.vue'
import HueSlider from './hue-slider.vue'
import AlphaSlider from './alpha-slider.vue'
import Color from '../utils/color'
export default {
  name: 'PickerDropdown',
  components: {
    Panel,
    HueSlider,
    AlphaSlider
  },
  props: {
    showPopper: {
      type: Boolean,
      default: false
    },
    value: {
      required: true,
      type: String,
      default: ''
    }
  },
  model: {
    event: 'change',
    prop: 'showPopper'
  },
  data() {
    const color = new Color({
      enableAlpha: true,
      format: 'rgb'
    })
    return {
      activeTab: 'tab1',
      color,
      updateName: 'deg',
      updateIndex: 0,
      linearColor: {
        deg: 0,
        group: [
          {
            color: 'rgba(255, 255, 255, 1)',
            deg: [0, 0]
          },
          {
            color: 'rgba(255, 255, 255, 1)',
            deg: [0, 0]
          }
        ]
      },
      quickValueBtns: [
        { label: 'to top', value: '0' },
        { label: 'to right', value: '90' },
        { label: 'to top right', value: '45' },
        { label: 'to bottom', value: '180' },
        { label: 'to left', value: '270' },
        { label: 'to bottom right', value: '135' },
        { label: 'to left top', value: '315' },
        { label: 'to bottom left', value: '225' }
      ],
      customColor: ''
    }
  },
  watch: {
    showPopper(val) {
      if (val) {
        if (!this.value?.includes('linear-gradient')) {
          this.color.fromString(this.value)
          this.linearColorInit()
          this.$nextTick(() => {
            const { sl, hue, alpha } = this.$refs
            sl && sl.update()
            hue && hue.update()
            alpha && alpha.update()
          })
          this.activeTab = 'tab1'
        } else {
          this.activeTab = 'tab2'
          this.splitLinearColor(this.value)
        }
        this.customColor = ''
      }
    },
    activeTab: {
      handler(n) {
        if (n === 'tab1') {
          if (this.value?.includes('linear-gradient')) {
            this.splitLinearColor(this.value)
          } else {
            this.color.fromString(this.value)
            this.linearColorInit()
          }
        }
      }
    },
    'color.value': {
      deep: true,
      handler(e) {
        if (this.activeTab === 'tab2') this.linearColor.group[this.updateIndex].color = e
      }
    }
  },
  mounted() {},
  methods: {
    clear() {
      this.$emit('clearColor')
    },
    confirm() {
      const map = {
        tab1: this.color.value,
        tab2: `linear-gradient(${this.linearColor.deg}deg,${this.linearColor.group
          .map(
            (item) =>
              ' ' + item.color + (item.deg.some(Boolean) ? ` ${item.deg[0]}% ${item.deg[1]}%` : '')
          )
          .join(',')})`,
        tab3: this.customColor?.replace(';', '')
      }
      this.$emit('pick', map[this.activeTab])
    },
    handleConfirm() {
      this.color.fromString(this.color.value)
    },
    linearColorInit() {
      this.linearColor = {
        deg: 0,
        group: [
          {
            color: 'rgba(255, 255, 255, 1)',
            deg: [0, 0]
          },
          {
            color: 'rgba(255, 255, 255, 1)',
            deg: [0, 0]
          }
        ]
      }
    },
    splitLinearColor(value) {
      const n = this.splitByCommaOutsideRgba(value?.replace(/^linear-gradient\((.*)\)$/, '$1'))
      if (n) {
        const includeColorList = n.filter((item) => this.containsColor(item))
        const excludeColorList = n.filter((item) => !this.containsColor(item))
        excludeColorList.map((item) => {
          if (item.includes('deg')) {
            this.linearColor.deg = item.replace('deg', '')
          } else {
            const f = this.quickValueBtns.find((citem) => citem.label === item)
            this.linearColor.deg = f ? f.value : 0
          }
        })
        includeColorList.map((item, index) => {
          const colorAndDegreesRegex =
            /(#[0-9a-fA-F]{3,6}|hsl\(\s*[\d.]+\s*,\s*[\d.]+%\s*,\s*[\d.]+%\s*\)|rgb\(\s*[\d.]+\s*,\s*[\d.]+\s*,\s*[\d.]+\s*\)|rgba\(\s*[\d.]+\s*,\s*[\d.]+\s*,\s*[\d.]+\s*,\s*[\d.]+\s*\))\s*(\d+%)?\s*(\d+%)?/
          const matches = item.match(colorAndDegreesRegex)
          if (matches) {
            this.linearColor.group[index].color = matches[1]
            this.linearColor.group[index].deg = [
              matches[2]?.replace('%', '') || 0,
              matches[3]?.replace('%', '') || 0
            ]
          }
        })
      }
    },
    splitByCommaOutsideRgba(input) {
      // 使用正则表达式查找并匹配rgba()外的逗号
      const parts = input?.split(/,(?![^()]*\))/g)

      // 清理并定义最终结果
      const result = parts?.map((s) => s.trim())
      return result
    },
    containsColor(str) {
      // 使用正则表达式匹配常见的颜色表示格式，例如 #RRGGBB、rgb(r, g, b)、rgba(r, g, b, a) 或预定义的颜色名称
      const colorRegex =
        /#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})|rgb\(\d{1,3},\s*\d{1,3},\s*\d{1,3}\)|rgba\(\d{1,3},\s*\d{1,3},\s*\d{1,3},\s*(1|0(\.\d+)?)\)|\b(red|blue|green|yellow|purple)\b/g

      // 使用正则表达式的 exec 方法在字符串中寻找颜色并返回匹配结果
      const match = colorRegex?.exec(str)

      // 如果找到匹配的颜色，则返回 true，否则返回 false
      return match !== null
    },
    handleLinearColor(type, index, val) {
      this.updateName = type
      if (index >= 0) {
        this.updateIndex = index
      }
      if (type === 'color') {
        this.color.fromString(val)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.zoom-in-top-enter-active,
.zoom-in-top-leave-active {
  opacity: 1;
  transform: scaleY(1);
  transition: transform 300ms cubic-bezier(0.23, 1, 0.32, 1),
    opacity 300ms cubic-bezier(0.23, 1, 0.32, 1);
  transform-origin: center top;
}
.zoom-in-top-enter,
.zoom-in-top-leave-active {
  opacity: 0;
  transform: scaleY(0);
}

.custom-btn {
  background-color: #eee;
  border: none;
  padding: 5px 10px;
  font-size: 14px;
  width: fit-content;
  border-radius: 4px;
  color: rgb(14, 165, 196);
  box-shadow: 0 2px #dfd9d9;
  cursor: pointer;
}

.custom-btn:active {
  color: white;
  box-shadow: 0 2px #dfd9d9;
  transform: translateY(2px);
}

.custom-btn:hover:not(:disabled) {
  background: lightcoral;
  color: white;
  text-shadow: 0 1px #bcb4b4;
}

.custom-btn:disabled {
  cursor: auto;
  color: grey;
}

.color-dropdown {
  min-width: 300px;
  width: fit-content;
  &__menu {
    margin-bottom: 5px;
    height: 20px;
    width: fit-content;
    display: flex;
    padding: 2px;
    align-items: center;
    color: #666666;
    & > div {
      width: fit-content;
      height: fit-content;
      cursor: pointer;
      display: flex;
      font-size: 15px;
      border-bottom: 2px solid transparent;
      font-weight: bold;
      justify-content: center;
      align-items: center;
      &:not(:first-of-type) {
        margin-left: 10px;
      }
      &:hover {
        filter: brightness(120%); /* 鼠标悬停时，图标颜色变亮 */
        border-bottom: 2px solid rgb(2, 166, 188);
      }
    }
    .active {
      color: #007bff;
      border-bottom: 2px solid rgb(2, 166, 188);
    }
  }
  .linear-color-config {
    display: flex;
    align-items: center;
    margin-bottom: 5px;
    white-space: nowrap;
    .linear-title {
      padding-bottom: 2px;
    }
    .color-group {
      display: flex;
    }

    .update-btn {
      cursor: pointer;
      border-bottom: 1px solid #666666;
      padding: 0 4px;
    }
    .active-btn {
      color: #0999d6;
      font-weight: bold;
    }
  }
  .ml-xs {
    margin-left: 4px;
  }
  .linear-color-value {
    padding: 10px 20px 10px 0;
    .quick-value-btns {
      margin-top: 10px;
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      grid-gap: 5px; /* 可选：添加间距 */
    }
    .value-row {
      display: flex;
      align-items: center;

      &:not(:last-of-type) {
        margin-bottom: 10px;
      }
      .row-title {
        font-weight: bold;
        width: 80px;
        color: #666666;
        text-align: right;
      }
      & > input {
        margin-top: 0;
        width: calc(100% - 80px) !important;
      }
      .row-unit {
        margin-left: 4px;
      }
    }
  }
  &__main-wrapper {
    margin-bottom: 6px;

    &::after {
      content: '';
      display: table;
      clear: both;
    }
  }
  .color-info {
    margin-top: 10px;
  }
  .input-style {
    padding: 4px 10px;
    border: 2px solid #ccc;
    border-radius: 5px;
    font-size: 14px;
    color: #555;
    outline: none;
  }

  .input-style:focus {
    border-color: #007bff;
    box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
  }
  .custom-color-input {
    width: 80%;
    margin-bottom: 10px;
  }
  .control {
    text-align: right;
    .control-btn {
      border: 1px solid #dcdfe6;
      display: inline-block;
      line-height: 1;
      text-align: center;
      background: #ffffff;
      white-space: nowrap;
      cursor: pointer;
      padding: 8px 12px;
      color: #606266;
      border-radius: 4px;
      &:not(:last-of-type) {
        margin-right: 10px;
      }
    }
    .primary {
      color: #ffffff;
      background-color: #1890ff;
      border-color: #1890ff;
    }
  }
}
</style>
<style></style>
