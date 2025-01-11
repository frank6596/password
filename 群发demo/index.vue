<script setup lang="tsx">
import { computed, nextTick, reactive, ref, watch } from 'vue';
import { Button, Popconfirm, Tag } from 'ant-design-vue';
import { useBoolean } from '@sa/hooks';
import { jsonClone } from '@sa/utils';
import { batchDeleteMassmsg, continueTaskMassmsg, fetchGetMassmsgList, pauseTaskMassmsg } from '@/service/api';
import { useTable, useTableOperate, useTableScroll } from '@/hooks/common/table';
import { $t } from '@/locales';
import { timestampToDate } from '@/utils/date';

import MassMsgStatistics from './modules/massmsgstatistics/index.vue';
import MassmsgOperateDrawer from './modules/massmsg-operate-drawer.vue';
import DeleteIFOperateDrawer from './modules/deleteif-operate-drawer.vue';
import MassmsgSearch from './modules/massmsg-search.vue';
import DetailsList from './modules/details/details-list.vue';

const {
  tableWrapperRef,
  scrollConfig,
  dealScrollConfig,
  mainWrapperRef,
  searchWrapperRef,
  tableHeaderOperationWrapperRef,
  cardTableScrollConfig
} = useTableScroll();

const {
  columns,
  columnChecks,
  data,
  getData,
  getDataByPage,
  loading,
  mobilePagination,
  searchParams,
  resetSearchParams
} = useTable({
  apiFn: fetchGetMassmsgList,
  apiParams: {
    page: 1,
    limit: 20,
    // if you want to use the searchParams in Form, you need to define the following properties, and the value is null
    // the value can not be undefined, otherwise the property in Form will not be reactive
    // id: '',
    sender: '',
    mass_msg_type: getMassmsgType(),
    status: 0
  },
  columns: () => [
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
      key: 'task_name',
      dataIndex: 'task_name',
      title: '任务名称',
      align: 'center',
      width: 150
    },
    {
      key: 'account_group_names',
      dataIndex: 'account_group_names',
      title: '发送用户分组',
      align: 'center',
      width: 150,
      ellipsis: true
      // customRender: ({ record }) => {
      //   return record.account_group_names.join(',');
      // }
    },
    {
      key: 'data_packet_name',
      dataIndex: 'data_packet_name',
      title: '数据包名称',
      align: 'center',
      width: 120
    },
    {
      key: 'mass_msg_type',
      dataIndex: 'mass_msg_type',
      title: '群发类型',
      align: 'center',
      width: 100,
      // 传0则不加入筛选 1群发普通类型 2群发新用户类型
      customRender: ({ text }) => {
        return text == 1 ? '群发普通类型' : '群发新用户类型';
      }
    },
    {
      key: 'task_type',
      dataIndex: 'task_type',
      title: '任务类型',
      align: 'center',
      width: 100,
      customRender: ({ text }) => {
        const labels = ['直推任务', '回复推任务'];
        return labels[text - 1];
      }
    },
    {
      key: 'num',
      dataIndex: 'num',
      title: '总数 / 成功 / 失败 / 超时',
      align: 'center',
      width: 180,
      customRender: ({ record }) => {
        return `${record.total_num} / ${record.success_num} / ${record.fail_num} / ${record.timeout_num}`;
      }
    },
    {
      key: 'retry_count',
      dataIndex: 'retry_count',
      title: '失败后重试次数',
      align: 'center',
      width: 120
    },
    {
      key: 'cur_retry_count',
      dataIndex: 'cur_retry_count',
      title: '当前失败后重试次数',
      align: 'center',
      width: 120
    },
    {
      key: 'status',
      dataIndex: 'status',
      title: '状态',
      align: 'center',
      width: 100,
      customRender: ({ record }) => {
        if (record.status === null) {
          return null;
        }

        const tagMap: Record<Api.Common.EnableStatus, string> = {
          1: 'default',
          2: 'processing',
          3: 'warning',
          4: 'success',
          5: 'error'
        };

        const statusList = [
          {
            value: 0,
            label: '未知'
          },
          {
            value: 1,
            label: '等待'
          },
          {
            value: 2,
            label: '运行中'
          },
          {
            value: 3,
            label: '暂停'
          },
          {
            value: 4,
            label: '完成'
          },
          {
            value: 5,
            label: '失败'
          }
        ];

        const label = statusList[record.status].label;

        return <Tag color={tagMap[record.status]}>{label}</Tag>;
      }
    },
    {
      key: 'media_type',
      dataIndex: 'media_type',
      title: '消息类型',
      align: 'center',
      width: 100,
      customRender: ({ text }) => {
        const labels = [
          '文本',
          '图片',
          '语音',
          '视频',
          '文件',
          '扩展',
          '名片',
          '通过广告发送消息',
          '商业扩展文本消息',
          '',
          '',
          '',
          '转发消息'
        ];
        return labels[text - 1];
      }
    },
    {
      key: 'add_contacts',
      dataIndex: 'add_contacts',
      title: '是否添加联系人',
      align: 'center',
      width: 100,
      customRender: ({ text }) => {
        return text === 0 ? '否' : '是';
      }
    },
    {
      key: 'ac_send_interval',
      dataIndex: 'ac_send_interval',
      title: '添加联系人间隔(秒)',
      align: 'center',
      width: 100
    },
    {
      key: 'send_contents',
      dataIndex: 'send_contents',
      title: '发送内容',
      align: 'center',
      width: 250,
      ellipsis: true
    },
    {
      key: 'greeting',
      dataIndex: 'greeting',
      title: '招呼语',
      align: 'center',
      width: 120
    },
    {
      key: 'greeting_type',
      dataIndex: 'greeting_type',
      title: '招呼类型',
      align: 'center',
      width: 120,
      customRender: ({ text }) => {
        const labels = ['文字', '语音通话', '视频通话'];
        return labels[text];
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
      key: 'per_account_send_num',
      dataIndex: 'per_account_send_num',
      title: '每个账户发送数量',
      align: 'center',
      width: 120
    },
    {
      key: 'pa_execution_interval',
      dataIndex: 'pa_execution_interval',
      title: '每个账户执行间隔(秒)',
      align: 'center',
      width: 120
    },
    {
      key: 'desc',
      dataIndex: 'desc',
      title: '描述',
      align: 'center',
      width: 220,
      ellipsis: true
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
    },
    {
      key: 'operate',
      title: $t('common.operate'),
      align: 'center',
      fixed: 'right',
      width: 180
    }
  ]
});

