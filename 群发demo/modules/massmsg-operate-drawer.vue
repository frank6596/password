<script setup lang="ts">
import { computed, nextTick, onMounted, reactive, ref, watch } from 'vue';
import { useAntdForm, useFormRules } from '@/hooks/common/form';
import {
  creatMassmsg,
  fetchGetAccountGroupList,
  fetchGetDatapacketList,
  fetchGetMaterialList,
  updateMassmsg
} from '@/service/api';
import { $t } from '@/locales';
import { enableStatusOptions, userGenderOptions } from '@/constants/business';

import InputListOperateComponent from './input-list-operate-component.vue';

defineOptions({
  name: 'MassmsgOperateDrawer'
});

interface Props {
  /** the type of operation */
  operateType: AntDesign.TableOperateType;
  /** the edit row data */
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

const { formRef, validate, resetFields } = useAntdForm();
const { defaultRequiredRule } = useFormRules();

const title = computed(() => {
  const titles: Record<AntDesign.TableOperateType, string> = {
    add: '新增任务',
    edit: '编辑'
  };
  return titles[props.operateType];
});

type Model = {
  mass_msg_type: number;
  task_type: number;
  media_type: number;
  add_contacts: number;
  ac_send_interval: number;
  greeting: string;
  greeting_type: number;
  send_contents: string[];
  material_ids: string[];
  account_group_names: string[];
  sender: string;
  data_packet_name: string;
  receivers: string[];
  receiversTextarea: string;
  per_account_send_num: number;
  pa_execution_interval: number;
  desc: string;
  retry_count: number;
  ad_msg_type_params: {
    first_message: boolean;
    first_text: string;
    first_title: string;
    first_body: string;
    first_source_url: string;
    first_material_id: string;
    first_show_ad_attribution: boolean;
    fs_msg_interval: number;
    text: string;
    title: string;
    body: string;
    source_url: string;
    material_id: string;
    show_ad_attribution: boolean;

    first_render_larger_thumbnail: boolean;
    first_thumbnail_url: string;
    render_larger_thumbnail: boolean;
    thumbnail_url: string;
  };
  bus_msg_type_params: {
    text: string;
    business_owner_jid: string;
    is_forwarded: boolean;
    always_show_ad_attribution: boolean;
  };
};

const model: Model = reactive(createDefaultModel());

function createDefaultModel(): Model {
  return {
    id: undefined,
    mass_msg_type: 1,
    task_type: 1,
    media_type: 1,
    add_contacts: 0,
    ac_send_interval: 0,
    greeting: '',
    greeting_type: 0,
    retry_count: 1,
    send_contents: [],
    material_ids: [],
    account_group_names: [],
    sender: '',
    data_packet_name: '',

    receivers: [],
    receiversTextarea: '',

    per_account_send_num: 1,
    pa_execution_interval: 0,
    desc: '',
    ad_msg_type_params: {
      first_message: false,
      first_text: '',
      first_title: '',
      first_body: '',
      first_source_url: '',
      first_material_id: '',
      first_show_ad_attribution: false,
      fs_msg_interval: 0,
      text: '',
      title: '',
      body: '',
      source_url: '',
      material_id: '',
      show_ad_attribution: false,

      first_render_larger_thumbnail: false,
      first_thumbnail_url: '',
      render_larger_thumbnail: false,
      thumbnail_url: ''
    },
    bus_msg_type_params: {
      text: '',
      business_owner_jid: '',
      is_forwarded: false,
      always_show_ad_attribution: false
    }
  };
}

type RuleKey = Model;

const checkType = async (_rule: Rule, value: string[]) => {
  if (model.media_type == 1) {
    let haveNull = false;
    if (value && value.length > 0) {
      value.map(item => {
        if (!item) {
          haveNull = true;
        }
        return item;
      });
    } else {
      haveNull = true;
    }

    if (haveNull) {
      return Promise.reject('不能为空');
    }
    return Promise.resolve();
  }
  return Promise.resolve();
};

const rules: Record<string, Rule[]> = {
  mass_msg_type: defaultRequiredRule,
  task_type: defaultRequiredRule,
  media_type: defaultRequiredRule,
  add_contacts: defaultRequiredRule,
  ac_send_interval: defaultRequiredRule,
  greeting: defaultRequiredRule,
  greeting_type: defaultRequiredRule,
  send_contents: [{ validator: checkType, trigger: 'change' }],
  material_ids: defaultRequiredRule,
  account_group_names: defaultRequiredRule,
  sender: defaultRequiredRule,
  // data_packet_name: defaultRequiredRule,
  receivers: defaultRequiredRule,
  per_account_send_num: defaultRequiredRule,
  pa_execution_interval: defaultRequiredRule,
  retry_count: defaultRequiredRule,

  ad_msg_type_params: {
    first_message: defaultRequiredRule,
    first_text: defaultRequiredRule,
    first_title: defaultRequiredRule,
    first_body: defaultRequiredRule,
    first_source_url: defaultRequiredRule,
    // first_material_id: defaultRequiredRule,
    first_show_ad_attribution: defaultRequiredRule,
    fs_msg_interval: defaultRequiredRule,
    text: defaultRequiredRule,
    title: defaultRequiredRule,
    body: defaultRequiredRule,
    source_url: defaultRequiredRule,
    // material_id: defaultRequiredRule,
    show_ad_attribution: defaultRequiredRule,
    first_render_larger_thumbnail: defaultRequiredRule,
    first_thumbnail_url: defaultRequiredRule,
    render_larger_thumbnail: defaultRequiredRule,
    thumbnail_url: defaultRequiredRule
  },
  bus_msg_type_params: {
    text: defaultRequiredRule,
    business_owner_jid: defaultRequiredRule,
    is_forwarded: defaultRequiredRule,
    always_show_ad_attribution: defaultRequiredRule
  }
};

const activeKey = ref('1');

// 群发类型 1群发普通类型 2群发新用户类型
const typeOptions1 = ref([
  {
    value: 1,
    label: '群发普通类型'
  },
  {
    value: 2,
    label: '群发新用户类型'
  }
]);

// 任务类型 1直推任务类型 2回复推任务类型
const typeOptions2 = ref([
  {
    value: 1,
    label: '直推任务类型'
  },
  {
    value: 2,
    label: '回复推任务类型'
  }
]);

// 发送类型 1文本 2图片 3语音 4视频 5文件 6扩展 7名片 8通过广告发送消息 9商业扩展文本消息
const typeOptions3 = ref([
  {
    value: 1,
    label: '文本'
  },
  {
    value: 2,
    label: '图片'
  },
  {
    value: 3,
    label: '语音'
  },
  {
    value: 4,
    label: '视频'
  },
  {
    value: 5,
    label: '文件'
  },
  {
    value: 6,
    label: '扩展'
  },
  {
    value: 7,
    label: '名片'
  },
  {
    value: 8,
    label: '通过广告发送消息'
  },
  {
    value: 9,
    label: '商业扩展文本消息'
  },

  // {
  //   value: 10,
  //   label: '拨打语音消息',
  // },
  // {
  //   value: 11,
  //   label: '拨打视频消息',
  // },
  // {
  //   value: 12,
  //   label: '回复指定消息',
  // },

  {
    value: 13,
    label: '转发消息'
  }
]);

async function handleInitModel() {
  if (props.operateType === 'edit' && props.rowData) {
    await nextTick();
    Object.assign(model, props.rowData);
    return;
  }
  Object.assign(model, createDefaultModel());
}

function closeDrawer() {
  visible.value = false;
}

async function handleSubmit() {
  await validate();

  console.log(model);
  // return
  // request
  // 编辑
  if (props.operateType === 'edit' && props.rowData) {
    const { error } = await updateMassmsg(model);
    if (!error) {
      window.$message?.success($t('common.modifySuccess'));
      closeDrawer();
      emit('submitted');
    }
    return;
  }

  // 添加
  const { error } = await creatMassmsg(model);
  if (!error) {
    window.$message?.success($t('common.addSuccess'));
    closeDrawer();
    emit('submitted');
  }
}

// 接受用户号码集合  输入框输入换行隔开号码
// model.receivers   model.receiversTextarea
function receiversTextareaChange(e) {
  // console.log( 'receivers change ', e ,  e.target.value)
  const arr = e.target.value.split('\n');
  model.receivers = arr.filter(item => Boolean(item));
  // console.log(arr, model.receivers)
}

function receiversTextareaPressEnter(e) {
  // console.log( 'receivers PressEnter ', e)
}

// 账号分组多选下拉
const state1 = reactive({
  data: [],
  value: [],
  fetching: false
});
watch(state1.value, () => {
  state1.data = [];
  state1.fetching = false;
});
const listParams1 = {
  page: 1,
  limit: 20,
  group_name: undefined,
  total: 0
};

function handleSearch1(value: string) {
  console.log('fetching search 1', value);
  listParams1.group_name = value;
  listParams1.page = 1;
  listParams1.total = 0;
  state1.data = [];
  getList1(listParams1);
}
const handleChange1 = (val: any) => {
  console.log(val);
  model.account_group_names = [];
  val.map(item => {
    model.account_group_names.push(item.label);
    return item;
  });
  console.log('model.account_group_names', model.account_group_names);
};
function getList1(params) {
  console.log(params);
  state1.fetching = true;
  fetchGetAccountGroupList(params)
    .then(res => {
      console.log(res);
      listParams1.total = res.data.total || 0;
      // response.json()
      const list = res.data.list.map(d => ({
        label: d.name,
        value: d.id,
        count: d.all_num
      }));
      state1.data = listParams1.page == 1 ? list : state1.data.concat(list);
      console.log(state1.data);
    })
    .finally(() => {
      state1.fetching = false;
    });
}

function fetchGroupScroll1() {
  console.log(state1.data.length, listParams1.total);
  if (state1.data.length < listParams1.total && listParams1.page * listParams1.limit < listParams1.total) {
    listParams1.page++;
    getList1(listParams1);
  }
}
function handleFocus1() {
  listParams1.group_name = '';
  listParams1.page = 1;
  listParams1.total = 0;
  state1.data = [];
  getList1(listParams1);
}

// 素材多选下拉
const state2 = reactive({
  data: [],
  value: [],
  fetching: false
});
watch(state2.value, () => {
  state2.data = [];
  state2.fetching = false;
});
const listParams2 = {
  page: 1,
  limit: 20,
  name: undefined,
  material_type: -1, // 传-1则不加入筛选 0文字 1图片 2链接 3语音 4视频 5头像 6名片
  usage_type: 0, // 传-1则不加入筛选 0群发 1昵称 2聊天 3个人说明
  total: 0
};

function handleSearch2(value: string) {
  console.log('fetching search 2', value);
  listParams2.name = value;
  listParams2.page = 1;
  listParams2.total = 0;
  state2.data = [];
  getList2(listParams2);
}
const handleChange2 = (val: any) => {
  console.log(val);
  model.material_ids = [];
  val.map(item => {
    model.material_ids.push(item.value);
    return item;
  });
  console.log('model.material_ids', model.material_ids);
};
function getList2(params) {
  // 发送类型： 1文本 2图片 3语音 4视频 5文件 6扩展 7名片 8通过广告发送消息 9商业扩展文本消息
  // material_type: -1  ,// 传-1则不加入筛选 0文字 1图片 2链接 3语音 4视频 5头像 6名片
  const types = [-1, -1, 1, 3, 4, 7, 2, 6];
  params.material_type = types[model.media_type] || -1;
  console.log(params);
  state2.fetching = true;
  fetchGetMaterialList(params)
    .then(res => {
      console.log(res);
      listParams2.total = res.data.total || 0;
      // response.json()
      const list = res.data.list.map(d => ({
        label: d.name,
        value: d.id
      }));
      state2.data = listParams2.page == 1 ? list : state2.data.concat(list);
      console.log(state2.data);
    })
    .finally(() => {
      state2.fetching = false;
    });
}

function fetchGroupScroll2() {
  console.log(state2.data.length, listParams2.total);
  if (state2.data.length < listParams2.total && listParams2.page * listParams2.limit < listParams2.total) {
    listParams2.page++;
    getList2(listParams2);
  }
}
function handleFocus2() {
  listParams2.name = '';
  listParams2.page = 1;
  listParams2.total = 0;
  state2.data = [];
  getList2(listParams2);
}

// 数据包名称下拉
const state3 = reactive({
  data: [],
  value: {
    label: '',
    value: ''
  },
  // value: [],
  fetching: false
});
watch(state3.value, () => {
  state3.data = [];
  state3.fetching = false;
});
const listParams3 = {
  page: 1,
  limit: 20,
  name: undefined,
  total: 0
};

function handleSearch3(value: string) {
  console.log('fetching search 3', value);
  listParams2.name = value;
  listParams2.page = 1;
  listParams2.total = 0;
  state2.data = [];
  getList2(listParams2);
}
const handleChange3 = (val: any) => {
  console.log(val, state3.value);
  model.data_packet_name = val ? val.label : '';
};
function getList3(params) {
  console.log(params);
  state3.fetching = true;
  fetchGetDatapacketList(params)
    .then(res => {
      console.log(res);
      listParams3.total = res.data.total || 0;
      // response.json()
      const list = res.data.list.map(d => ({
        label: d.name,
        // value: d.name,
        value: d.id,
        count: d.wait_num
      }));
      state3.data = listParams3.page == 1 ? list : state3.data.concat(list);
      console.log(state3.data);
    })
    .finally(() => {
      state3.fetching = false;
    });
}

function fetchGroupScroll3() {
  console.log(state3.data.length, listParams2.total);
  if (state3.data.length < listParams3.total && listParams3.page * listParams3.limit < listParams3.total) {
    listParams3.page++;
    getList3(listParams3);
  }
}
function handleFocus3() {
  // listParams3.name = ''
  listParams3.page = 1;
  listParams3.total = 0;
  state3.data = [];
  getList3(listParams3);
}

// 素材单选下拉选择
const state4 = reactive({
  data: [],
  // value: undefined,
  fetching: false
});
// watch(state4.value, () => {
//   state4.data = [];
//   state4.fetching = false;
// });
const listParams4 = {
  page: 1,
  limit: 20,
  name: undefined,
  material_type: 1, // 传-1则不加入筛选 0文字 1图片 2链接 3语音 4视频 5头像 6名片
  usage_type: 0, // 传-1则不加入筛选 0群发 1昵称 2聊天 3个人说明
  total: 0
};

function handleSearch4(value: string) {
  console.log('fetching search 4', value);
  listParams4.name = value;
  listParams4.page = 1;
  listParams4.total = 0;
  state4.data = [];
  getList4(listParams4);
}
const handleChange4 = (val: any) => {
  console.log(val);
  model.ad_msg_type_params.first_material_id = val;
  console.log(model.ad_msg_type_params.first_material_id);
};
function getList4(params) {
  console.log(params);
  state4.fetching = true;
  fetchGetMaterialList(params)
    .then(res => {
      console.log(res);
      listParams4.total = res.data.total || 0;
      // response.json()
      const list = res.data.list.map(d => ({
        label: d.name,
        value: d.id
        // value: d.file_paths[0]
      }));
      state4.data = listParams4.page == 1 ? list : state4.data.concat(list);
      console.log(state4.data);
    })
    .finally(() => {
      state4.fetching = false;
    });
}

function fetchGroupScroll4() {
  console.log(state4.data.length, listParams4.total);
  if (state4.data.length < listParams4.total && listParams4.page * listParams4.limit < listParams4.total) {
    listParams4.page++;
    getList4(listParams4);
  }
}
function handleFocus4() {
  listParams4.name = '';
  listParams4.page = 1;
  listParams4.total = 0;
  state4.data = [];
  getList4(listParams4);
}

// ad_msg_type_params.material_id
const state5 = reactive({
  data: [],
  // value: undefined,
  fetching: false
});
// watch(state5.value, () => {
//   state5.data = [];
//   state5.fetching = false;
// });
const listParams5 = {
  page: 1,
  limit: 20,
  name: undefined,
  material_type: 1, // 传-1则不加入筛选 0文字 1图片 2链接 3语音 4视频 5头像 6名片
  usage_type: 0, // 传-1则不加入筛选 0群发 1昵称 2聊天 3个人说明
  total: 0
};

function handleSearch5(value: string) {
  console.log('fetching search 5', value);
  listParams5.name = value;
  listParams5.page = 1;
  listParams5.total = 0;
  state5.data = [];
  getList5(listParams5);
}
const handleChange5 = (val: any) => {
  console.log(val);
  model.ad_msg_type_params.material_id = val;
  console.log(model.ad_msg_type_params.material_id);
};
function getList5(params) {
  console.log(params);
  state5.fetching = true;
  fetchGetMaterialList(params)
    .then(res => {
      // console.log(res);
      console.log('res++: ', res);
      listParams5.total = res.data.total || 0;
      // response.json()
      const list = res.data.list.map(d => ({
        label: d.name,
        value: d.id
        // value: d.file_paths[0]
      }));
      state5.data = listParams5.page == 1 ? list : state5.data.concat(list);
      console.log('state5.data: ', state5.data);
    })
    .finally(() => {
      state5.fetching = false;
    });
}

function fetchGroupScroll5() {
  console.log(state5.data.length, listParams5.total);
  if (state5.data.length < listParams5.total && listParams5.page * listParams5.limit < listParams5.total) {
    listParams5.page++;
    getList5(listParams5);
  }
}
function handleFocus5() {
  listParams5.name = '';
  listParams5.page = 1;
  listParams5.total = 0;
  state5.data = [];
  getList5(listParams5);
}

//
function listchange1(val) {
  model.send_contents = [];
  val.map(item => {
    if (item.value) {
      model.send_contents.push(item.value);
    }
    return item;
  });
  console.log('model.send_contents', model.send_contents);
}

function listchange2(val) {
  model.receivers = [];
  val.map(item => {
    if (item.value) {
      model.receivers.push(item.value);
    }
    return item;
  });
  console.log('model.receivers', model.receivers);
}

// 切换招呼类型时，清空招呼语
function greetingTypeChange(val) {
  model.greeting = '';
}

// 初始化抽屉
watch(visible, () => {
  if (visible.value) {
    handleInitModel();
    resetFields();

    activeKey.value = '1';

    listParams1.group_name = '';
    listParams1.page = 1;
    listParams1.total = 0;
    state1.value = [];
    state1.data = [];
    state1.fetching = false;

    listParams2.name = '';
    listParams2.material_type = -1;
    listParams2.page = 1;
    listParams2.total = 0;
    state2.value = [];
    state2.data = [];
    state2.fetching = false;

    listParams3.name = '';
    listParams3.page = 1;
    listParams3.total = 0;
    state3.value = {
      label: '',
      value: ''
    };
    // state3.value = [];
    state3.data = [];
    state3.fetching = false;

    listParams4.name = '';
    listParams4.page = 1;
    listParams4.total = 0;
    state4.value = {
      label: '',
      value: ''
    };
    // state4.value = undefined;
    state4.data = [];
    state4.fetching = false;

    listParams5.name = '';
    listParams5.page = 1;
    listParams4.total = 0;
    state5.value = {
      label: '',
      value: ''
    };
    // state5.value = undefined;
    state5.data = [];
    state5.fetching = false;
  }
});

onMounted(() => {});
</script>

<template>
  <AModal v-model:open="visible" :title="title" :width="800" destroy-on-close @cancel="closeDrawer">
    <AForm ref="formRef" layout="vertical" :model="model" :rules="rules">
      <ATabs v-model:active-key="activeKey">
        <ATabPane key="1" tab="基本设置" force-render>
          <ARow :gutter="24">
            <ACol :span="12">
              <AFormItem label="群发类型" name="mass_msg_type">
                <ASelect
                  v-model:value="model.mass_msg_type"
                  placeholder="请选择群发类型"
                  :options="typeOptions1"
                  clearable
                />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="任务类型" name="task_type">
                <ASelect
                  v-model:value="model.task_type"
                  placeholder="请选择任务类型"
                  :options="typeOptions2"
                  clearable
                />
              </AFormItem>
            </ACol>
            <ACol v-if="model.task_type === 2" :span="12">
              <AFormItem label="招呼类型" name="greeting_type">
                <!-- <AInput v-model:value="model.greeting_type" placeholder="请输入招呼类型" /> -->
                <ASelect
                  v-model:value="model.greeting_type"
                  placeholder="请选择招呼类型"
                  clearable
                  @change="greetingTypeChange"
                >
                  <ASelectOption :value="0">文字</ASelectOption>
                  <ASelectOption :value="1">语音通话</ASelectOption>
                  <ASelectOption :value="2">视频通话</ASelectOption>
                </ASelect>
              </AFormItem>
            </ACol>
            <ACol v-if="model.task_type === 2 && model.greeting_type === 0" :span="12">
              <AFormItem label="招呼语" name="greeting">
                <AInput v-model:value="model.greeting" placeholder="请输入招呼语" />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="发送类型" name="media_type">
                <ASelect
                  v-model:value="model.media_type"
                  placeholder="请选择发送类型"
                  :options="typeOptions3"
                  clearable
                />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="是否添加好友" name="add_contacts">
                <ARadioGroup v-model:value="model.add_contacts" name="radioGroup">
                  <ARadio :value="1">是</ARadio>
                  <ARadio :value="0">否</ARadio>
                </ARadioGroup>
              </AFormItem>
            </ACol>
            <ACol v-if="model.add_contacts === 1" :span="12">
              <AFormItem label="添加好友与发送间隔时间(秒)" name="per_account_send_num">
                <!-- <AInput v-model:value="model.ac_send_interval" placeholder="请输入" /> -->
                <AInputNumber
                  v-model:value="model.ac_send_interval"
                  style="width: 100%"
                  placeholder="请输入添加好友与发送间隔时间(秒)"
                  :precision="0"
                />
              </AFormItem>
            </ACol>
            <ACol v-if="model.mass_msg_type === 2" :span="12">
              <AFormItem label="发送者" name="sender">
                <AInput v-model:value="model.sender" placeholder="请输入发送者" />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="数据包名称" name="data_packet_name">
                <!-- <AInput v-model:value="model.data_packet_name" placeholder="请输入数据包名称" /> -->
                <ASelect
                  v-model:value="state3.value"
                  allow-clear
                  label-in-value
                  placeholder="选择数据包名称"
                  style="width: 100%"
                  :filter-option="false"
                  :not-found-content="state3.fetching ? undefined : '暂无数据'"
                  :options="state3.data"
                  @popup-scroll="fetchGroupScroll3"
                  @change="handleChange3"
                  @focus="handleFocus3"
                >
                  <template v-if="state3.fetching" #notFoundContent>
                    <ASpin size="small" />
                  </template>
                  <template #option="{ value: val, label, count }">
                    <div style="display: flex; justify-content: space-between">
                      <span :aria-label="val">{{ label.length > 16 ? label.slice(0, 16) + '...' : label }}</span>
                      <span>等待数量:{{ count }}</span>
                    </div>
                  </template>
                </ASelect>
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="每个账户发送数量" name="per_account_send_num">
                <!-- <AInput v-model:value="model.per_account_send_num" placeholder="请输入每个账户发送数量" /> -->
                <AInputNumber
                  v-model:value="model.per_account_send_num"
                  style="width: 100%"
                  placeholder="请输入每个账户发送数量"
                  :precision="0"
                />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="每个账户执行间隔(秒)" name="pa_execution_interval">
                <!-- <AInput v-model:value="model.pa_execution_interval" placeholder="请输入每个账户执行间隔(秒)" /> -->
                <AInputNumber
                  v-model:value="model.pa_execution_interval"
                  style="width: 100%"
                  placeholder="请输入每个账户执行间隔(秒)"
                  :precision="0"
                />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="任务描述" name="desc">
                <AInput v-model:value="model.desc" placeholder="请输入任务描述" />
              </AFormItem>
            </ACol>

