<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="朋友类型" prop="friendType">
      <el-input v-model="dataForm.friendType" placeholder="朋友类型"></el-input>
    </el-form-item>
    <el-form-item label="朋友id" prop="friendId">
      <el-input v-model="dataForm.friendId" placeholder="朋友id"></el-input>
    </el-form-item>
    <el-form-item label="朋友关系" prop="friendship">
      <el-input v-model="dataForm.friendship" placeholder="朋友关系"></el-input>
    </el-form-item>
    <el-form-item label="老人id" prop="laorenId">
      <el-input v-model="dataForm.laorenId" placeholder="老人id"></el-input>
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
          friendType: '',
          friendId: '',
          friendship: '',
          laorenId: ''
        },
        dataRule: {
          friendType: [
            { required: true, message: '朋友类型不能为空', trigger: 'blur' }
          ],
          friendId: [
            { required: true, message: '朋友id不能为空', trigger: 'blur' }
          ],
          friendship: [
            { required: true, message: '朋友关系不能为空', trigger: 'blur' }
          ],
          laorenId: [
            { required: true, message: '老人id不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/laoren/laorenfriend/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.friendType = data.laorenFriend.friendType
                this.dataForm.friendId = data.laorenFriend.friendId
                this.dataForm.friendship = data.laorenFriend.friendship
                this.dataForm.laorenId = data.laorenFriend.laorenId
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
              url: this.$http.adornUrl(`/laoren/laorenfriend/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'friendType': this.dataForm.friendType,
                'friendId': this.dataForm.friendId,
                'friendship': this.dataForm.friendship,
                'laorenId': this.dataForm.laorenId
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
