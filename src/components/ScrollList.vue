<template>
  <div ref="containerRef" class="list-container">
    <ul ref="listRef" class="list">
      <li
        v-for="(item, index) in listData"
        :key="`list-${index}`"
        :data-idx="item.idx"
      >
        <slot :listItem="item"></slot>
      </li>
    </ul>
  </div>
</template>
<script lang="ts" setup>
import {
  ref,
  reactive,
  computed,
  watchEffect,
  nextTick,
  onUnmounted,
  onMounted,
  watch,
} from "vue";

const props = defineProps<{
  listData: Array<any>;
}>();

// 列表HTMLElementDom
const listRef = ref<HTMLElement>();
const containerRef = ref<HTMLElement>();

const state = reactive({
  // 容器高度
  containerH: 0,
  // 行高度
  rowH: 50,
  // 显示行数量
  rowCount: 0,
  // 列表初始索引
  startIdx: 0,
  // 列表末端索引
  endIdx: 1,
}) as any;

const scrollHandler = (e: any) => {
  // 当前滚动高度
  const { scrollTop: curScrollTop } = e.target;
  state.startIdx = Math.floor(curScrollTop / state.rowH);
  state.endIdx = state.startIdx + state.rowCount;
  listRef.value?.style.setProperty(
    "padding-top",
    `${state.rowH * state.startIdx}px`
  );
};

const listData = computed(() => {
  return (
    props.listData
      .slice(state.startIdx, state.endIdx || 0)
      .map((item, index) => {
        item.idx = state.startIdx + index;
        return item;
      }) ?? []
  );
});

onMounted(() => {
  if (listData.value.length > 0) {
    nextTick(() => {
      // 计算容器高度
      state.containerH = containerRef.value?.clientHeight;
      // eslint-disable-next-line no-debugger
      // debugger;
      // 计算每行高度
      state.rowH = listRef.value?.children[0]?.offsetHeight;
      // 计算容器内真实 DOM 数量 = Math.ceil(容器高度 / 条目高度)
      state.rowCount = Math.ceil(state.containerH / state.rowH);
      state.endIdx = state.rowCount;
      // 设置离别OA总高度 = 一行的高度*行数
      const listH = state.rowH * props.listData.length;
      // 设置容器高度
      listRef.value?.style.setProperty("height", `${listH}px`);
    });
  }
  containerRef.value?.removeEventListener("scroll", scrollHandler);
  containerRef.value?.addEventListener("scroll", scrollHandler);
});
onUnmounted(() => {
  containerRef.value?.removeEventListener("scroll", scrollHandler);
});
</script>

<style lang="scss" scoped>
.list-container {
  border: 1px solid #000;
  height: 600px;
  overflow-y: auto;
  width: 400px;
  // 滚动条样式设置
  &::-webkit-scrollbar {
    /*滚动条整体样式*/
    width: 8px; /*高宽分别对应横竖滚动条的尺寸*/
    height: 1px;
  }
  &::-webkit-scrollbar-thumb {
    /*滚动条里面小方块*/
    border-radius: 10px;
    -webkit-box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
    background: #e5e5e5;
  }
  &::-webkit-scrollbar-track {
    /*滚动条里面轨道*/
    -webkit-box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
    border-radius: 10px;
  }
  ul {
    margin: 0;
    padding: 0;
    & li {
      list-style-type: none;
      height: 50px;
      line-height: 50px;
      border-bottom: 1px solid #313131;
      padding: 0 20px;
    }
  }
}
</style>
