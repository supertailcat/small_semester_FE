<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="操作类型" prop="operationType">
      <el-input v-model="dataForm.operationType" placeholder="操作类型"></el-input>
    </el-form-item>
    <el-form-item label="操作时间" prop="operationDate">
      <el-input v-model="dataForm.operationDate" placeholder="操作时间"></el-input>
    </el-form-item>
    <el-form-item label="操作人员" prop="guanliyuanId">
      <el-input v-model="dataForm.guanliyuanId" placeholder="操作人员"></el-input>
    </el-form-item>
    <el-form-item label="操作描述" prop="operationDescription">
      <el-input v-model="dataForm.operationDescription" placeholder="操作描述"></el-input>
    </el-form-item>
    <el-form-item label="操作对象类型" prop="operationDxType">
      <el-input v-model="dataForm.operationDxType" placeholder="操作对象类型"></el-input>
    </el-form-item>
    <el-form-item label="操作对象id" prop="operationDxId">
      <el-input v-model="dataForm.operationDxId" placeholder="操作对象id"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          operationType: '',
          operationDate: '',
          guanliyuanId: '',
          operationDescription: '',
          operationDxType: '',
          operationDxId: ''
        },
        dataRule: {
          operationType: [
            { required: true, message: '操作类型不能为空', trigger: 'blur' }
          ],
          operationDate: [
            { required: true, message: '操作时间不能为空', trigger: 'blur' }
          ],
          guanliyuanId: [
            { required: true, message: '操作人员不能为空', trigger: 'blur' }
          ],
          operationDescription: [
            { required: true, message: '操作描述不能为空', trigger: 'blur' }
          ],
          operationDxType: [
            { required: true, message: '操作对象类型不能为空', trigger: 'blur' }
          ],
          operationDxId: [
            { required: true, message: '操作对象id不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/guanliyuan/operationrecord/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.operationType = data.operationRecord.operationType
                this.dataForm.operationDate = data.operationRecord.operationDate
                this.dataForm.guanliyuanId = data.operationRecord.guanliyuanId
                this.dataForm.operationDescription = data.operationRecord.operationDescription
                this.dataForm.operationDxType = data.operationRecord.operationDxType
                this.dataForm.operationDxId = data.operationRecord.operationDxId
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/guanliyuan/operationrecord/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'operationType': this.dataForm.operationType,
                'operationDate': this.dataForm.operationDate,
                'guanliyuanId': this.dataForm.guanliyuanId,
                'operationDescription': this.dataForm.operationDescription,
                'operationDxType': this.dataForm.operationDxType,
                'operationDxId': this.dataForm.operationDxId
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
