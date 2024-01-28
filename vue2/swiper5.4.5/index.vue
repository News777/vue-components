<template>
  <div class="swiper-container" :style="slideItemStyle" ref="swiperContainer">
    <div class="swiper-wrapper" :style="slideItemStyle">
      <div
        v-for="item in ctx"
        :key="item"
        class="swiper-no-swiping swiper-slide"
      >
        <img :src="item" alt="" class="img" />
        <!-- <div
          class="img"
          :style="{
            background: `url(${item})`
          }"
        ></div> -->
      </div>
    </div>
    <!-- 如果需要分页器 -->
    <div class="swiper-pagination"></div>

    <!-- 如果需要导航按钮 -->
    <div
      class="swiper-button-prev"
      :class="{ 'swiper-button-hidden': direction === 'vertical' }"
    ></div>
    <div
      class="swiper-button-next"
      :class="{ 'swiper-button-hidden': direction === 'vertical' }"
    ></div>
  </div>
</template>

<script>
import Swiper from 'swiper'; // 注意引入的是Swiper
import 'swiper/css/swiper.min.css'; // 注意这里的引入

export default {
  name: 'Swiper',
  props: {
    // 接受图片路径的集合
    ctx: {
      type: Array,
      default: () => [],
    },
    // swiper 的样式，主要是限制width、hieght
    ele: {
      required: true,
      type: Object,
      default: () => ({}),
      validator: function (value) {
        return ['height', 'width'].every((key) =>
          Boolean(Object.keys?.(value)?.includes(key))
        );
      },
    },
    // 轮播速度，以秒为单位
    speed: {
      type: Number,
      default: 5,
      validator: function (value) {
        return value >= 1 && value <= 100;
      },
    },
    // 轮播图方向
    direction: {
      type: String,
      default: 'horizontal',
      validator: function (value) {
        return ['horizontal', 'vertical'].includes(value);
      },
    },
    // 轮播图切换效果，具体参考文档
    effect: {
      type: String,
      default: 'slide',
    },
    // swiper的配置项，接受合并
    swiperConfig: {
      type: Object,
      default: () => ({}),
    },
  },
  data() {
    return {
      swiper: null, // Swiper 实例
    };
  },
  computed: {
    slideItemStyle() {
      return {
        width: this.ele.width + 'px',
        height: this.ele.height + 'px',
      };
    },
  },
  watch: {
    speed: {
      handler(n) {
        this.reInit();
      },
    },
    effect: {
      handler(n) {
        this.reInit();
      },
    },
    direction: {
      handler(n) {
        this.reInit();
      },
    },
  },
  mounted() {
    // 初始化 Swiper 实例
    this.$nextTick(() => {
      this.init();
    });
  },
  beforeDestroy() {
    // 销毁 Swiper 实例
    if (this.swiper) {
      this.swiper.destroy();
      this.swiper = null;
    }
  },
  methods: {
    init() {
      this.swiper = new Swiper('.swiper-container', {
        // Swiper 配置项
        loop: true,
        direction: this.direction || 'horizontal',
        autoplay: {
          //自动开始
          delay: (this.speed || 5) * 1000, //时间间隔
          disableOnInteraction: false, //*手动操作轮播图后不会暂停*
        },
        slidesPerView: 1,
        allowTouchMove: false,
        noSwiping: true,
        autoHeight: false,
        // 窗口变化,重新init,针对F11全屏和放大缩小,必须加
        observer: true,
        observeParents: false,
        setWrapperSize: false,
        // 如果需要分页器
        pagination: {
          el: '.swiper-pagination',
          clickable: true, // 分页器可以点击
        },
        on: {
          init: () => {
            this.$nextTick(() => {
              this.swiper.el.style.height = this.ele.height + 'px';
            });
          },
        },
        effect: this.effect || 'slide',
        // 如果需要前进后退按钮
        navigation: {
          nextEl: '.swiper-button-next',
          prevEl: '.swiper-button-prev',
          hiddenClass: 'swiper-button-hidden',
        },
        ...this.swiperConfig,
      });
    },
    reInit() {
      if (this.swiper) {
        this.swiper.destroy();
        this.swiper = null;
      }
      this.init();
    },
  },
};
</script>

<style lang="scss" scoped>
.swiper-container {
  z-index: 0;

  .swiper-slide {
    width: 100%;
    height: 100%;
  }
  .swiper-button-hidden {
    display: none;
  }
  .img {
    width: 100%;
    height: 100%;
  }
}
</style>
