<script setup lang="tsx">
import { $t } from '@/locales';

defineOptions({
  name: 'MassMsgstatisticsdetails'
});

interface Props {
  /** row data */
  rowData?: null;
}

const props = defineProps<Props>();

const visible = defineModel<boolean>('visible', {
  default: false
});

function closeModal() {
  visible.value = false;
}

const columns = [
  {
    key: 'index',
    title: $t('common.index'),
    dataIndex: 'index',
    align: 'center',
    width: 64,
    customRender: ({ index }) => {
      return index + 1;
    }
  },
  {
    key: 'account',
    dataIndex: 'account',
    title: '账号',
    align: 'center'
  },
  {
    key: 'complete_count',
    dataIndex: 'complete_count',
    title: '完成数量',
    align: 'center'
  }
];
</script>

<template>
  <AModal v-model:open="visible" title="任务记录集合" :width="2000" destroy-on-close @cancel="closeModal">
    <template #footer>
      <AButton key="back" @click="closeModal">关闭</AButton>
    </template>
    <div class="min-h-680px flex-col-stretch gap-16px overflow-hidden lt-sm:overflow-auto">
      <ACard
        title="任务记录集合列表"
        :bordered="false"
        :body-style="{ flex: 1, overflow: 'hidden' }"
        class="flex-col-stretch sm:flex-1-hidden card-wrapper"
      >
        <ATable
          class="h-full"
          size="small"
          row-key="id"
          :scroll="{ x: 1000, y: 390 }"
          :columns="columns"
          :data-source="props.rowData"
        />
      </ACard>
    </div>
  </AModal>
</template>

<style scoped></style>
