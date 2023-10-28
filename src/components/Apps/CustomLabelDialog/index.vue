<template>
  <Dialog
    v-if="iVisible"
    :destroy-on-close="true"
    :show-cancel="false"
    :show-confirm="false"
    :title="$tc('assets.UpdateCustomLabel')"
    :visible.sync="iVisible"
    top="35vh"
    width="40%"
  >
    <el-row :gutter="20">
      <el-col :md="4" :sm="24">
        <div style="line-height: 34px">{{ $t('assets.CustomLabel') }}</div>
      </el-col>
      <el-col :md="15" :sm="24">
        <el-input v-model="value" />
      </el-col>
      <el-col :md="4" :sm="24">
        <el-button
          :loading="loading"
          size="mini"
          style="line-height:20px "
          type="primary"
          @click="dialogConfirm"
        >
          {{ this.$t('common.Confirm') }}
        </el-button>
      </el-col>
    </el-row>
  </Dialog>
</template>

<script>
import Dialog from '@/components/Dialog/index.vue'

export default {
  name: 'CustomLabelDialog',
  components: {
    Dialog
  },
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    loading: {
      type: Boolean,
      default: false
    },
    value: {
      type: String,
      default: ''
    },
    cell: {
      type: String,
      default: ''
    },
    assets: {
      type: [Array],
      default: () => []
    }
  },
  data() {
    return {}
  },
  computed: {
    iVisible: {
      get() {
        return this.visible
      },
      set(val) {
        this.$emit('update:visible', val)
      }
    }
  },
  methods: {
    dialogConfirm() {
      if (this.cell !== '') {
        const url = `/api/v1/assets/custom-labels/${this.cell}/`
        const data = { id: this.cell, value: this.value, assets: this.assets }
        this.$axios.patch(url, data).then(() => {
          this.$emit('update', true)
          this.$message.success(this.$tc('common.updateSuccessMsg'))
        }).finally(() => {
          this.iVisible = false
        })
      } else {
        const url = `/api/v1/assets/custom-labels/`
        const data = { value: this.value, assets: this.assets }
        this.$axios.post(url, data).then(() => {
          this.$emit('update', true)
          this.$message.success(this.$i18n.t('common.updateSuccessMsg'))
        }).finally(() => {
          this.iVisible = false
        })
      }
    }
  }
}
</script>
