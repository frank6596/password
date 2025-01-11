<script setup lang="ts">
import { computed, ref } from 'vue';
import { $t } from '@/locales';
import { useAntdForm, useFormRules } from '@/hooks/common/form';
import { enableStatusOptions, userGenderOptions } from '@/constants/business';
import { translateOptions } from '@/utils/common';

defineOptions({
  name: 'DetailsListSearch'
});

interface Emits {
  (e: 'reset'): void;
  (e: 'search'): void;
}

const emit = defineEmits<Emits>();

const { formRef, validate, resetFields } = useAntdForm();

const model = defineModel('model', { required: true });

type Model = {
  id: string,
  status: number,
  receiver: string,
  sender: string
};
type RuleKey = Extract<keyof Model, 'id' | 'status' | 'sender' |  'receiver'>


const rules = computed<Record<RuleKey, App.Global.FormRule>>(() => {
  // const { patternRules } = useFormRules(); // inside computed to make locale reactive

  return {
  };
});

//状态 传-1则不加入筛选 0等待中 1运行中 2成功 3失败 4超时
const statusOptions = ref([
  {
    value: -1,
    label: '全部',
  },
  {
    value: 0,
    label: '等待中',
  },
  {
    value: 1,
    label: '运行中',
  },
  {
    value: 2,
    label: '成功',
  },
  {
    value: 3,
    label: '失败',
  },
  {
    value: 4,
    label: '超时',
  },
]);

async function reset() {
  await resetFields();
  emit('reset');
}

async function search() {
  await validate();
  emit('search');
}
const activeKey = ref(['1']);
</script>

<template>
<a-collapse v-model:activeKey="activeKey" style="background-color: #fff;" ghost>
  <a-collapse-panel key="1" :header="$t('common.search')">
    <AForm
      ref="formRef"
      :model="model"
      :rules="rules"
      :label-col="{
        span: 5,
        md: 7
      }"
    >
      <ARow :gutter="[16, 16]" wrap>
        <!-- <ACol :span="24" :md="12" :lg="6">
          <AFormItem label="任务id" name="id" class="m-0">
            <AInput v-model:value="model.id" placeholder="请输入任务id" />
          </AFormItem>
        </ACol> -->
        <ACol :span="24" :md="12" :lg="12">
          <AFormItem label="发送者" name="sender" class="m-0">
            <AInput v-model:value="model.sender" placeholder="请输入发送者" />
          </AFormItem>
        </ACol>
        <ACol :span="24" :md="12" :lg="12">
          <AFormItem label="接收用户号码" name="receiver" class="m-0">
            <AInput v-model:value="model.receiver" placeholder="请输入接收用户号码" />
          </AFormItem>
        </ACol>
        <ACol :span="24" :md="12" :lg="12">
          <AFormItem label="状态" name="status" class="m-0">
            <ASelect
              v-model:value="model.status"
              placeholder="请选择状态"
              :options="statusOptions"
              clearable
            />
          </AFormItem>
        </ACol>
        <div class="flex-1">
          <AFormItem class="m-0">
            <div class="w-full flex-y-center justify-end gap-12px">
              <AButton @click="reset">
                <template #icon>
                  <icon-ic-round-refresh class="align-sub text-icon" />
                </template>
                <span class="ml-8px">{{ $t('common.reset') }}</span>
              </AButton>
              <AButton type="primary" ghost @click="search">
                <template #icon>
                  <icon-ic-round-search class="align-sub text-icon" />
                </template>
                <span class="ml-8px">{{ $t('common.search') }}</span>
              </AButton>
            </div>
          </AFormItem>
        </div>
      </ARow>
    </AForm>
  </a-collapse-panel>
</a-collapse>
</template>

<style scoped></style>