            <!-- 数组类 -->
            <!--
 发送类型： 1文本 2图片 3语音 4视频 5文件 6扩展 7名片 8通过广告发送消息 9商业扩展文本消息
                素材 显示：  2图片 3语音 4视频 5文件 6扩展 7名片
                发送的消息 显示： 1文本 2图片  8通过广告发送消息 9商业扩展文本消息
                -->
            <ACol v-if="[2, 3, 4, 5, 6, 7].includes(model.media_type)" :span="12">
              <AFormItem label="素材集合" name="material_ids">
                <!-- <AInput v-model:value="model.material_ids" placeholder="请输入" /> -->
                <ASelect
                  v-model:value="state2.value"
                  mode="multiple"
                  label-in-value
                  placeholder="输入素材名称可查询"
                  style="width: 100%"
                  :filter-option="false"
                  :not-found-content="state2.fetching ? undefined : '暂无数据'"
                  :options="state2.data"
                  @search="handleSearch2"
                  @popup-scroll="fetchGroupScroll2"
                  @change="handleChange2"
                  @focus="handleFocus2"
                >
                  <template v-if="state2.fetching" #notFoundContent>
                    <ASpin size="small" />
                  </template>
                </ASelect>
              </AFormItem>
            </ACol>
            <ACol v-if="model.mass_msg_type === 1" :span="12">
              <AFormItem label="发送群组名称集合" name="account_group_names">
                <!-- <AInput v-model:value="model.account_group_names" placeholder="请输入" /> -->
                <ASelect
                  v-model:value="state1.value"
                  mode="multiple"
                  label-in-value
                  placeholder="输入分组名称可查询"
                  style="width: 100%"
                  :filter-option="false"
                  :not-found-content="state1.fetching ? undefined : '暂无数据'"
                  :options="state1.data"
                  @search="handleSearch1"
                  @popup-scroll="fetchGroupScroll1"
                  @change="handleChange1"
                  @focus="handleFocus1"
                >
                  <template v-if="state1.fetching" #notFoundContent>
                    <ASpin size="small" />
                  </template>
                  <template #option="{ value: val, label, count }">
                    <div style="display: flex; justify-content: space-between">
                      <span :aria-label="val">{{ label.length > 20 ? label.slice(0, 20) + '...' : label }}</span>
                      <span>总数量:{{ count }}</span>
                    </div>
                  </template>
                </ASelect>
              </AFormItem>
            </ACol>
            <ACol v-if="!model.data_packet_name" :span="12">
              <AFormItem label="接收用户号码集合" name="receivers">
                <ATextarea
                  v-model:value="model.receiversTextarea"
                  autosize
                  placeholder="请输入接收用户号码,使用换行隔开"
                  allow-clear
                  @change="receiversTextareaChange"
                  @press-enter="receiversTextareaPressEnter"
                />
                <!-- <AInput v-model:value="model.receivers" placeholder="请输入" /> -->
                <!--
 <InputListOperateComponent
                    ref="inputListRef2"
                    @change="listchange2"
                    listName="receivers"
                  />
-->
              </AFormItem>
            </ACol>
            <ACol v-if="[1, 2, 13].includes(model.media_type)" :span="12">
              <AFormItem
                label="发送的消息"
                name="send_contents"
                :rules="[{ required: model.media_type == 2 ? false : true, message: '不能为空', trigger: 'blur' }]"
              >
                <!-- :rules="[{ required: model.media_type==2 ? false : true, message: '不能为空' }]" -->
                <!-- <AInput v-model:value="model.send_contents" placeholder="请输入发送的消息" /> -->
                <InputListOperateComponent ref="inputListRef1" list-name="send_contents" @change="listchange1" />
              </AFormItem>
            </ACol>

