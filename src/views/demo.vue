<template>
  <div>
    <!-- <VirtualList :listData="listData">
      <template #default="{ listItem }">
        <span>{{ listItem.title }}</span>
        <span>{{ listItem.content }}</span>
      </template>
    </VirtualList>
    <VirtualTree :treeData="treeData"></VirtualTree> -->
    <DynamicVirtualList :listData="listData" :estimatedItemH="estimatedItemH">
      <template #default="{ listItem }">
        <span style="margin-right: 30px">{{ listItem.title }}</span>
        <span>{{ listItem.content }}</span>
      </template>
    </DynamicVirtualList>
  </div>
</template>
<script lang="ts" setup>
// import VirtualList from "@/components/VirtualList.vue";
// import VirtualList from "@/components/ScrollList.vue";
// import VirtualTree from "@/components/virtualTree.vue";
import DynamicVirtualList from "@/components/DynamicVirtualList.vue";

import { ref } from "vue";
import { faker } from "@faker-js/faker";

const estimatedItemH = ref(25);

const getListData = (count: number) => {
  let result = [];
  for (let i = 0; i < count; i++) {
    result.push({
      id: i,
      title: `item--${i}`,
      content: faker.lorem.sentences(),
    });
  }
  return result;
};
const listData = getListData(1000);

type TreeNode = {
  title: any;
  label: any;
  key: any;
  children?: any;
};
// 构建 dataSource 所需的数据
const createTreeData = (path = "0", level = 3, count = 10) => {
  const list = [];
  for (let i = 0; i < count; i += 1) {
    const key = `${path}-${i}`;
    const treeNode: TreeNode = {
      title: key,
      label: key,
      key,
    };

    if (level === 1 && key === "0-0-0-0") {
      // 构建100万个子节点
      // treeNode.children = createTreeData(key, level - 1, 1000000);
    } else if (level > 0) {
      treeNode.children = createTreeData(key, level - 1);
    }

    list.push(treeNode);
  }
  return list;
};
const treeData = createTreeData();
console.log(treeData);
</script>
<style lang="sass" scoped></style>
