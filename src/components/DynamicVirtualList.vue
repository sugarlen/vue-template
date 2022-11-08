<template>
  <div
    ref="containerRef"
    class="list-container"
    @scroll="scrollHandler($event)"
  >
    <ul ref="listRef" class="list">
      <li
        v-for="(item, index) in listData"
        :key="`list-${index}`"
        :data-idx="item.idx"
        :id="item.id"
      >
        <slot :listItem="item"></slot>
      </li>
    </ul>
  </div>
</template>
<script lang="ts" setup>
import { ref, reactive, computed, nextTick, onMounted, onUpdated } from "vue";

const props = defineProps<{
  // 列表数据
  listData: Array<any>;
  //预估行高度
  estimatedItemH: number;
}>();

// 列表HTMLElementDom
const listRef = ref<HTMLElement>();
const containerRef = ref<HTMLElement>();

const state = reactive({
  // 容器高度
  containerH: 600,
  // 行高度
  rowH: 50,
  // 显示行数量
  rowCount: 0,
  // 列表初始索引
  startIdx: 0,
  // 列表末端索引
  endIdx: 1,
}) as any;

// 列表项位置
const positions: Record<string, any>[] = props.listData.map((item, index) => {
  return {
    index,
    height: props.estimatedItemH,
    top: index * props.estimatedItemH,
    bottom: (index + 1) * props.estimatedItemH,
  };
});

const scrollHandler = (e: any) => {
  // 设置缓冲区域的数据量
  const aboveCount = 5;
  const belowCount = 5;
  // 当前滚动高度
  const { scrollTop: curScrollTop } = e.target;
  //此时的开始索引
  state.startIdx =
    getStartOrEndIndex(curScrollTop) - aboveCount > 0
      ? getStartOrEndIndex(curScrollTop) - aboveCount
      : 0;
  //此时的结束索引
  state.endIdx =
    getStartOrEndIndex(curScrollTop + state.containerH) + belowCount >
    props.listData.length
      ? props.listData.length
      : getStartOrEndIndex(curScrollTop + state.containerH) + belowCount;
  setStartOffset();
};

const setStartOffset = () => {
  let startOffset =
    state.startIdx >= 1 ? positions[state.startIdx - 1].bottom : 0;
  listRef.value?.style.setProperty("padding-top", `${startOffset}px`);
};

//获取列表起始或结束索引
const getStartOrEndIndex = (scrollTop = 0): number => {
  //二分法查找
  return binarySearch(positions, scrollTop);
};
//二分法查找
const binarySearch = (list: Record<string, any>[], value: number): number => {
  let start: number = 0;
  let end: number = list.length - 1;
  let tempIndex: any = null;
  while (start <= end) {
    let midIndex = Math.floor((start + end) / 2);
    let midValue = list[midIndex].bottom;
    if (midValue === value) {
      // 说明当前滚动区域加上可视区域刚好是一个结点的边界，那么我们可以以其下一个结点作为末尾元素
      return midIndex + 1;
    } else if (midValue < value) {
      // 由于当前值与目标值还有一定的差距，所以我们需要增加start值以让下次中点的落点更靠后
      start = midIndex + 1;
    } else if (midValue > value) {
      // 因为我们的目的并不是找到第一个满足条件的值，而是要找到满足条件的最小索引值
      if (tempIndex === null || tempIndex > midIndex) {
        tempIndex = midIndex;
      }
      // 由于我们要继续找更小的索引，所以需要让end-1以缩小范围，让下次中点的落点更靠前
      end = end - 1;
    }
  }
  return tempIndex;
};

const listData = computed(() => {
  console.log(state.startIdx, state.endIdx);
  return (
    props.listData.slice(state.startIdx, state.endIdx).map((item, index) => {
      item.idx = state.startIdx + index;
      return item;
    }) ?? []
  );
});

onMounted(() => {
  nextTick(() => {
    // 计算容器高度
    state.containerH = containerRef.value?.clientHeight;
    // 计算容器内真实 DOM 数量 = Math.ceil(容器高度 / 条目高度)
    state.rowCount = Math.ceil(state.containerH / props.estimatedItemH);
    state.endIdx = state.startIdx + state.rowCount;
    console.log("mounted");
  });
});

onUpdated(() => {
  // 获取真实列表项dom高度
  getItemRealH();
  // 更新列表总高度
  let listH = positions[positions.length - 1].bottom;
  listRef.value?.style.setProperty("height", `${listH}px`);
  console.log("updated");
});
const getItemRealH = () => {
  let nodes: any = listRef.value?.childNodes;
  nodes.forEach((node: any) => {
    if (node.nodeName === "LI") {
      let height = node.offsetHeight;
      let index = Number(node.id);
      let oldHeight = positions[index].height;
      let dValue = oldHeight - height;
      //存在差值
      if (dValue) {
        positions[index].bottom = positions[index].bottom - dValue;
        positions[index].height = height;
        for (let k = index + 1; k < positions.length; k++) {
          positions[k].top = positions[k - 1].bottom;
          positions[k].bottom = positions[k].bottom - dValue;
        }
      }
      console.log("height", height);
    }
  });
};
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
      // height: 50px;
      // line-height: 50px;
      border-bottom: 1px solid #313131;
      padding: 0 20px;
    }
  }
}
</style>