            <ACol :span="12">
              <AFormItem label="失败后重试次数" name="retry_count">
                <!-- <AInput v-model:value="model.pa_execution_interval" placeholder="请输入每个账户执行间隔(秒)" /> -->
                <AInputNumber
                  v-model:value="model.retry_count"
                  :precision="0"
                  style="width: 100%"
                  placeholder="请输入失败后重试次数"
                />
              </AFormItem>
            </ACol>
          </ARow>
        </ATabPane>
        <!-- 广告消息类型参数  model.ad_msg_type_params.*  -->
        <ATabPane v-if="model.media_type === 8" key="2" tab="广告消息类型" force-render>
          <ARow :gutter="24">
            <ACol :span="12">
              <AFormItem label="是否为首条消息" :name="['ad_msg_type_params', 'first_message']">
                <!-- 首条消息文字类型 -->
                <ARadioGroup v-model:value="model.ad_msg_type_params.first_message" name="radioGroup">
                  <ARadio :value="true">是</ARadio>
                  <ARadio :value="false">否</ARadio>
                </ARadioGroup>
              </AFormItem>
            </ACol>
          </ARow>
          <ARow v-if="model.ad_msg_type_params.first_message" :gutter="24">
            <ACol :span="12">
              <AFormItem
                label="首条消息顶部是否显示，通过广告发送消息"
                :name="['ad_msg_type_params', 'first_show_ad_attribution']"
              >
                <ARadioGroup v-model:value="model.ad_msg_type_params.first_show_ad_attribution" name="radioGroup">
                  <ARadio :value="true">是</ARadio>
                  <ARadio :value="false">否</ARadio>
                </ARadioGroup>
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="首条消息与第二条消息间隔(秒)" :name="['ad_msg_type_params', 'fs_msg_interval']">
                <!-- <AInput v-model:value="model.ad_msg_type_params.fs_msg_interval" placeholder="请输入" /> -->
                <AInputNumber
                  v-model:value="model.ad_msg_type_params.fs_msg_interval"
                  style="width: 100%"
                  placeholder="请输入"
                  :precision="0"
                />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="首条消息跳转链接" :name="['ad_msg_type_params', 'first_source_url']">
                <AInput v-model:value="model.ad_msg_type_params.first_source_url" placeholder="请输入" />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="首条消息缩略图地址" :name="['ad_msg_type_params', 'first_material_id']">
                <!-- <AInput v-model:value="model.ad_msg_type_params.first_material_id" placeholder="请输入" /> -->
                <ASelect
                  v-model:value="model.ad_msg_type_params.first_material_id"
                  placeholder="输入素材名称可查询"
                  style="width: 100%"
                  show-search
                  :filter-option="false"
                  :not-found-content="state4.fetching ? undefined : '暂无数据'"
                  :options="state4.data"
                  allow-clear
                  @search="handleSearch4"
                  @popup-scroll="fetchGroupScroll4"
                  @change="handleChange4"
                  @focus="handleFocus4"
                >
                  <template v-if="state4.fetching" #notFoundContent>
                    <ASpin size="small" />
                  </template>
                </ASelect>
              </AFormItem>
            </ACol>

