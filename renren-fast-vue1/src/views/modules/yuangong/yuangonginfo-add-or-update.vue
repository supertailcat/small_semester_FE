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
          @change="changezujian"
        >
        </el-date-picker>
      </el-form-item>
      <!-- <el-form-item label="入职日期" prop="hireDate">
        <el-input v-model="dataForm.hireDate" placeholder="入职日期"></el-input>
      </el-form-item>
      <el-form-item label="离职日期" prop="resignDate">
        <el-input
          v-model="dataForm.resignDate"
          placeholder="离职日期"
        ></el-input>
      </el-form-item> -->
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
      <el-form-item label="房间号" prop="roomNumber">
        <!-- <el-input v-model="dataForm.roomNumber" placeholder=""></el-input> -->
        <roomselect
          :roomvalue1="dataForm.roomNumber"
          @room-select-change="roomselectchange"
        ></roomselect>
      </el-form-item>
      <el-form-item label="描述" prop="description">
        <!-- <el-input v-model="dataForm.description" placeholder="描述"></el-input> -->
        <el-input
          type="textarea"
          :rows="3"
          placeholder="描述"
          v-model="dataForm.description"
        >
        </el-input>
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
          @change="changezujian"
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
import roomselect from "../common/roomselect.vue";
export default {
  components: { singleUpload ,roomselect },
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
        hireDate: "",
        resignDate: "",
        imgsetDir: "",
        profilePhoto: "",
        roomNumber: "",
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
        roomNumber: [{ required: true, message: "不能为空", trigger: "blur" }],
      },
    };
  },
  methods: {
    roomselectchange(data) {
      console.log("被改变了" + this.dataForm.roomNumber);
      this.dataForm.roomNumber = data
    },
    //测试组件能否正常使用
    changezujian(data) {
      console.log(data);
    },
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/yuangong/yuangonginfo/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.yuangongInfo.name;
              this.dataForm.gender = data.yuangongInfo.gender;
              this.dataForm.phone = data.yuangongInfo.phone;
              this.dataForm.idCard = data.yuangongInfo.idCard;
              this.dataForm.birth = data.yuangongInfo.birth;
              this.dataForm.hireDate = data.yuangongInfo.hireDate;
              this.dataForm.resignDate = data.yuangongInfo.resignDate;
              this.dataForm.imgsetDir = data.yuangongInfo.imgsetDir;
              this.dataForm.profilePhoto = data.yuangongInfo.profilePhoto;
              this.dataForm.roomNumber = data.yuangongInfo.roomNumber;
              this.dataForm.description = data.yuangongInfo.description;
              this.dataForm.isRemove = data.yuangongInfo.isRemove;
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
              `/yuangong/yuangonginfo/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              name: this.dataForm.name,
              gender: this.dataForm.gender,
              phone: this.dataForm.phone,
              idCard: this.dataForm.idCard,
              birth: this.dataForm.birth,
              hireDate: this.dataForm.hireDate,
              resignDate: this.dataForm.resignDate,
              imgsetDir: this.dataForm.imgsetDir,
              profilePhoto: this.dataForm.profilePhoto,
              roomNumber: this.dataForm.roomNumber,
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
