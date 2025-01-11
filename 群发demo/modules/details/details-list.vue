<script lang="ts" setup>
import { computed, nextTick, onMounted, reactive, ref, watch } from 'vue';
import { jsonClone } from '@sa/utils';
import type { TablePaginationConfig, TableProps } from 'ant-design-vue';
import { useBoolean, useLoading } from '@sa/hooks';
import { Button, Popconfirm, Tag } from 'ant-design-vue';
import { timestampToDate } from '@/utils/date';

import { $t } from '@/locales';
import { fetchGetMassmsgDetailList, fetchGetWsprotocolList } from '@/service/api';
// import {  useTableScroll } from '@/hooks/common/table';

import DetailsListSearch from './details-list-search.vue';

defineOptions({
  name: 'DetailsList'
});

interface Props {
  /** row data */
  rowData?: null;
}

const props = defineProps<Props>();

interface Emits {
  (e: 'submitted'): void;
}

const emit = defineEmits<Emits>();

const visible = defineModel<boolean>('visible', {
  default: false
});

function closeModal() {
  visible.value = false;
}

// 初始化抽屉
watch(visible, () => {
  if (visible.value) {
    console.log(props.rowData);
    nextTick(() => {
      resetSearchParams();
      getData();
    });
  }
});

onMounted(() => {});

const { loading, startLoading, endLoading } = useLoading();
const { bool: empty, setBool: setEmpty } = useBoolean();

interface apiParams {
  page: number;
  limit: number;
  id: string;
  sender: string;
  status: number;
  receiver: string;
}

const searchParams: apiParams = reactive({
  page: 1,
  limit: 20,
  id: props?.rowData?.id || '',
  sender: '',
  status: -1,
  receiver: ''
});

/** reset search params */
function resetSearchParams() {
  const initPramas = {
    page: 1,
    limit: 20,
    id: props?.rowData?.id || '',
    sender: '',
    status: -1,
    receiver: ''
  };
  Object.assign(searchParams, jsonClone(initPramas));
}

/**
 * update search params
 *
 * @param params
 */
function updateSearchParams(params: Partial<Parameters<A>[0]>) {
  Object.assign(searchParams, params);
}

/**
 * get data by page number
 *
 * @param pageNum the page number. default is 1
 */
async function getDataByPage(pageNum: number = 1) {
  updatePagination({
    current: pageNum
  });

  updateSearchParams({
    page: pageNum,
    limit: pagination.pageSize!
  });

  await getData();
}
async function getData() {
  await nextTick();
  startLoading();

  const formattedParams = formatSearchParams(searchParams);

  const response = await fetchGetMassmsgDetailList(formattedParams);
  // const response = await fetchGetWsprotocolList(formattedParams);
  console.log(response);

  let total = 0;
  const { data, error } = response;
  if (!error) {
    dataSource.value = data.list || [];

    total = data?.total || 0;
  }

  updatePagination({
    current: formattedParams.page,
    pageSize: formattedParams.limit,
    total
  });

  endLoading();
}
function formatSearchParams(params: Record<string, unknown>) {
  const formattedParams: Record<string, unknown> = {};

  Object.entries(params).forEach(([key, value]) => {
    if (value !== null && value !== undefined) {
      formattedParams[key] = value;
    }
  });

  return formattedParams;
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
    key: 'sender',
    dataIndex: 'sender',
    title: '发送者',
    align: 'center',
    width: 150
  },
  {
    key: 'receiver',
    dataIndex: 'receiver',
    title: '接收者',
    align: 'center',
    width: 150
  },
  {
    key: 'step',
    dataIndex: 'step',
    title: '步骤',
    align: 'center',
    width: 150
  },
  {
    key: 'send_block',
    dataIndex: 'send_block',
    title: '发送阻塞',
    align: 'center',
    width: 150,
    customRender: ({ text }) => {
      return text == true ? '是' : '否';
    }
  },
  {
    key: 'send_index',
    dataIndex: 'send_index',
    title: '发送下标',
    align: 'center',
    width: 150
  },
  {
    key: 'status',
    dataIndex: 'status',
    title: '状态',
    align: 'center',
    width: 100
  },
  {
    key: 'desc',
    dataIndex: 'desc',
    title: '备注',
    align: 'center',
    width: 180
  },
  {
    key: 'update_time',
    dataIndex: 'update_time',
    title: '更新时间',
    align: 'center',
    width: 220,
    customRender: ({ text }) => {
      return timestampToDate(text);
    }
  }
];