            <ACol :span="12">
              <AFormItem
                label="首条消息是否显示大缩略图"
                :name="['ad_msg_type_params', 'first_render_larger_thumbnail']"
              >
                <ARadioGroup v-model:value="model.ad_msg_type_params.first_render_larger_thumbnail" name="radioGroup">
                  <ARadio :value="true">是</ARadio>
                  <ARadio :value="false">否</ARadio>
                </ARadioGroup>
              </AFormItem>
            </ACol>
            <ACol v-if="model.ad_msg_type_params.first_render_larger_thumbnail" :span="12">
              <AFormItem label="首条消息大缩略图链接" :name="['ad_msg_type_params', 'first_thumbnail_url']">
                <AInput v-model:value="model.ad_msg_type_params.first_thumbnail_url" placeholder="请输入" />
              </AFormItem>
            </ACol>

            <ACol :span="12">
              <AFormItem label="首条消息标题" :name="['ad_msg_type_params', 'first_title']">
                <AInput v-model:value="model.ad_msg_type_params.first_title" placeholder="请输入" />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="首条消息正文内容" :name="['ad_msg_type_params', 'first_text']">
                <ATextarea
                  v-model:value="model.ad_msg_type_params.first_text"
                  autosize
                  placeholder="请输入"
                  allow-clear
                />
                <!-- <AInput v-model:value="model..ad_msg_type_paramsfirst_text" placeholder="请输入" /> -->
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="首条消息引用内容" :name="['ad_msg_type_params', 'first_body']">
                <ATextarea
                  v-model:value="model.ad_msg_type_params.first_body"
                  autosize
                  placeholder="请输入"
                  allow-clear
                />
              </AFormItem>
            </ACol>
          </ARow>

