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
      label-width="120px"
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
      <el-form-item label="身份证号" prop="idCard">
        <el-input v-model="dataForm.idCard" placeholder="身份证号"></el-input>
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
      <!-- <el-form-item label="入院日期" prop="checkinDate">
        <el-input
          v-model="dataForm.checkinDate"
          placeholder="入院日期"
        ></el-input>
      </el-form-item> -->
      <!-- <el-form-item label="出院日期" prop="checkoutDate">
        <el-input
          v-model="dataForm.checkoutDate"
          placeholder="出院日期"
        ></el-input>
      </el-form-item> -->
      <!-- <el-form-item label="图像路径" prop="imgsetDir">
        <el-input
          v-model="dataForm.imgsetDir"
          placeholder="图像路径"
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
        <!-- <el-input v-model="dataForm.roomNumber" placeholder="房间号"></el-input> -->
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
      <el-form-item label="是否需要义工" prop="isVolunteer">
        <!-- <el-input
          v-model="dataForm.isVolunteer"
          placeholder="是否需要义工"
        ></el-input> -->
        <el-switch
          v-model="dataForm.isVolunteer"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
          active-text="需要"
          inactive-text="不需要"
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
        checkinDate: "",
        checkoutDate: "",
        imgsetDir: "",
        profilePhoto: "",
        roomNumber: "",
        description: "",
        isRemove: 1,
        isVolunteer: 1,
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
        roomNumber: [
          { required: true, message: "房间号不能为空", trigger: "blur" },
        ],
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
              `/laoren/laoreninfo/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.laorenInfo.name;
              this.dataForm.gender = data.laorenInfo.gender;
              this.dataForm.phone = data.laorenInfo.phone;
              this.dataForm.idCard = data.laorenInfo.idCard;
              this.dataForm.birth = data.laorenInfo.birth;
              this.dataForm.checkinDate = data.laorenInfo.checkinDate;
              this.dataForm.checkoutDate = data.laorenInfo.checkoutDate;
              this.dataForm.imgsetDir = data.laorenInfo.imgsetDir;
              this.dataForm.profilePhoto = data.laorenInfo.profilePhoto;
              this.dataForm.roomNumber = data.laorenInfo.roomNumber;
              this.dataForm.description = data.laorenInfo.description;
              this.dataForm.isRemove = data.laorenInfo.isRemove;
              this.dataForm.isVolunteer = data.laorenInfo.isVolunteer;
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
              `/laoren/laoreninfo/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              name: this.dataForm.name,
              gender: this.dataForm.gender,
              phone: this.dataForm.phone,
              idCard: this.dataForm.idCard,
              birth: this.dataForm.birth,
              checkinDate: this.dataForm.checkinDate,
              checkoutDate: this.dataForm.checkoutDate,
              imgsetDir: this.dataForm.imgsetDir,
              profilePhoto: this.dataForm.profilePhoto,
              roomNumber: this.dataForm.roomNumber,
              description: this.dataForm.description,
              isRemove: this.dataForm.isRemove,
              isVolunteer: this.dataForm.isVolunteer,
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
