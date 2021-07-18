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
      <el-form-item label="老人" prop="laorenId">
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
      <el-form-item label="员工" prop="yuangongId">
        <!-- <el-input v-model="dataForm.yuangongId" placeholder="员工id"></el-input> -->
        <el-select v-model="dataForm.yuangongId" placeholder="请选择员工">
          <el-option
            v-for="item in yuangongoptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="开始时间" prop="beginDate">
        <!-- <el-input
          v-model="dataForm.beginDate"
          placeholder="开始时间"
        ></el-input> -->
        <el-date-picker
          v-model="dataForm.beginDate"
          type="date"
          placeholder="选择日期"
        >
        </el-date-picker>
      </el-form-item>
      <el-form-item label="结束时间" prop="endTime">
        <!-- <el-input v-model="dataForm.endTime" placeholder="结束时间"></el-input> -->
        <el-date-picker
          v-model="dataForm.endTime"
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
export default {
  data() {
    return {
      yuangongoptions: [],
      laorenoptions: [],
      visible: false,
      dataForm: {
        id: 0,
        laorenId: "",
        yuangongId: "",
        beginDate: "",
        endTime: "",
      },
      dataRule: {
        laorenId: [
          { required: true, message: "老人id不能为空", trigger: "blur" },
        ],
        yuangongId: [
          { required: true, message: "员工id不能为空", trigger: "blur" },
        ],
        beginDate: [
          { required: true, message: "开始时间不能为空", trigger: "blur" },
        ],
        endTime: [
          { required: true, message: "结束时间不能为空", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getRenyuan();
  },
  methods: {
    getRenyuan() {
      this.$http({
        url: this.$http.adornUrl("/yuangong/yuangonginfo/list"),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        for (let i = 0; i < data.page.list.length; i++) {
          const { id, name } = data.page.list[i];
          this.yuangongoptions.push({ label: name, value: id });
        }
      });
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
    },
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/yuangong/laorenyuangong/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.laorenId = data.laorenYuangong.laorenId;
              this.dataForm.yuangongId = data.laorenYuangong.yuangongId;
              this.dataForm.beginDate = data.laorenYuangong.beginDate;
              this.dataForm.endTime = data.laorenYuangong.endTime;
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
              `/yuangong/laorenyuangong/${
                !this.dataForm.id ? "save" : "update"
              }`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              laorenId: this.dataForm.laorenId,
              yuangongId: this.dataForm.yuangongId,
              beginDate: this.dataForm.beginDate,
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
