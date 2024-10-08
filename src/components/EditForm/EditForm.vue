<template>
  <!-- 抽屉模式 -->
  <el-drawer
    v-if="mode === 'drawer'"
    :size="globalStore.sizeGet"
    :fullscreen="globalStore.isMobile"
    ref="ElRef"
    append-to-body
    :title="`${editProps.title}`"
    v-model="visible"
    :destroy-on-close="true"
    v-bind="editProps.attrs">
    <template #header>
      <slot name="header" />
    </template>

    <slot name="top" v-bind="editProps" />

    <slot v-bind="editProps">
      <basic-form
        v-if="flush"
        ref="formRef"
        :model="editProps.row"
        :schema="editProps.formSchema!"
        v-bind="editProps.formAttrs" />
    </slot>
    <template #footer>
      <slot name="footer">
        <el-button @click="close">取消</el-button>
        <el-button type="primary"  v-throttle="onSubmit">确定</el-button>
      </slot>
    </template>
  </el-drawer>

  <!-- 弹窗模式 -->
  <el-dialog
    v-else
    class="rounded-11px!"
    ref="ElRef"
    append-to-body
    :width="globalStore.sizeGet"
    :fullscreen="globalStore.isMobile"
    :title="`${editProps.title}`"
    v-model="visible"
    :destroy-on-close="true"
    v-bind="editProps.attrs">
    <template #header>
      <slot name="header" />
    </template>
    <slot name="top" v-bind="editProps" />
    <slot v-bind="editProps">
      <basic-form
        v-if="flush"
        ref="formRef"
        :model="editProps.row"
        :schema="editProps.formSchema!"
        v-bind="editProps.formAttrs" />
    </slot>
    <template #footer>
      <slot name="footer">
        <el-button @click="close">取消</el-button>
        <el-button type="primary" v-throttle="onSubmit">确定</el-button>
      </slot>
    </template>
  </el-dialog>
</template>

<script setup lang="ts">
import { BasicForm, BasicDrawer, IEditForm } from '@/components'
import { useGlobalStore } from '@/stores'

const globalStore = useGlobalStore()

withDefaults(defineProps<{ mode?: string }>(), {
  mode: 'drawer'
})

const editProps = ref<IEditForm>({
  title: '',
  formSchema: [],
  row: {}
})

const emit = defineEmits<{ (e: 'onSubmit', params: IEditForm, valid: boolean): void }>()

const ElRef = shallowRef<InstanceType<typeof BasicDrawer>>()
const formRef = shallowRef<InstanceType<typeof BasicForm>>()

// 刷新表单组件的标识
const flush = ref(false)
// 控制弹窗打开或关闭
const visible = ref(false)
// loading
const loading = ref(false)

// 接收父组件传过来的参数
const accept = async (params: IEditForm) => {
  Object.assign(editProps.value, params)
  visible.value = true
  // 当表单域配置项（editProps.formSchema）有改变时会自动触发校验，比如菜单管理页面点击新增后切换菜单类型。使用下面方式刷新dom来解决该问题
  flush.value = false
  nextTick(() => {
    flush.value = true
  })
}

// 确认提交
const onSubmit = async () => {
  if (formRef.value) {
    formRef.value.validate(async (valid: any) => {
      emit('onSubmit', editProps.value, valid)
    })
  } else {
    emit('onSubmit', editProps.value, true)
  }
}

const close = () => {
  visible.value = false
}

const resetFields = () => {
  formRef?.value?.resetFields()
}

defineExpose({ accept, close, resetFields, loading })
</script>

<style scoped lang="scss"></style>