const {
  drawerVisible,
  operateType,
  editingData,
  handleAdd,
  handleEdit,
  checkedRowKeys,
  rowSelection,
  onBatchDeleted,
  onDeleted,
  handleResizeColumn
  // closeDrawer
} = useTableOperate(data, getData);

async function handleBatchDelete() {
  // request
  console.log(checkedRowKeys.value);
  const ids = checkedRowKeys.value;
  const { error } = await batchDeleteMassmsg({ ids });
  if (!error) {
    onBatchDeleted();
  }
}

function getMassmsgType() {
  let value = 0;
  console.log(JSON.parse(localStorage.getItem('massmsgType')));
  const type = JSON.parse(localStorage.getItem('massmsgType'));
  if (type && type.value) {
    value = type.value;
  }
  return value;
}

async function handleDelete(id: string) {
  // request
  console.log(id);

  const ids = [id];

  const { error } = await batchDeleteMassmsg({ ids });
  if (!error) {
    onDeleted();
  }
}

function edit(id: string) {
  handleEdit(id);
}

async function continueTask(record: object) {
  const params = {
    task_id: record.id
  };
  const { error } = await continueTaskMassmsg(params);
  if (!error) {
    getData();
  }
}
async function pauseTask(record: object) {
  const params = {
    task_id: record.id
  };
  const { error } = await pauseTaskMassmsg(params);
  if (!error) {
    getData();
  }
}

// 任务详情列表
const { bool: detailsVisible, setTrue: openDetailsModal, setFalse: closeDetailsModal } = useBoolean();
const detailsData: Ref<T | null> = ref(null);

function handleDetails(record: object) {
  detailsData.value = jsonClone(record);

  openDetailsModal();
}

// 条件删除
const { bool: deleteDrawerVisible, setTrue: opendeleteDrawer, setFalse: closedeleteDrawer } = useBoolean();
function handleDeleteOfIf() {
  opendeleteDrawer();
}

// 群发任务统计
const { bool: massmsgstaticVisible, setTrue: openmassmsgstaticModal, setFalse: closemassmsgstaticModal } = useBoolean();

