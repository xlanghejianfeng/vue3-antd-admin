<template>
  <a-modal
      title="新增账号"
      v-model:visible="visible"
      :confirm-loading="confirmLoading"
      :afterClose="remove"
      @ok="handleOk"
  >
    <schema-form ref="dynamicForm" :dynamic-validate-form="dynamicValidateForm" />
  </a-modal>
</template>

<script lang="ts">
import {defineComponent, reactive, toRefs, onMounted, ref} from 'vue'
import {Modal} from 'ant-design-vue'
import {addSchema} from "./add-schema";
import {useAsync} from "@/hooks";
import {SchemaForm} from '@/components/JSON-schema-form/'
import {postAdminAccount} from "@/api/system/account";

export default defineComponent({
  name: "add-modal",
  components: { [Modal.name]: Modal, SchemaForm},
  props: {
    remove: { // 移除模态框
      type: Function
    },
    callback: {
      type: Function
    }
  },
  setup(props) {
    const dynamicForm = ref<any>(null)

    const state = reactive({
      visible: true,
      confirmLoading: false,
      dynamicValidateForm: addSchema
    })

    const handleOk = () => {
      state.confirmLoading = true;
      dynamicForm.value.validate()
          .then( async res => {
            const param = {
              ...dynamicForm.value.modelRef,
              roles: dynamicForm.value.modelRef.roles.toString()
            }
            await useAsync(postAdminAccount(param), {ref: state, loadingName: 'confirmLoading'})
            state.visible = false;
            props.callback && props.callback()
          })
          .catch(err => {
            console.log('error', err);
            state.confirmLoading = false
          });
    }

    return {
      ...toRefs(state),
      handleOk,
      dynamicForm,
    }
  }
})
</script>

<style scoped>
</style>
