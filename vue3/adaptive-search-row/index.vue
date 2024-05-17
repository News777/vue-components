<template>
  <div class="relative search-layout" :style="realHeight">
    <slot v-if="!FormItemList.length"></slot>
    <component :is="Render" v-else />
    <a-space class="absolute right-10px bottom-16px">
      <slot name="operate"></slot>
      <Transition name="fade">
        <a-button v-if="rowNum > limitRowNumber" type="link" @click="fold">
          <span>{{ isFold ? '展开' : '收起' }}</span>
          <DownOutlined v-if="isFold" />
          <up-outlined v-else />
        </a-button>
      </Transition>
    </a-space>
  </div>
</template>

<script setup lang="tsx">
  import { UpOutlined, DownOutlined } from '@ant-design/icons-vue';
  import { Col, type ColProps, Row, type RowProps } from 'ant-design-vue';
  import { computed, CSSProperties, h, onMounted, ref, useSlots, VNode } from 'vue';
  import { useWindowSizeFn } from '@/hooks/event/useWindowSizeFn';
  import { sortBy } from 'lodash-es';

  const props = withDefaults(
    defineProps<{
      rowConfig?: RowProps;
      colConfig?: ColProps;
      limitRowNumber?: number;
    }>(),
    {
      colConfig: () => ({
        span: 8,
        sm: 24,
        lg: 12,
        xl: 8,
        xxl: 6,
      }),
      rowConfig: () => ({}),
      limitRowNumber: 2,
    }
  );

  const slots = useSlots();

  // 一行 一列所占 span 参考 a-col
  const currentColSpan = ref<number>(8);

  // 控制展开/收起
  const isFold = ref<boolean>(true);

  // 接受的插槽 转变成虚拟DOM列表重新进行渲染
  const FormItemList = computed<VNode[]>(
    () =>
      slots
        ?.default?.()
        ?.filter((item) => item.children && item.children !== 'v-if' && item.children !== '') || []
  );

  const FormItemListLength = computed<number>(() => FormItemList.value?.length || 0);

  // 搜索框列数
  const ColNum = computed<number>(() => {
    return 24 / currentColSpan.value;
  });

  // 搜索框行数
  const rowNum = computed<number>(() => {
    return Math.ceil(FormItemListLength.value / ColNum.value);
  });

  // 判断需要渲染的搜索列表 除以 列数 是否为整数，若为，则按钮另起一行，反之，则与末尾共用一行
  const isNeedNewRow = computed<boolean>(() => {
    return (
      limitFormItemList.value?.length ===
      Math.ceil(limitFormItemList.value?.length / ColNum.value) * ColNum.value
    );
  });

  const realHeight = computed<CSSProperties>(() => {
    const realRow = Math.ceil(limitFormItemList.value?.length / ColNum.value);
    return {
      height: (realRow + (isNeedNewRow.value ? 1 : 0)) * 48 + 'px',
    };
  });

  // 需要展开/收起限制搜索框数量
  const limitFormItemList = computed<VNode[]>(() => {
    // 展开/收起的前提 为2行以上
    let sliceAll = FormItemListLength.value || 0; // 处理为全部
    if (isFold.value) {
      sliceAll =
        rowNum.value > props.limitRowNumber
          ? props.limitRowNumber * ColNum.value
          : FormItemListLength.value;
    }
    return FormItemList.value?.slice(0, sliceAll) || [];
  });

  const fold = () => {
    isFold.value = !isFold.value;
  };

  const Render = () => {
    return h(
      Row,
      {
        ...props.rowConfig,
        class: 'w-full',
      },
      {
        default: () =>
          limitFormItemList.value.map((vnode) => {
            return h(Col, props.colConfig, {
              default: () => vnode,
            });
          }),
      }
    );
  };

  const spanMatch = (width: number) => {
    const map = {
      1600: 6,
      1200: 8,
      992: 12,
      576: 24,
    };

    for (const [key, value] of sortBy(Object.entries(map), (o) => o[1])) {
      if (width >= Number(key)) {
        currentColSpan.value = value;
        break;
      }
    }
  };
  onMounted(() => {
    spanMatch(document.documentElement.clientWidth || document.body.clientWidth); // 挂载时先执行一次
  });
  const sizeHandler = (e: any) => {
    spanMatch(e.target.innerWidth);
  };
  useWindowSizeFn(sizeHandler, 16);
</script>

<style scoped lang="scss">
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity 0.5 ease;
  }

  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
  }

  :deep(.ant-form-item) {
    width: 100% !important;
  }

  .search-layout {
    width: 100%;
    transition: height 0.5s ease;
  }
</style>