          <ARow :gutter="24">
            <!-- 引用的消息 -->
            <ADivider orientation="left" style="height: 2px">引用的消息</ADivider>
            <ACol :span="12">
              <AFormItem label="顶部是否显示，通过广告发送消息" :name="['ad_msg_type_params', 'show_ad_attribution']">
                <ARadioGroup v-model:value="model.ad_msg_type_params.show_ad_attribution" name="radioGroup">
                  <ARadio :value="true">是</ARadio>
                  <ARadio :value="false">否</ARadio>
                </ARadioGroup>
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="跳转链接" :name="['ad_msg_type_params', 'source_url']">
                <AInput v-model:value="model.ad_msg_type_params.source_url" placeholder="请输入" />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="缩略图地址" :name="['ad_msg_type_params', 'material_id']">
                <!-- <AInput v-model:value="model.ad_msg_type_params.material_id" placeholder="请输入" /> -->
                <ASelect
                  v-model:value="model.ad_msg_type_params.material_id"
                  placeholder="输入素材名称可查询"
                  style="width: 100%"
                  show-search
                  :filter-option="false"
                  :not-found-content="state5.fetching ? undefined : '暂无数据'"
                  :options="state5.data"
                  allow-clear
                  @search="handleSearch5"
                  @popup-scroll="fetchGroupScroll5"
                  @change="handleChange5"
                  @focus="handleFocus5"
                >
                  <template v-if="state5.fetching" #notFoundContent>
                    <ASpin size="small" />
                  </template>
                </ASelect>
              </AFormItem>
            </ACol>

