<template>
    <div ref="pageRef" class="list-page">
        <ul ref="listRef" class="list">
            <li v-for="(item,index) in listData" :key="`list-${index}`"
            :data-idx="item.idx">
                <slot :listItem="item"></slot>
            </li>
        </ul>
    </div>
</template>
<script lang="ts" setup>
import { ref, reactive, computed, watchEffect, nextTick, onUnmounted, onMounted } from 'vue'


const props = defineProps<{
  listData: Array<any>
}>();

// 列表HTMLElementDom
const listRef = ref<HTMLElement>();
const pageRef = ref<HTMLElement>();

const state = reactive({
  // 列表第一项的高度（起始高度）
  initH:0,

  // 一行的高度
  unitH:0,

  // 屏幕范围内能显示个数
  displayCount:1,
  
  // 列表项起始值
  startIdx:0,

  // 容器高度
  screenH: 0

}) as any;

// 实时计算展示的李彪数据
const listData = computed(()=>{
  let endIdx = state.startIdx + state.displayCount;
  if(endIdx >= props.listData.length) endIdx = props.listData.length;
  return props.listData.slice(state.startIdx,endIdx).map((item,index)=>{
    item.idx = state.startIdx + index + 1;
    return item
  })
});

const scrollHandler = ()=>{
  // 当前滚动高度
  const curScrollTop:any = pageRef.value?.scrollTop;
  if(curScrollTop > state.initH){
      const addCount = Math.floor((curScrollTop - state.initH) / state.unitH)
      listRef.value?.style.setProperty('padding-top', `${addCount * state.unitH}px`)
      state.startIdx = addCount
  } else {
      listRef.value?.style.setProperty('padding-top','0px')
      state.startIdx = 0
  }
  console.log(state.displayCount)
};

watchEffect(()=>{
    if(props.listData.length>0){
        nextTick(()=>{
            // 列表距离顶部的距离
            state.initH = listRef.value?.getBoundingClientRect().top
                + pageRef.value?.scrollTop;
                console.log(listRef.value?.getBoundingClientRect().top
                , pageRef.value?.scrollTop)
            // 计算每行高度
            state.unitH = listRef.value?.children[0]?.offsetHeight;
            // 计算屏幕内能显示的行数
            state.displayCount = Math.ceil(state.screenH / state.unitH)
            console.log(state.displayCount)
            // 设置离别OA总高度 = 一行的高度*行数
            const listH = state.unitH * props.listData.length
            listRef.value?.style.setProperty('height', `${listH}px`)
            pageRef.value?.removeEventListener('scroll', scrollHandler)
            pageRef.value?.addEventListener('scroll', scrollHandler)
        })
    }
});

onMounted(()=>{
    state.screenH = pageRef.value?.clientHeight;
})

onUnmounted(()=>{
    pageRef.value?.removeEventListener('scroll', scrollHandler);
})





</script>
<style lang="scss" scoped>
.list-page {
    border: 1px solid #000;
    height: 800px;
    overflow-y: auto;
    ul{
        padding:20px;
        width: 400px;
        & li{
            list-style-type: none;
            height: 50px;
        }
    }
}
</style>