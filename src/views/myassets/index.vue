<template>
  <div>
    <el-col :md="24" :sm="24">
      <GenericTreeListPage
        ref="GenericTreeListPage"
        :header-actions="headerActions"
        :table-config="tableConfig"
        :tree-setting="treeSetting"
      />
      <CustomLabelDialog
        :cell="cell"
        :value="value"
        :assets="assets"
        :visible.sync="visible"
        @update="updateSuccess"
      />
    </el-col>
  </div>
</template>

<script>
import GenericTreeListPage from '@/layout/components/GenericTreeListPage'
import { AccountShowFormatter, DialogDetailFormatter } from '@/components/Table/TableFormatters'
import CustomLabelDialog from '@/components/Apps/CustomLabelDialog'

export default {
  components: {
    GenericTreeListPage,
    CustomLabelDialog
  },
  data() {
    return {
      allFavorites: [],
      treeSetting: {
        showMenu: false,
        showRefresh: true,
        showAssets: false,
        url: '/api/v1/perms/users/self/users/assets/',
        nodeUrl: '/api/v1/perms/users/self/nodes/',
        // ?assets=0不显示资产. =1显示资产
        treeUrl: '/api/v1/perms/users/self/nodes/children/tree/',
        callback: {
          refresh: () => {},
          onSelected: function(event, treeNode) {
            if (treeNode.meta.type === 'node') {
              const currentNodeId = treeNode.meta.data.id
              this.tableConfig.url = `/api/v1/perms/users/self/nodes/${currentNodeId}/assets/?cache_policy=1`
            }
          }.bind(this)
        }
      },
      tableConfig: {
        url: '/api/v1/perms/users/self/assets/',
        hasTree: true,
        columnsExclude: ['spec_info'],
        columnsShow: {
          default: ['name', 'address', 'platform', 'accounts', 'custom_labels', 'connectivity', 'is_active', 'actions'],
          min: ['name', 'address', 'actions']
        },
        columns: [
          'name', 'address', 'domain', 'platform', 'connectivity', 'is_active',
          'nodes', 'org_name', 'created_by', 'labels', 'custom_labels', 'accounts', 'comment', 'actions'
        ],
        columnsMeta: {
          name: {
            prop: 'name',
            label: this.$t('assets.Name'),
            formatter: DialogDetailFormatter,
            formatterArgs: {
              getDialogTitle: function({ col, row, cellValue }) { this.$t('assets.AssetDetail') }.bind(this),
              getDetailItems: function({ col, row, cellValue }) {
                return [
                  {
                    key: this.$t('assets.Name'),
                    value: row.name
                  },
                  {
                    key: this.$t('assets.AssetAddress'),
                    value: row.address
                  },
                  {
                    key: this.$t('assets.Protocols'),
                    value: row.protocols.map(item => item.name).join(', ')
                  },
                  {
                    key: this.$t('assets.Category'),
                    value: row.category.label
                  },
                  {
                    key: this.$t('assets.Type'),
                    value: row.type.label
                  },
                  {
                    key: this.$t('assets.Platform'),
                    value: row.platform?.name || ''
                  },
                  {
                    key: this.$t('common.Active'),
                    value: row.is_active
                  },
                  {
                    key: this.$t('assets.Comment'),
                    value: row.comment
                  }
                ]
              }.bind(this)
            },
            sortable: true
          },
          address: {
            sortable: 'custom',
            width: '150px'
          },
          accounts: {
            align: 'center',
            label: this.$t('assets.Account'),
            width: '120px',
            formatter: AccountShowFormatter,
            formatterArgs: {
              getUrl: ({ row }) => {
                return `/api/v1/perms/users/self/assets/${row.id}/accounts/?cache_policy=1`
              }
            }
          },
          custom_labels: {
            label: this.$t('assets.CustomLabel'),
            name: 'custom_labels',
            width: '100px'
          },
          platform: {
            width: '120px'
          },
          comment: {
            width: '100px'
          },
          actions: {
            width: '150px',
            align: 'center',
            formatterArgs: {
              hasDelete: false,
              loading: true,
              hasClone: false,
              hasUpdate: false,
              extraActions: [
                {
                  name: 'connect',
                  fa: 'fa-terminal',
                  type: 'primary',
                  can: ({ row }) => row.is_active,
                  callback: ({ row }) => {
                    window.open(`/luna/?login_to=${row.id}`, '_blank')
                  }
                },
                {
                  name: 'favor',
                  type: 'info',
                  fa: ({ row }) => {
                    return this.checkFavorite(row.id) ? 'fa-star' : 'fa-star-o'
                  },
                  callback: ({ row }) => this.toggleFavorite(row.id)
                },
                {
                  name: 'Update',
                  fa: 'fa-edit',
                  type: 'primary',
                  // can: this.$hasPerm('assets.updateCustomLabel') && !this.$store.getters.currentOrgIsRoot,
                  callback: function({ row }) {
                    this.cell = ''
                    this.value = ''
                    this.visible = true
                    this.assets = [row.id]
                    const custom_labels = row.custom_labels
                    if (custom_labels.length !== 0) {
                      const custom_label = custom_labels[0]
                      this.value = custom_label.value
                      this.cell = custom_label.id
                    }
                  }.bind(this)
                }
              ]
            }
          }
        },
        tableAttrs: {
          rowClassName({ row }) {
            return !row.is_active ? 'row_disabled' : ''
          }
        }
      },
      headerActions: {
        hasExport: false,
        hasImport: false,
        hasLeftActions: false,
        hasSearch: true
      },
      cell: '',
      value: '',
      assets: [],
      visible: false
    }
  },
  computed: {
    treeTable() {
      return this.$refs.GenericTreeListPage.$refs.TreeTable
    }
  },
  watch: {},
  mounted() {
    this.refreshAllFavorites()
  },
  methods: {
    refreshAllFavorites() {
      const formatterArgs = this.tableConfig.columnsMeta.actions.formatterArgs
      formatterArgs.loading = true
      this.$axios.get('/api/v1/assets/favorite-assets/').then(resp => {
        this.allFavorites = resp
        formatterArgs.loading = false
      })
    },
    favor(assetId) {
      const data = { asset: assetId }
      const url = '/api/v1/assets/favorite-assets/'
      this.$axios.post(url, data).then(() => {
        this.allFavorites.push({ asset: assetId })
        this.$message.success(this.$i18n.t('common.CollectionSucceed'))
      })
    },
    disfavor(assetId) {
      const url = `/api/v1/assets/favorite-assets/?asset=${assetId}`
      this.$axios.delete(url).then(() => {
        this.allFavorites = this.allFavorites.filter(item => item['asset'] !== assetId)
        this.$message.success(this.$i18n.t('common.CancelCollection'))
      })
    },
    toggleFavorite(assetId) {
      const favorite = this.checkFavorite(assetId)
      if (favorite) {
        this.disfavor(assetId)
      } else {
        this.favor(assetId)
      }
    },
    checkFavorite(assetId) {
      let ok = false
      this.allFavorites.forEach(element => {
        if (element['asset'] === assetId) {
          ok = true
        }
      })
      return ok
    },
    updateSuccess() {
      this.treeTable.forceRerender()
    }
  }
}
</script>

<style>
  .el-card {
    border: 0 !important;
  }
  .row_disabled,.row_disabled:hover,.row_disabled:hover > td{
    cursor: not-allowed;
    background-color:rgba(192,196,204,0.28) !important;
  }

  .link-more {
    margin-left: 10px;
    border-bottom: solid 1px;
    font-size: 12px
  }
</style>
