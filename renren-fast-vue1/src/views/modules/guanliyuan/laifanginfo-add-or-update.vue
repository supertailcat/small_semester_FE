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
      label-width="280px"
    >
      <el-form-item
        label="来访人员类型(只有义工才有数据显示)"
        prop="laifangType"
      >
        <!-- <el-input v-model="dataForm.laifangType" placeholder="来访人员类型"></el-input> -->
        <renyuantype
          :typevalue1="dataForm.laifangType"
          @room-select-change="typechanger"
        >
        </renyuantype>
      </el-form-item>
      <el-form-item label="来访人员" prop="peopleId">
        <!-- <el-input
          v-model="dataForm.peopleId"
          placeholder="来访人员id"
        ></el-input> -->
        <el-select v-model="dataForm.peopleId" placeholder="来访人员">
          <el-option
            v-for="item in renyuanoptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="来访时间" prop="beginTime">
        <!-- <el-input
          v-model="dataForm.beginTime"
          placeholder="来访时间"
        ></el-input> -->
        <el-date-picker
          v-model="dataForm.beginTime"
          type="datetime"
          placeholder="来访时间"
          default-time="12:00:00"
        >
        </el-date-picker>
      </el-form-item>
      <el-form-item label="离开时间" prop="endTime">
        <!-- <el-input v-model="dataForm.endTime" placeholder="离开时间"></el-input> -->
        <el-date-picker
          v-model="dataForm.endTime"
          type="datetime"
          placeholder="离开时间"
          default-time="12:00:00"
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
import renyuantype from "../common/renyuantype.vue";
export default {
  components: { renyuantype },
  data() {
    return {
      renyuanoptions: [],
      visible: false,
      dataForm: {
        id: 0,
        laifangType: "",
        peopleId: "",
        beginTime: "",
        endTime: "",
      },
      dataRule: {
        laifangType: [
          { required: true, message: "来访人员类型不能为空", trigger: "blur" },
        ],
        beginTime: [
          { required: true, message: "来访时间不能为空", trigger: "blur" },
        ],
        endTime: [
          { required: true, message: "离开时间不能为空", trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    typechanger(data) {
      this.dataForm.peopleId = "";
      this.renyuanoptions = [];
      this.dataForm.laifangType = data;
      this.getRenyuanList();
    },
    getRenyuanList(mydata) {
      var type = mydata == null ? this.dataForm.laifangType : mydata;
      var myurl = "";
      console.log(type);
      if (type == 2) {
        myurl = "/yigong/yigonginfo";
      } else {
        return "";
      }
      console.log(myurl);
      this.$http({
        url: this.$http.adornUrl(myurl + "/list"),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        for (let i = 0; i < data.page.list.length; i++) {
          const { id, name } = data.page.list[i];
          this.renyuanoptions.push({ label: name, value: id });
        }
      });
    },
    init(id) {
      this.renyuanoptions = [];
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/guanliyuan/laifanginfo/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.laifangType = data.laifangInfo.laifangType;
              this.dataForm.peopleId = data.laifangInfo.peopleId;
              this.dataForm.beginTime = data.laifangInfo.beginTime;
              this.getRenyuanList(data.laifangInfo.laifangType);
              this.dataForm.endTime = data.laifangInfo.endTime;
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
              `/guanliyuan/laifanginfo/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              laifangType: this.dataForm.laifangType,
              peopleId: this.dataForm.peopleId,
              beginTime: this.dataForm.beginTime,
              endTime: this.dataForm.endTime,
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