            <ACol :span="12">
              <AFormItem label="是否显示大缩略图" :name="['ad_msg_type_params', 'render_larger_thumbnail']">
                <ARadioGroup v-model:value="model.ad_msg_type_params.render_larger_thumbnail" name="radioGroup">
                  <ARadio :value="true">是</ARadio>
                  <ARadio :value="false">否</ARadio>
                </ARadioGroup>
              </AFormItem>
            </ACol>
            <ACol v-if="model.ad_msg_type_params.render_larger_thumbnail" :span="12">
              <AFormItem label="大缩略图链接" :name="['ad_msg_type_params', 'thumbnail_url']">
                <AInput v-model:value="model.ad_msg_type_params.thumbnail_url" placeholder="请输入" />
              </AFormItem>
            </ACol>

            <ACol :span="12">
              <AFormItem label="标题" :name="['ad_msg_type_params', 'title']">
                <AInput v-model:value="model.ad_msg_type_params.title" placeholder="请输入" />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="正文内容" :name="['ad_msg_type_params', 'text']">
                <ATextarea v-model:value="model.ad_msg_type_params.text" autosize placeholder="请输入" allow-clear />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="引用内容" :name="['ad_msg_type_params', 'body']">
                <ATextarea v-model:value="model.ad_msg_type_params.body" autosize placeholder="请输入" allow-clear />
                <!-- <AInput v-model:value="model.ad_msg_type_params.body" placeholder="请输入" /> -->
              </AFormItem>
            </ACol>
          </ARow>
        </ATabPane>

