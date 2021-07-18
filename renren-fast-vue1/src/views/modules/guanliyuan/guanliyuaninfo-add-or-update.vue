<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="姓名" prop="name">
      <el-input v-model="dataForm.name" placeholder="姓名"></el-input>
    </el-form-item>
    <el-form-item label="性别" prop="gender">
      <el-input v-model="dataForm.gender" placeholder="性别"></el-input>
    </el-form-item>
    <el-form-item label="电话" prop="phone">
      <el-input v-model="dataForm.phone" placeholder="电话"></el-input>
    </el-form-item>
    <el-form-item label="图像路径" prop="imgsetDir">
      <el-input v-model="dataForm.imgsetDir" placeholder="图像路径"></el-input>
    </el-form-item>
    <el-form-item label="头像路径" prop="profilePhoto">
      <el-input v-model="dataForm.profilePhoto" placeholder="头像路径"></el-input>
    </el-form-item>
    <el-form-item label="房间号" prop="roomNumber">
      <el-input v-model="dataForm.roomNumber" placeholder="房间号"></el-input>
    </el-form-item>
    <el-form-item label="描述" prop="description">
      <el-input v-model="dataForm.description" placeholder="描述"></el-input>
    </el-form-item>
    <el-form-item label="删除标志" prop="isRemove">
      <el-input v-model="dataForm.isRemove" placeholder="删除标志"></el-input>
    </el-form-item>
    <el-form-item label="用户名" prop="guanliyuanUsername">
      <el-input v-model="dataForm.guanliyuanUsername" placeholder="用户名"></el-input>
    </el-form-item>
    <el-form-item label="密码" prop="guanliyuanPassword">
      <el-input v-model="dataForm.guanliyuanPassword" placeholder="密码"></el-input>
    </el-form-item>
    <el-form-item label="邮箱" prop="guanliyuanEmail">
      <el-input v-model="dataForm.guanliyuanEmail" placeholder="邮箱"></el-input>
    </el-form-item>
    <el-form-item label="移动电话" prop="guanliyuanMobile">
      <el-input v-model="dataForm.guanliyuanMobile" placeholder="移动电话"></el-input>
    </el-form-item>
    <el-form-item label="主题" prop="theme">
      <el-input v-model="dataForm.theme" placeholder="主题"></el-input>
    </el-form-item>
    <el-form-item label="缺省页面" prop="defaultpage">
      <el-input v-model="dataForm.defaultpage" placeholder="缺省页面"></el-input>
    </el-form-item>
    <el-form-item label="Logo" prop="logoimage">
      <el-input v-model="dataForm.logoimage" placeholder="Logo"></el-input>
    </el-form-item>
    <el-form-item label="版本控制" prop="appversion">
      <el-input v-model="dataForm.appversion" placeholder="版本控制"></el-input>
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
          name: '',
          gender: '',
          phone: '',
          imgsetDir: '',
          profilePhoto: '',
          roomNumber: '',
          description: '',
          isRemove: '',
          guanliyuanUsername: '',
          guanliyuanPassword: '',
          guanliyuanEmail: '',
          guanliyuanMobile: '',
          theme: '',
          defaultpage: '',
          logoimage: '',
          appversion: ''
        },
        dataRule: {
          name: [
            { required: true, message: '姓名不能为空', trigger: 'blur' }
          ],
          gender: [
            { required: true, message: '性别不能为空', trigger: 'blur' }
          ],
          phone: [
            { required: true, message: '电话不能为空', trigger: 'blur' }
          ],
          imgsetDir: [
            { required: true, message: '图像路径不能为空', trigger: 'blur' }
          ],
          profilePhoto: [
            { required: true, message: '头像路径不能为空', trigger: 'blur' }
          ],
          roomNumber: [
            { required: true, message: '房间号不能为空', trigger: 'blur' }
          ],
          description: [
            { required: true, message: '描述不能为空', trigger: 'blur' }
          ],
          isRemove: [
            { required: true, message: '删除标志不能为空', trigger: 'blur' }
          ],
          guanliyuanUsername: [
            { required: true, message: '用户名不能为空', trigger: 'blur' }
          ],
          guanliyuanPassword: [
            { required: true, message: '密码不能为空', trigger: 'blur' }
          ],
          guanliyuanEmail: [
            { required: true, message: '邮箱不能为空', trigger: 'blur' }
          ],
          guanliyuanMobile: [
            { required: true, message: '移动电话不能为空', trigger: 'blur' }
          ],
          theme: [
            { required: true, message: '主题不能为空', trigger: 'blur' }
          ],
          defaultpage: [
            { required: true, message: '缺省页面不能为空', trigger: 'blur' }
          ],
          logoimage: [
            { required: true, message: 'Logo不能为空', trigger: 'blur' }
          ],
          appversion: [
            { required: true, message: '版本控制不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/guanliyuan/guanliyuaninfo/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.guanliyuanInfo.name
                this.dataForm.gender = data.guanliyuanInfo.gender
                this.dataForm.phone = data.guanliyuanInfo.phone
                this.dataForm.imgsetDir = data.guanliyuanInfo.imgsetDir
                this.dataForm.profilePhoto = data.guanliyuanInfo.profilePhoto
                this.dataForm.roomNumber = data.guanliyuanInfo.roomNumber
                this.dataForm.description = data.guanliyuanInfo.description
                this.dataForm.isRemove = data.guanliyuanInfo.isRemove
                this.dataForm.guanliyuanUsername = data.guanliyuanInfo.guanliyuanUsername
                this.dataForm.guanliyuanPassword = data.guanliyuanInfo.guanliyuanPassword
                this.dataForm.guanliyuanEmail = data.guanliyuanInfo.guanliyuanEmail
                this.dataForm.guanliyuanMobile = data.guanliyuanInfo.guanliyuanMobile
                this.dataForm.theme = data.guanliyuanInfo.theme
                this.dataForm.defaultpage = data.guanliyuanInfo.defaultpage
                this.dataForm.logoimage = data.guanliyuanInfo.logoimage
                this.dataForm.appversion = data.guanliyuanInfo.appversion
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
              url: this.$http.adornUrl(`/guanliyuan/guanliyuaninfo/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'gender': this.dataForm.gender,
                'phone': this.dataForm.phone,
                'imgsetDir': this.dataForm.imgsetDir,
                'profilePhoto': this.dataForm.profilePhoto,
                'roomNumber': this.dataForm.roomNumber,
                'description': this.dataForm.description,
                'isRemove': this.dataForm.isRemove,
                'guanliyuanUsername': this.dataForm.guanliyuanUsername,
                'guanliyuanPassword': this.dataForm.guanliyuanPassword,
                'guanliyuanEmail': this.dataForm.guanliyuanEmail,
                'guanliyuanMobile': this.dataForm.guanliyuanMobile,
                'theme': this.dataForm.theme,
                'defaultpage': this.dataForm.defaultpage,
                'logoimage': this.dataForm.logoimage,
                'appversion': this.dataForm.appversion
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
