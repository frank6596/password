<script setup lang="tsx">
import { nextTick, reactive, ref, watch } from 'vue';
import type { TablePaginationConfig, TableProps } from 'ant-design-vue';
import { Button } from 'ant-design-vue';
import { jsonClone } from '@sa/utils';
import { useBoolean, useLoading } from '@sa/hooks';
import { fetchGetMassmsgStatistics } from '@/service/api';
import { $t } from '@/locales';
import { timestampToDate } from '@/utils/date';
import MassmsgStatisticsSearch from './modules/massmsgstatistics-search.vue';
import Details from './modules/details.vue';

defineOptions({
  name: 'MassMsgstatistics'
});

const visible = defineModel<boolean>('visible', {
  default: false
});

function closeModal() {
  visible.value = false;
}

// 初始化抽屉
watch(visible, () => {
  if (visible.value) {
    nextTick(() => {
      resetSearchParams();
      getData();
    });
  }
});

const { loading, startLoading, endLoading } = useLoading();
interface apiParams {
  page: number;
  limit: number;
  task_id: string;
  account: string;
}

const searchParams: apiParams = reactive({
  page: 1,
  limit: 20,
  task_id: '',
  account: ''
});

/** reset search params */
function resetSearchParams() {
  const initPramas = {
    page: 1,
    limit: 20,
    task_id: '',
    account: '',
    status: 0
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
  startLoading();

  const formattedParams = formatSearchParams(searchParams);

  const response = await fetchGetMassmsgStatistics(formattedParams);
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
    key: 'task_id',
    dataIndex: 'task_id',
    title: '任务ID',
    align: 'center',
    width: 150
  },
  {
    key: 'task_record',
    dataIndex: 'task_record',
    title: '任务记录集合',
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
    key: 'create_time',
    dataIndex: 'create_time',
    title: '创建时间',
    align: 'center',
    width: 220,
    customRender: ({ text }) => {
      return text ? timestampToDate(text) : '';
    }
  },
  {
    key: 'update_time',
    dataIndex: 'update_time',
    title: '更新时间',
    align: 'center',
    width: 220,
    customRender: ({ text }) => {
      return text ? timestampToDate(text) : '';
    }
  }
];

const statusOptions = [
  {
    value: 0,
    label: '全部',
    color: 'default'
  },
  {
    value: 1,
    label: '等待',
    color: 'default'
  },
  {
    value: 2,
    label: '运行中',
    color: 'processing'
  },
  {
    value: 3,
    label: '暂停',
    color: 'warning'
  },
  {
    value: 4,
    label: '完成',
    color: 'success'
  },
  {
    value: 5,
    label: '失败',
    color: 'error'
  }
];

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
  updatePagination({
    current: pag.current,
    pageSize: pag.pageSize
  });
};

// 任务详情列表
const { bool: detailsVisible, setTrue: openDetailsModal } = useBoolean();
const detailsData: Ref<T | null> = ref(null);

function handleDetails(record: object) {
  detailsData.value = jsonClone(record);

  openDetailsModal();
}
</script>

<template>
  <Details v-model:visible="detailsVisible" :row-data="detailsData"></Details>
  <AModal v-model:open="visible" title="群发任务统计" :width="2000" destroy-on-close @cancel="closeModal">
    <template #footer>
      <AButton key="back" @click="closeModal">关闭</AButton>
    </template>
    <div class="min-h-600px flex-col-stretch gap-16px overflow-hidden lt-sm:overflow-auto">
      <MassmsgStatisticsSearch v-model:model="searchParams" @reset="resetSearchParams" @search="getDataByPage" />
      <ACard
        title="群发任务统计列表"
        :bordered="false"
        :body-style="{ flex: 1, overflow: 'hidden' }"
        class="flex-col-stretch sm:flex-1-hidden card-wrapper"
      >
        <ATable
          class="h-full"
          size="small"
          row-key="id"
          :scroll="{ x: 1800, y: 390 }"
          :columns="columns"
          :data-source="dataSource"
          :pagination="pagination"
          :loading="loading"
          @change="handleTableChange"
        >
          <!-- <DetailsList v-model:visible="detailsVisible" :row-data="detailsData" /> -->
          <template #bodyCell="{ column, record }">
            <template v-if="column.key === 'status'">
              <ATag :color="statusOptions[record.status]['color']">
                {{ statusOptions[record.status]['label'] || '' }}
              </ATag>
            </template>
            <template v-if="column.key === 'task_record'">
              <Button v-if="record.task_record" type="primary" size="small" @click="handleDetails(record.task_record)">
                查看
              </Button>
              <span v-else>暂无数据</span>
            </template>
          </template>
        </ATable>
      </ACard>
    </div>
  </AModal>
</template>

<style scoped></style>
