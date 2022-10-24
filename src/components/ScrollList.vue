<template>
  <div ref="pageRef" class="list-page">
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
} from "vue";

const props = defineProps<{
  listData: Array<any>;
}>();

// 列表HTMLElementDom
const listRef = ref<HTMLElement>();
const pageRef = ref<HTMLElement>();

const state = reactive({
  // 容器高度
  containerH: 0,
  // 行高度
  rowH: 0,
  // 显示行数量
  rowCount: 0,
}) as any;

const scrollHandler = ()=>{
    // 当前滚动高度
  const curScrollTop = pageRef.value?.scrollTop;
  const addCount = Math.floor((curScrollTop - data.initH) / data.unitH)
  listRef.value?.style.setProperty('padding-top', `${0 * state.rowH}px`)
  console.log(curScrollTop)
}

onMounted(async () => {
    await nextTick(() => {
      // 计算容器高度
      state.containerH = pageRef.value?.clientHeight;
      // 可见视图内行数
      state.rowCount = Math.ceil(state.containerH / state.rowH);
    });
    // 计算每行高度
    state.rowH = listRef.value?.children[0]?.offsetHeight;
    state.rowCount = Math.ceil(state.containerH / state.rowH);
    // 设置离别OA总高度 = 一行的高度*行数
    const listH = state.rowH * props.listData.length
    listRef.value?.style.setProperty('height', `${listH}px`)

    // pageRef.value.removeEventListener('scroll', scrollHandler)
    pageRef.value?.addEventListener('scroll', scrollHandler)

    console.log(state.containerH,state.rowH,state.rowCount);
});

const listData = computed(() => {
  return props.listData.slice(0, state.rowCount);
});
// 实时计算展示的李彪数据
// const listData = computed(()=>{
//   let endIdx = state.startIdx + state.displayCount;
//   if(endIdx >= props.listData.length) endIdx = props.listData.length;
//   return props.listData.slice(state.startIdx,endIdx).map((item,index)=>{
//     item.idx = state.startIdx + index + 1;
//     return item
//   })
// });
</script>
<style lang="scss" scoped>
.list-page {
  border: 1px solid #000;
  height: 600px;
  overflow-y: auto;
  ul {
    padding: 20px;
    width: 400px;
    & li {
      list-style-type: none;
      height: 50px;
    }
  }
}
</style>