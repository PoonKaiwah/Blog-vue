<!--
 * @Author: Gaiwa 13012265332@163.com
 * @Date: 2023-10-30 23:22:44
 * @LastEditors: Gaiwa 13012265332@163.com
 * @LastEditTime: 2023-11-19 00:15:14
 * @FilePath: \vue-blog\src\components\base\BaseModal.vue
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
<template>
  <div>
    <el-dialog
      :title="title"
      :visible.sync="isShow"
      :width="width"
      :before-close="close"
      class="blog-modal"
    >
      <BaseForm :type="type" v-if="formType" ref="form" />
      <div slot="footer" class="dialog-footer">
        <el-button
          v-for="btn in btns"
          :key="btn.targetName"
          @click="handlerBtn(btn.targetName, btn.isSubmit)"
          class="blog-modal--btn"
          >{{ btn.name }}
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import MODAL_DATA from "@/config/modal.config";
import { createNamespacedHelpers } from "vuex";
import BaseForm from "./BaseForm";
const { mapState, mapActions } = createNamespacedHelpers("modal");
export default {
  name: "BaseModal",
  components: {
    BaseForm,
  },
  data() {
    return {
      modalBtnName: "btnName",
    };
  },
  computed: {
    ...mapState(["isShow", "type"]),
    title() {
      return MODAL_DATA[this.type]?.title ?? "默认modal";
    },
    width() {
      return MODAL_DATA[this.type]?.width ?? "60%";
    },
    formType() {
      return MODAL_DATA[this.type]?.formType;
    },
    needUpdate() {
      return MODAL_DATA[this.type]?.needUpdate;
    },
    btns() {
      return (
        MODAL_DATA[this.type]?.btns ?? [
          {
            targetName: "close",
            name: "取消",
          },
          {
            targetName: "confirm",
            name: "提交",
          },
        ]
      );
    },
  },
  methods: {
    handlerBtn(action, isSubmit) {
      if (isSubmit) {
        this.submitForm();
        return;
      }
      this[action] && this[action]();
    },
    submitForm() {
      if (!this.formType) {
        return false;
      }
      let refForm = this.$refs["form"];
      refForm.$refs["elForm"].validate(async (valid) => {
        if (valid) {
          try {
            await this.$api({ type: this.formType, data: refForm.form });
            this.close();
            if (this.needUpdate) {
              this.$EventBus.$emit("updateView");
            }
          } catch (err) {
            refForm.$refs["elForm"].resetFields();
          }
        } else {
          return false;
        }
      });
    },
    ...mapActions(["close", "open", "confirm"]),
  },
};
</script>

<style lang="stylus">
@import '@/assets/css/base.styl';

.blog-modal .el-input__inner {
  font-size: font-size-h;
  font-weight: normal;
  border: 1px solid theme-color;

  &::placeholder {
    font-weight: 300;
  }
}

.blog-modal .blog-modal--btn {
  &:not(:last-of-type) {
    margin-right: margin-space;
  }
}
</style>