function massmsgStatistics() {
  openmassmsgstaticModal();
}
</script>

<template>
  <div ref="mainWrapperRef" class="min-h-500px flex-col-stretch gap-16px overflow-hidden lt-sm:overflow-auto">
    <MassmsgSearch
      ref="searchWrapperRef"
      v-model:model="searchParams"
      @reset="resetSearchParams"
      @search="getDataByPage"
    />
    <ACard
      title="群发列表"
      :bordered="false"
      :body-style="{ padding: '0 24px', flex: 1, overflow: 'hidden' }"
      class="flex-col-stretch sm:flex-1-hidden card-wrapper"
    >
      <template #extra>
        <TableHeaderOperation
          ref="tableHeaderOperationWrapperRef"
          v-model:columns="columnChecks"
          :disabled-delete="checkedRowKeys.length === 0"
          :loading="loading"
          :is-show-delete="true"
          @add="handleAdd"
          @delete="handleBatchDelete"
          @refresh="getData"
        >
          <template #prefix>
            <div class="flex flex-wrap justify-end gap-x-12px gap-y-8px lt-sm:(w-200px py-12px)">
              <!-- :disabled="checkedRowKeys.length === 0" -->
              <Button type="primary" ghost size="small" @click="massmsgStatistics()">群发获取统计</Button>
              <Button type="primary" danger ghost size="small" @click="handleDeleteOfIf()">
                <template #icon>
                  <icon-ic-round-delete class="align-sub text-icon" />
                </template>
                <span>条件删除</span>
              </Button>
            </div>
          </template>
        </TableHeaderOperation>
      </template>
      <ATable
        ref="tableWrapperRef"
        :scroll="cardTableScrollConfig"
        :columns="columns"
        :data-source="data"
        size="small"
        :row-selection="rowSelection"
        :loading="loading"
        row-key="id"
        :pagination="mobilePagination"
        class="table-wrapper h-full"
        :row-class-name="(_record, index) => 'table-row-height-fix'"
        @resize-column="handleResizeColumn"
      >
        <template #bodyCell="{ column, record }">
          <template v-if="column.key === 'operate'">
            <div class="flex-center gap-8px">
              <Button type="primary" ghost size="small" @click="handleDetails(record)">详情</Button>
              <Popconfirm title="确定删除吗？" @confirm="handleDelete(record.id)">
                <Button danger size="small">删除</Button>
              </Popconfirm>
              <Popconfirm v-if="record.status === 2" title="确定暂停任务吗？" @confirm="pauseTask(record)">
                <Button danger size="small">暂停</Button>
              </Popconfirm>
              <Popconfirm v-if="record.status === 3" title="确定继续任务吗？" @confirm="continueTask(record)">
                <Button danger size="small">继续</Button>
              </Popconfirm>
            </div>
          </template>

          <template v-if="column.key === 'send_contents'">
            <div v-for="(item, ind) in record.send_contents" :key="ind">
              <ATooltip>
                <template #title>消息{{ ind + 1 }}【{{ item }}】</template>
                <p style="white-space: nowrop; overflow: hidden; text-overflow: ellipsis">
                  <span>消息{{ ind + 1 }}</span>
                  【{{ item }}】
                </p>
              </ATooltip>
            </div>
          </template>

          <template v-if="column.key === 'account_group_names'">
            <ATooltip>
              <template #title>{{ record.account_group_names.join(',') }}</template>
              <p style="white-space: nowrop; overflow: hidden; text-overflow: ellipsis">
                {{ record.account_group_names.join(',') }}
              </p>
            </ATooltip>
          </template>
        </template>
      </ATable>

      <MassmsgOperateDrawer
        v-model:visible="drawerVisible"
        :operate-type="operateType"
        :row-data="editingData"
        @submitted="getDataByPage"
      />

      <DeleteIFOperateDrawer v-model:visible="deleteDrawerVisible" @submitted="getDataByPage"></DeleteIFOperateDrawer>

      <DetailsList v-model:visible="detailsVisible" :row-data="detailsData" />

      <MassMsgStatistics v-model:visible="massmsgstaticVisible"></MassMsgStatistics>
    </ACard>
  </div>
</template>

<style scoped>
.table-wrapper :deep(.table-row-height-fix) td {
  height: 45px;
}
</style>
