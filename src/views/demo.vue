<template>
    <div>
      <VirtualList :listData="listData" >
        <template #default="{ listItem }">
          <span>{{ listItem.title }}</span>
          <span>{{ listItem.content }}</span>
        </template>
      </VirtualList>
    </div>
</template>
<script lang="ts" setup>
import VirtualList from "@/components/VirtualList.vue"
// import VirtualList from "@/components/ScrollList.vue"


const getListData = (count:number) => {
  let result = [];
  for(let i = 0; i< count; i++){
    result.push({
      title: `item--${i}`,
      content: `content--${i}`
    });
  };
  return result;
};

const listData = getListData(100);
console.log(listData)

</script>
<style lang="sass" scoped>

</style>