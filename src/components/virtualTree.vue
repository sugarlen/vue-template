<template>
  <div ref="containerRef" class="tree-container">
    <ul ref="treeRef" class="tree">
      <li
        v-for="(item, index) in props.treeData"
        :key="`tree-${index}`"
        :data-idx="item.idx"
      >
        <el-icon><ArrowRight /><ArrowDown /></el-icon>
        <span>{{ item.key }}</span>
      </li>
    </ul>
  </div>
</template>
<script lang="ts" setup>
const props = defineProps<{
  treeData: Array<any>;
}>();
console.log(props.treeData);

function resolveTreeDataToList(treeData: any, keysMap = {}) {
  // const { id: kId, children, text } = keysMap;
  // 将 Tree 结构的数据打平成一个 list 存在其中，用于渲染
  const list: Array<any> = [];
  // 将 Tree 中的每一项使用 id(key) 作为键存到 obj 中，方便后续通过 id(key) 取数
  const obj = {};

  // 使用递归遍历所有数据
  const traverseData = (tree = [], pIds = [], pNames = [], pLevels = []) => {
    return tree.map((item, index) => {});
  };

  traverseData(treeData);

  return { obj, list };
}
resolveTreeDataToList(props.treeData);
</script>
<style lang="scss" scoped>
.tree-container {
  border: 1px solid #000;
  height: 700px;
  overflow-y: auto;
  width: 300px;
  ul {
    margin: 0;
    padding: 0;
    & li {
      list-style-type: none;
      height: 30px;
      line-height: 30px;
      padding-left: 20px;
    }
  }
}
</style>
