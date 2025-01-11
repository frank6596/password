<script setup lang="ts">
import { onMounted, reactive, watch } from 'vue';
import { useAntdForm, useFormRules } from '@/hooks/common/form';
import { batchDeleteOfIf } from '@/service/api';
import { $t } from '@/locales';

defineOptions({
  name: 'DeleteOfIfOperateDrawer'
});

interface Emits {
  (e: 'submitted'): void;
}

const emit = defineEmits<Emits>();

const visible = defineModel<boolean>('visible', {
  default: false
});

const { formRef, validate, resetFields } = useAntdForm();
const { defaultRequiredRule } = useFormRules();

type Model = {
  mass_msg_type: number | undefined;
};

const model: Model = reactive(createDefaultModel());

function createDefaultModel(): Model {
  return {
    mass_msg_type: undefined
  };
}

type RuleKey = Extract<keyof Model, 'mass_msg_type'>;

const rules: Record<RuleKey, App.Global.FormRule> = {
  mass_msg_type: defaultRequiredRule
};

function closeDrawer() {
  visible.value = false;
}

async function handleSubmit() {
  await validate();

  console.log(model);
  const { error } = await batchDeleteOfIf(model);
  if (!error) {
    window.$message?.success('删除成功');
    closeDrawer();
    emit('submitted');
  }
}

async function handleInitModel() {
  Object.assign(model, createDefaultModel());
}

// 初始化抽屉
watch(visible, () => {
  if (visible.value) {
    handleInitModel();
    resetFields();
  }
});

onMounted(() => {});
</script>

<template>
  <ADrawer v-model:open="visible" :title="title" :width="360">
    <AForm ref="formRef" layout="vertical" :model="model" :rules="rules">
      <AFormItem label="群发类型" name="mass_msg_type">
        <ARadioGroup v-model:value="model.mass_msg_type">
          <ARadio :value="0">全部</ARadio>
          <ARadio :value="1">普通类型</ARadio>
          <ARadio :value="2">新用户类型</ARadio>
        </ARadioGroup>
      </AFormItem>
    </AForm>
    <template #footer>
      <ASpace :size="16">
        <AButton @click="closeDrawer">{{ $t('common.cancel') }}</AButton>
        <AButton type="primary" @click="handleSubmit">{{ $t('common.confirm') }}</AButton>
      </ASpace>
    </template>
  </ADrawer>
</template>

<style scoped></style>
