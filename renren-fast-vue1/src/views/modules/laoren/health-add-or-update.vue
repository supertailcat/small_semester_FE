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
      <el-form-item label="记录时间" prop="recordTime">
        <!-- <el-input v-model="dataForm.recordTime" placeholder="记录时间"></el-input> -->
        <el-date-picker
          v-model="dataForm.recordTime"
          type="datetime"
          placeholder="记录时间"
          default-time="12:00:00"
        >
        </el-date-picker>
      </el-form-item>
      <el-form-item label="心情类型" prop="emotionType">
        <!-- <el-input v-model="dataForm.emotionType" placeholder="心情类型"></el-input> -->
        <el-select v-model="dataForm.emotionType" placeholder="请选择心情类型">
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="健康评分" prop="healthPoint">
        <el-input
          v-model="dataForm.healthPoint"
          placeholder="健康评分"
        ></el-input>
      </el-form-item>
      <el-form-item label="老人" prop="laorenId">
        <!-- <el-input v-model="dataForm.laorenId" placeholder="老人"></el-input> -->
        <el-select v-model="dataForm.laorenId" placeholder="请选择心情类型">
          <el-option
            v-for="item in laorenoptions"
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
      options: [
        {
          value: 1,
          label: "愤怒",
        },
        {
          value: 2,
          label: "厌恶",
        },
        {
          value: 3,
          label: "恐惧",
        },
        {
          value: 4,
          label: "高兴",
        },
        {
          value: 5,
          label: "正常",
        },
        {
          value: 6,
          label: "悲伤",
        },
        {
          value: 7,
          label: "惊讶",
        },
      ],
      visible: false,
      dataForm: {
        id: 0,
        recordTime: "",
        emotionType: "",
        healthPoint: "",
        laorenId: "",
      },
      dataRule: {
        recordTime: [
          { required: true, message: "记录时间不能为空", trigger: "blur" },
        ],
        emotionType: [
          { required: true, message: "心情类型不能为空", trigger: "blur" },
        ],
        laorenId: [
          { required: true, message: "老人id不能为空", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getLaoren()
  },
  methods: {
    getLaoren() {
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
            url: this.$http.adornUrl(`/laoren/health/info/${this.dataForm.id}`),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.recordTime = data.health.recordTime;
              this.dataForm.emotionType = data.health.emotionType;
              this.dataForm.healthPoint = data.health.healthPoint;
              this.dataForm.laorenId = data.health.laorenId;
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
              `/laoren/health/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              recordTime: this.dataForm.recordTime,
              emotionType: this.dataForm.emotionType,
              healthPoint: this.dataForm.healthPoint,
              laorenId: this.dataForm.laorenId,
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
