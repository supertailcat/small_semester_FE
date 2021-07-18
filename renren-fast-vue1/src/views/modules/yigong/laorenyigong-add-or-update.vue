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
      <el-form-item label="来访时间" prop="beginDate">
        <!-- <el-input v-model="dataForm.beginDate" placeholder="来访时间"></el-input> -->
        <el-date-picker
          v-model="dataForm.beginDate"
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
      <el-form-item label="老人id" prop="laorenId">
        <!-- <el-input v-model="dataForm.laorenId" placeholder="老人id"></el-input> -->
        <el-select v-model="dataForm.laorenId" placeholder="请选择老人">
          <el-option
            v-for="item in laorenoptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="义工" prop="yigongId">
        <!-- <el-input v-model="dataForm.yigongId" placeholder="义工id"></el-input> -->
        <el-select v-model="dataForm.yigongId" placeholder="请选择义工" @change="changeyigong">
          <el-option
            v-for="item in yigongoptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="来访id" prop="laifangId">
        <!-- <el-input v-model="dataForm.laifangId" placeholder="来访id"></el-input> -->
        <el-select v-model="dataForm.laifangId" placeholder="请选择来访id">
          <el-option
            v-for="item in laifangoptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
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
  data() {
    return {
      laorenoptions: [],
      yigongoptions: [],
      laifangoptions: [],
      visible: false,
      dataForm: {
        id: 0,
        beginDate: "",
        endTime: "",
        laorenId: "",
        yigongId: "",
        laifangId: "",
      },
      dataRule: {
        beginDate: [
          { required: true, message: "来访时间不能为空", trigger: "blur" },
        ],
        endTime: [
          { required: true, message: "离开时间不能为空", trigger: "blur" },
        ],
        laorenId: [
          { required: true, message: "老人id不能为空", trigger: "blur" },
        ],
        yigongId: [
          { required: true, message: "义工id不能为空", trigger: "blur" },
        ],
        laifangId: [
          { required: true, message: "来访id不能为空", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getRenyuan();
  },
  methods: {
    changeyigong() {
      this.laifangoptions = [];
      this.dataForm.laifangId = ''
      this.$http({
        url: this.$http.adornUrl(
          "/guanliyuan/laifanginfo/yigong/" + this.dataForm.yigongId
        ),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        console.log(data)
        for (let i = 0; i < data.laifang.length; i++) {
          const { id, name } = data.laifang[i];
          this.laifangoptions.push({ label: name, value: id });
        }
      });
    },
    getRenyuan() {
      this.$http({
        url: this.$http.adornUrl("/laoren/laoreninfo/list"),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        for (let i = 0; i < data.page.list.length; i++) {
          const { id, name } = data.page.list[i];
          this.laorenoptions.push({ label: name, value: id });
        }
      });
      this.$http({
        url: this.$http.adornUrl("/yigong/yigonginfo/list"),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        for (let i = 0; i < data.page.list.length; i++) {
          const { id, name } = data.page.list[i];
          this.yigongoptions.push({ label: name, value: id });
        }
      });
    },
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/yigong/laorenyigong/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.beginDate = data.laorenYigong.beginDate;
              this.dataForm.endTime = data.laorenYigong.endTime;
              this.dataForm.laorenId = data.laorenYigong.laorenId;
              this.dataForm.yigongId = data.laorenYigong.yigongId;
              this.dataForm.laifangId = data.laorenYigong.laifangId;
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
              `/yigong/laorenyigong/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              beginDate: this.dataForm.beginDate,
              endTime: this.dataForm.endTime,
              laorenId: this.dataForm.laorenId,
              yigongId: this.dataForm.yigongId,
              laifangId: this.dataForm.laifangId,
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
