<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="80px"
    >
      <el-form-item label="姓名" prop="name">
        <el-input v-model="dataForm.name" placeholder="姓名"></el-input>
      </el-form-item>
      <el-form-item label="性别" prop="gender">
        <!-- <el-input v-model="dataForm.gender" placeholder="性别"></el-input> -->
        <el-radio v-model="dataForm.gender" :label="1">男</el-radio>
        <el-radio v-model="dataForm.gender" :label="0">女</el-radio>
      </el-form-item>
      <el-form-item label="电话" prop="phone">
        <el-input v-model="dataForm.phone" placeholder="电话"></el-input>
      </el-form-item>
      <el-form-item label="身份证" prop="idCard">
        <el-input v-model="dataForm.idCard" placeholder="身份证"></el-input>
      </el-form-item>
      <el-form-item label="出生日期" prop="birth">
        <!-- <el-input v-model="dataForm.birth" placeholder="出生日期"></el-input> -->
        <el-date-picker
          v-model="dataForm.birth"
          type="date"
          placeholder="选择日期"
        >
        </el-date-picker>
      </el-form-item>
      <!-- <el-form-item label="图像目录" prop="imgsetDir">
        <el-input
          v-model="dataForm.imgsetDir"
          placeholder="图像目录"
        ></el-input>
      </el-form-item> -->
      <el-form-item label="头像路径" prop="profilePhoto">
        <!-- <el-input
          v-model="dataForm.profilePhoto"
          placeholder="头像路径"
        ></el-input> -->
        <single-upload v-model="dataForm.profilePhoto"></single-upload>
      </el-form-item>
      <el-form-item label="描述" prop="description">
        <el-input v-model="dataForm.description" placeholder="描述"></el-input>
      </el-form-item>
      <el-form-item label="删除标志" prop="isRemove">
        <!-- <el-input v-model="dataForm.isRemove" placeholder="删除标志"></el-input> -->
        <el-switch
          v-model="dataForm.isRemove"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
          active-text="未删除"
          inactive-text="删除"
        >
        </el-switch>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import { isPhone, isShenFenZheng } from "../../../utils/validate.js";
import singleUpload from "../../../components/upload/singleUpload.vue";
export default {
  components: { singleUpload },
  data() {
    return {
      visible: false,
      dataForm: {
        id: 0,
        name: "",
        gender: 0,
        phone: "",
        idCard: "",
        birth: "",
        imgsetDir: "",
        profilePhoto: "",
        description: "",
        isRemove: 1,
      },
      dataRule: {
        name: [{ required: true, message: "姓名不能为空", trigger: "blur" }],
        gender: [{ required: true, message: "性别不能为空", trigger: "blur" }],
        phone: [
          {
            required: true,
            validator: (rule, value, callback) => {
              if (value == "") {
                callback(new Error("请输入电话号码"));
              } else if (!isPhone(value)) {
                callback(
                  new Error("请输入正确的电话号码{xxxx(可省略)-xxxxxxxx}")
                );
              } else {
                callback();
              }
            },
            trigger: "blur",
          },
        ],
        idCard: [
          {
            required: true,
            validator: (rule, value, callback) => {
              if (value == "") {
                callback(new Error("请输入身份证号"));
              } else if (!isShenFenZheng(value)) {
                callback(new Error("请输入正确的身份证号"));
              } else {
                callback();
              }
            },
            trigger: "blur",
          },
        ],
        birth: [
          { required: true, message: "出生日期不能为空", trigger: "blur" },
        ],
        profilePhoto: [
          { required: true, message: "头像路径不能为空", trigger: "blur" },
        ],
      },
    };
  },

  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/yigong/yigonginfo/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.yigongInfo.name;
              this.dataForm.gender = data.yigongInfo.gender;
              this.dataForm.phone = data.yigongInfo.phone;
              this.dataForm.idCard = data.yigongInfo.idCard;
              this.dataForm.birth = data.yigongInfo.birth;
              this.dataForm.imgsetDir = data.yigongInfo.imgsetDir;
              this.dataForm.profilePhoto = data.yigongInfo.profilePhoto;
              this.dataForm.description = data.yigongInfo.description;
              this.dataForm.isRemove = data.yigongInfo.isRemove;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/yigong/yigonginfo/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              name: this.dataForm.name,
              gender: this.dataForm.gender,
              phone: this.dataForm.phone,
              idCard: this.dataForm.idCard,
              birth: this.dataForm.birth,
              imgsetDir: this.dataForm.imgsetDir,
              profilePhoto: this.dataForm.profilePhoto,
              description: this.dataForm.description,
              isRemove: this.dataForm.isRemove,
            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
  },
};
</script>
