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
      label-width="140px"
    >
      <el-form-item label="地点名字" prop="name">
        <el-input v-model="dataForm.name" placeholder="地点名字"></el-input>
      </el-form-item>
      <el-form-item label="摄像头访问路径" prop="sxtPath">
        <el-input
          v-model="dataForm.sxtPath"
          placeholder="摄像头访问路径"
        ></el-input>
      </el-form-item>
      <el-form-item label="开始使用时间" prop="beginDate">
        <!-- <el-input
          v-model="dataForm.beginDate"
          placeholder="开始使用时间"
        ></el-input> -->
        <el-date-picker
          v-model="dataForm.beginDate"
          type="date"
          placeholder="选择日期"
        >
        </el-date-picker>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import { isURL } from "../../../utils/validate";
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        id: 0,
        name: "",
        sxtPath: "",
        beginDate: "",
      },
      dataRule: {
        name: [
          { required: true, message: "地点名字不能为空", trigger: "blur" },
        ],
        sxtPath: [
          {
            required: true,
            validator: (rule, value, callback) => {
              if (value == "") {
                callback(new Error("请输入摄像头访问路径"));
              } else if (!isURL(value)) {
                callback(new Error("请输入正确的访问路径"));
              } else {
                callback();
              }
            },
            trigger: "blur",
          },
        ],
        beginDate: [
          { required: true, message: "开始使用时间不能为空", trigger: "blur" },
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
              `/guanliyuan/room/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.room.name;
              this.dataForm.sxtPath = data.room.sxtPath;
              this.dataForm.beginDate = data.room.beginDate;
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
              `/guanliyuan/room/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              name: this.dataForm.name,
              sxtPath: this.dataForm.sxtPath,
              beginDate: this.dataForm.beginDate,
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
