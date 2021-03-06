<template>
  <div>
    <ListTable :table-config="tableConfig" :header-actions="headerActions" />
    <Dialog
      v-if="dialogVisible"
      :title="this.$t('assets.TestGatewayTestConnection')"
      :visible.sync="dialogVisible"
      width="40%"
      top="35vh"
      :show-confirm="false"
      :show-cancel="false"
      :destroy-on-close="true"
    >
      <el-row :gutter="20">
        <el-col :span="4">
          <div style="line-height: 34px;text-align: center">{{ $t('assets.SshPort') }}</div>
        </el-col>
        <el-col :span="14">
          <el-input v-model="portInput" />
          <span class="help-tips help-block">{{ $t('assets.TestGatewayHelpMessage') }}</span>
        </el-col>
        <el-col :span="4">
          <el-button size="mini" type="primary" style="line-height:20px " :loading="buttonLoading" @click="dialogConfirm">{{ this.$t('common.Confirm') }}</el-button>
        </el-col>
      </el-row>
    </Dialog>
  </div>

</template>

<script>
import ListTable from '@/components/ListTable/index'
import DisplayFormatter from '@/components/ListTable/formatters/DisplayFormatter'
import Dialog from '@/components/Dialog'
export default {
  components: {
    ListTable,
    Dialog
  },
  props: {
    object: {
      type: Object,
      default: () => {}
    }
  },
  data() {
    return {
      tableConfig: {
        url: `/api/v1/assets/gateways/?domain=${this.$route.params.id}`,
        columns: ['name', 'ip', 'port', 'protocol', 'username', 'comment', 'actions'],
        columnsMeta: {
          name: {
            sortable: 'custom',
            formatter: DisplayFormatter
          },
          ip: {
            width: '140px'
          },
          port: {
            width: '60px'
          },
          protocol: {
            sortable: 'custom',
            width: '100px'
          },
          actions: {
            formatterArgs: {
              updateRoute: 'GatewayUpdate',
              performDelete: ({ row, col }) => {
                const id = row.id
                const url = `/api/v1/assets/gateways/${id}/`
                return this.$axios.delete(url)
              },
              extraActions: [
                {
                  name: 'TestConnection',
                  title: this.$t('assets.TestConnection'),
                  callback: function(val) {
                    this.dialogVisible = true
                    if (!val.row.port) {
                      return this.$message.error(this.$t('common.BadRequestErrorMsg'))
                    } else {
                      this.portInput = val.row.port
                      this.cellValue = val.cellValue
                    }
                  }.bind(this)
                }
              ]
            }
          }

        }
      },
      headerActions: {
        hasBulkDelete: false,
        hasSearch: true,
        createRoute: {
          name: 'GatewayCreate',
          query: {
            domain: this.object.id
          }
        }
      },
      dialogVisible: false,
      portInput: '',
      cellValue: '',
      buttonLoading: false
    }
  },
  methods: {
    dialogConfirm() {
      this.buttonLoading = true
      this.$axios.post(`/api/v1/assets/gateways/${this.cellValue}/test-connective/`, { port: this.portInput }).then(
        res => {
          return this.$message.success(this.$t('common.TestSuccessMsg'))
        }
      ).finally(() => {
        this.portInput = ''
        this.cellValue = ''
        this.buttonLoading = false
        this.dialogVisible = false
      }
      )
    }
  }
}
</script>

<style>

</style>