const stepList = ref([
  {
    value: 0,
    label: '等待中',
    color: 'default'
  },
  {
    value: 1,
    label: '添加好友',
    color: 'processing'
  },
  {
    value: 2,
    label: '发送招呼语',
    color: 'success'
  },
  {
    value: 3,
    label: '发送内容',
    color: 'error'
  },
  {
    value: 4,
    label: '运行结束',
    color: 'warning'
  }
]);

const statusList = ref([
  {
    value: 0,
    label: '等待',
    color: 'default'
  },
  {
    value: 1,
    label: '运行中',
    color: 'processing'
  },
  {
    value: 2,
    label: '成功',
    color: 'success'
  },
  {
    value: 3,
    label: '失败',
    color: 'error'
  },
  {
    value: 4,
    label: '超时',
    color: 'warning'
  }
]);

type TableDataWithIndex<T> = T & { index: number };

const dataSource: Ref<TableDataWithIndex<T>[]> = ref([]);

const pagination: TablePaginationConfig = reactive({
  current: 1,
  pageSize: 20,
  showSizeChanger: true,
  pageSizeOptions: ['20', '30', '50', '100'],
  total: 0,
  showTotal: total => `共 ${total} 条`,
  onChange: async (page: number, limit: number) => {
    pagination.current = page;
    updateSearchParams({
      page,
      limit
    });
    // update table data
    getData();
  }
});
function updatePagination(update: Partial<TablePaginationConfig>) {
  Object.assign(pagination, update);
}

const handleTableChange: TableProps['onChange'] = (
  pag: { pageSize: number; current: number },
  filters: any,
  sorter: any
) => {
  console.log(pag, filters.sorter);
  updatePagination({
    current: pag.current,
    pageSize: pag.pageSize
  });
};
</script>

<template>
  <AModal v-model:open="visible" title="群发任务详情" :width="2000" destroy-on-close @cancel="closeModal">
    <template #footer>
      <AButton key="back" @click="closeModal">关闭</AButton>
    </template>

    <div class="min-h-500px flex-col-stretch gap-16px overflow-hidden lt-sm:overflow-auto">
      <DetailsListSearch v-model:model="searchParams" @reset="resetSearchParams" @search="getDataByPage" />
      <ACard
        title="群发任务详情列表"
        :bordered="false"
        :body-style="{ flex: 1, overflow: 'hidden' }"
        class="flex-col-stretch sm:flex-1-hidden card-wrapper"
      >
        <ATable
          class="h-full"
          size="small"
          row-key="id"
          :scroll="{ x: 1800, y: 300 }"
          :columns="columns"
          :data-source="dataSource"
          :pagination="pagination"
          :loading="loading"
          @change="handleTableChange"
        >
          <template #bodyCell="{ column, record }">
            <template v-if="column.key === 'step'">
              <ATag :color="stepList[record.step]['color']">{{ stepList[record.step]['label'] || '' }}</ATag>
            </template>
            <template v-if="column.key === 'status'">
              <ATag :color="statusList[record.status]['color']">{{ statusList[record.status]['label'] || '' }}</ATag>
            </template>
            <template v-if="column.key === 'desc'">
              <APopover trigger="hover">
                <template #content>
                  <div class="popoverClass">{{ record.desc }}</div>
                </template>
                <span v-if="record.desc?.length > 20">{{ record.desc.slice(0, 20) + '...' }}</span>
                <span v-else>{{ record.desc }}</span>
              </APopover>
            </template>
          </template>
        </ATable>
      </ACard>
    </div>
  </AModal>
</template>

<style lang="scss" scoped>
.popoverClass {
  max-width: 400px;
  word-break: break-all;
  /* white-space: pre; */
}
</style>
