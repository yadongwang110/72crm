<template>
  <el-dialog :visible.sync="showDialog"
             :title="'导入'+crmTypeName"
             width="550px"
             :append-to-body="true"
             @close="closeView">
    <div class="dialog-body">
      <div class="sections">
        <div>一、请按照数据模板的格式准备要导入的数据。<span class="download"
                @click="download">点击下载</span>《{{crmTypeName}}导入模板》</div>
        <div class="content content-tips">
          <div>注意事项：</div>
          <div>1、模板中的表头名称不能更改，表头行不能删除</div>
          <div>2、其中标*为必填项，必须填写</div>
          <div>3、导入文件请勿超过20MB</div>
        </div>
      </div>
      <div class="sections">
        <div>二、请选择数据重复时的处理方式（查重规则：【{{crmTypeName}}名称】）</div>
        <div class="content">
          <el-select v-model="config"
                     placeholder="请选择">
            <el-option v-for="(item, index) in [{name: '覆盖系统原有数据',value: 1},{name: '跳过',value: 0}]"
                       :key="index"
                       :label="item.name"
                       :value="item.value">
            </el-option>
          </el-select>
        </div>
      </div>
      <div class="sections">
        <div>三、请选择需要导入的文件</div>
        <div class="content">
          <flexbox class="file-select">
            <el-input v-model="file.name"
                      :disabled="true"></el-input>
            <el-button type="primary"
                       @click="selectFile">选择文件</el-button>
          </flexbox>
        </div>
      </div>
      <div class="sections">
        <div>四、请选择负责人（{{crmType == 'leads' ? '必选' : '如不选择，导入的客户将进入公海'}}）</div>
        <div class="content">
          <div class="user-cell">
            <xh-user-cell :value="user"
                          @value-change="userSelect"></xh-user-cell>
          </div>
        </div>
      </div>
      <input type="file"
             id="importInputFile"
             accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel"
             @change="uploadFile">
    </div>
    <span slot="footer"
          class="dialog-footer">
      <el-button @click="closeView">取 消</el-button>
      <el-button type="primary"
                 @click="sure">确 定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import { mapGetters } from 'vuex'
import {
  crmCustomerExcelImport,
  crmCustomerExcelDownload,
  crmCustomerExcelDownloadURL
} from '@/api/customermanagement/customer'
import {
  crmLeadsExcelImport,
  crmLeadsExcelDownload,
  crmLeadsExcelDownloadURL
} from '@/api/customermanagement/clue'
import { XhUserCell } from '@/components/CreateCom'
import { Loading } from 'element-ui'

export default {
  name: 'c-r-m-import', // 文件导入
  components: {
    XhUserCell
  },
  data() {
    return {
      loading: false,
      showDialog: false,
      config: 1, // 	1 覆盖，0跳过
      file: { name: '' },
      user: []
    }
  },
  computed: {
    ...mapGetters(['userInfo']),
    crmTypeName() {
      if (this.crmType == 'customer') {
        return '客户'
      } else if (this.crmType == 'leads') {
        return '线索'
      }
      return ''
    }
  },
  props: {
    show: {
      type: Boolean,
      default: false
    },
    // CRM类型
    crmType: {
      type: String,
      default: ''
    }
  },
  watch: {
    show: function(val) {
      this.showDialog = val
    }
  },
  mounted() {
    this.user.push(this.userInfo)
  },
  methods: {
    sure() {
      var params = {}
      if (!this.file.name) {
        this.$message.error('请选择导入文件')
      } else if (
        this.crmType == 'leads' &&
        (!this.user || this.user.length == 0)
      ) {
        this.$message.error('请选择负责人')
      } else {
        params.config = this.config
        params.file = this.file
        params.owner_user_id = this.user.length > 0 ? this.user[0].id : ''
        var request
        if (this.crmType == 'customer') {
          request = crmCustomerExcelImport
        } else if (this.crmType == 'leads') {
          request = crmLeadsExcelImport
        }
        let loading = Loading.service({ fullscreen: true })
        request(params)
          .then(res => {
            loading.close()
            this.closeView()
          })
          .catch(() => {
            loading.close()
          })
      }
    },
    // 下载模板操作
    download() {
      var a = document.createElement('a')
      if (this.crmType == 'customer') {
        a.href = window.BASE_URL + crmCustomerExcelDownloadURL
      } else if (this.crmType == 'leads') {
        a.href = window.BASE_URL + crmLeadsExcelDownloadURL
      }
      a.target = '_black'
      document.body.appendChild(a)
      a.click()
      document.body.removeChild(a)
    },
    // 选择文件
    selectFile() {
      document.getElementById('importInputFile').click()
    },
    /** 图片选择出发 */
    uploadFile(event) {
      var files = event.target.files
      const file = files[0]
      this.file = file
      event.target.value = ''
    },
    // 用户选择
    userSelect(data) {
      if (data.value && data.value.length > 0) {
        this.user = data.value
      } else {
        this.user = []
      }
    },
    // 关闭操作
    closeView() {
      this.$emit('close')
    }
  }
}
</script>

<style scoped lang="scss">
.sections {
  font-size: 14px;
  .download {
    cursor: pointer;
    color: #3e84e9;
  }
}

.content {
  padding: 10px 10px 10px 30px;
  .el-select {
    width: 300px;
  }
  .user-cell {
    width: 300px;
  }
}

.content-tips {
  font-size: 12px;
  color: #a9a9a9;
  line-height: 15px;
}

#importInputFile {
  display: none;
}

.file-select {
  .el-input {
    width: 300px;
  }
  button {
    margin-left: 20px;
  }
}
</style>