        <!-- 商家消息类型参数  model.bus_msg_type_params.*-->
        <ATabPane v-if="model.media_type === 9" key="3" tab="商家消息类型" force-render>
          <ARow :gutter="24">
            <ACol :span="12">
              <AFormItem label="是否显示转发标识" :name="['bus_msg_type_params', 'is_forwarded']">
                <ARadioGroup v-model:value="model.bus_msg_type_params.is_forwarded" name="radioGroup">
                  <ARadio :value="true">是</ARadio>
                  <ARadio :value="false">否</ARadio>
                </ARadioGroup>
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="是否显示广告标识" :name="['bus_msg_type_params', 'always_show_ad_attribution']">
                <ARadioGroup v-model:value="model.bus_msg_type_params.always_show_ad_attribution" name="radioGroup">
                  <ARadio :value="true">是</ARadio>
                  <ARadio :value="false">否</ARadio>
                </ARadioGroup>
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="商家手机号" :name="['bus_msg_type_params', 'business_owner_jid']">
                <AInput v-model:value="model.bus_msg_type_params.business_owner_jid" placeholder="请输入" />
              </AFormItem>
            </ACol>
            <ACol :span="12">
              <AFormItem label="正文" :name="['bus_msg_type_params', 'text']">
                <ATextarea
                  v-model:value="model.bus_msg_type_params.text"
                  autosize
                  placeholder="请输入正文"
                  allow-clear
                />
                <!-- <AInput v-model:value="model.text" placeholder="请输入" /> -->
              </AFormItem>
            </ACol>
          </ARow>
        </ATabPane>
      </ATabs>
    </AForm>
    <template #footer>
      <ASpace :size="16">
        <AButton @click="closeDrawer">{{ $t('common.cancel') }}</AButton>
        <AButton type="primary" @click="handleSubmit">{{ $t('common.confirm') }}</AButton>
      </ASpace>
    </template>
  </AModal>
</template>

<style scoped></style>
