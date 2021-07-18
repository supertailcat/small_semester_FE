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
      <el-form-item label="事件id" prop="eventId">
        <el-input v-model="dataForm.eventId" placeholder="事件id"></el-input>
      </el-form-item>
      <el-form-item label="人员类型" prop="renyuanType" >
        <!-- <el-input v-model="dataForm.renyuanType" placeholder="人员类型"></el-input> -->
        <renyuantype
          :typevalue1="dataForm.renyuanType"
          @room-select-change="typechanger"
        >
        </renyuantype>
      </el-form-item>
      <el-form-item label="参与人员" prop="renyuanId">
        <!-- <el-input v-model="dataForm.renyuanId" placeholder="人员id"></el-input> -->
        <el-select v-model="dataForm.renyuanId" placeholder="事件类型">
          <el-option
            v-for="item in renyuanoptions"
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
import renyuantype from "../common/renyuantype.vue";
export default {
  components: { renyuantype },
  data() {
    return {
      renyuanoptions: [],
      visible: false,
      dataForm: {
        id: 0,
        eventId: "",
        renyuanType: "",
        renyuanId: "",
      },
      dataRule: {
        eventId: [
          { required: true, message: "事件id不能为空", trigger: "blur" },
        ],
        renyuanType: [
          { required: true, message: "人员类型不能为空", trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    typechanger(data) {
      this.dataForm.renyuanId = ''
      this.renyuanoptions = [];
      this.dataForm.renyuanType = data;
      this.getRenyuanList();
    },
    getRenyuanList(mydata) {
      var type = mydata == null ? this.dataForm.renyuanType : mydata;
      var url = "";
      if (type == 0) {
        url = "/laoren/laoreninfo";
      } else if (type == 2) {
        url = "/yigong/yigonginfo";
      } else if (type == 3) {
        url = "/yuangong/yuangonginfo";
      } else {
        return;
      }
      this.$http({
        url: this.$http.adornUrl(url + "/list"),
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
              `/event/eventpeople/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.eventId = data.eventPeople.eventId;
              this.dataForm.renyuanType = data.eventPeople.renyuanType;
              // this.dataForm.renyuanType = data.eventPeople.renyuanType;
              this.getRenyuanList(data.eventPeople.renyuanType);

              this.dataForm.renyuanId = data.eventPeople.renyuanId
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
              `/event/eventpeople/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              eventId: this.dataForm.eventId,
              renyuanType: this.dataForm.renyuanType,
              renyuanId: this.dataForm.renyuanId,
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
