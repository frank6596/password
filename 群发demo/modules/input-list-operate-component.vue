<script setup lang="ts">
import { computed, nextTick, reactive, ref, watch } from 'vue';
defineOptions({
  name: 'InputListOperateComponent'
});

interface Props {
  listName: string;
}

const props = defineProps<Props>();

interface Emits {
  (e: 'change'): void;
}

const emit = defineEmits<Emits>();

defineExpose({});

interface ListItem {
  value: string;
  key: number;
}
const list = reactive<ListItem[]>([
  {
    value: '',
    key: Date.now()
  }
]);
watch(list, (newList, oldList) => {
  console.log('Array changed!');
  console.log('New items:', newList);
  console.log('Old items:', oldList);
  emit('change', newList);
});

function init() {
  console.log('init');
  list = [
    {
      value: '',
      key: Date.now()
    }
  ];
}

const handleChange = (val: any) => {
  console.log(val);
  emit('change', list);
};

function addItem(ind) {
  // let item : any = undefined
  // list.push('')

  list.push({
    value: '',
    key: Date.now()
  });
}

function deleteItem(ind, item) {
  // list.splice(ind, 1)
  const index = list.indexOf(item);
  if (index !== -1) {
    list.splice(index, 1);
  }
}
</script>

<template>
  <div class="add-input-list-container">
    <!--
 <AButton size="small" ghost type="primary" @click="addItem">
      <template #icon>
        <icon-ic-round-plus class="align-sub text-icon" />
      </template>
      <span class="ml-8px">添加</span>
    </AButton>
-->
    <div class="input-list-container">
      <div v-for="(item, ind) in list" :key="item.key" class="input-list-item-container">
        <!--
 <AInput
          v-model:value="item.value"
          placeholder="请输入"
        />
-->
        <ATextarea v-model:value="item.value" autosize placeholder="请输入" allow-clear />
        <!-- <p class="item-value">{{item}}</p> -->
        <AButton size="small" type="text" @click="addItem">
          <template #icon>
            <icon-ic-round-plus class="align-sub text-icon" />
          </template>
        </AButton>
        <AButton size="small" :disabled="list.length == 1" type="text" danger @click.stop="deleteItem(ind, item)">
          <template #icon>
            <icon-ic-round-delete class="align-sub text-icon" />
          </template>
        </AButton>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.add-input-list-container {
  .input-list-container {
    // margin-top: 10px;
    .input-list-item-container {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 5px;

      .item-value {
        width: 80%;
      }
    }
  }
}
</style>
