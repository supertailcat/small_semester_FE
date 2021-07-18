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
      <el-form-item label="事件类型" prop="eventType">
        <!-- <el-input
          v-model="dataForm.eventType"
          placeholder="事件类型"
        ></el-input> -->
        <el-select v-model="dataForm.eventType" placeholder="事件类型">
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="事件发生的时间" prop="eventDate">
        <!-- <el-input v-model="dataForm.eventDate" placeholder="事件发生的时间"></el-input> -->
        <el-date-picker
          v-model="dataForm.eventDate"
          type="datetime"
          placeholder="事件发生的时间"
          default-time="24:00:00"
        >
        </el-date-picker> 
      </el-form-item>
      <el-form-item label="事件发生地点" prop="roomNumber">
        <!-- <el-input
          v-model="dataForm.roomNumber"
          placeholder="事件发生地点"
        ></el-input> -->
        <roomselect
          :roomvalue1="dataForm.roomNumber"
          @room-select-change="roomselectchange"
        ></roomselect>
      </el-form-item>
      <el-form-item label="事件描述" prop="description">
        <!-- <el-input
          v-model="dataForm.description"
          placeholder="事件描述"
        ></el-input> -->
        <single-upload v-model="dataForm.description"></single-upload>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import roomselect from "../common/roomselect.vue";
import singleUpload from "../../../components/upload/singleUpload.vue";
export default {
  components: { roomselect,singleUpload },
  data() {
    return {
      options: [
        {
          value: 1,
          label: "陌生人出现",
        },
        {
          value: 2,
          label: "摔倒",
        },
        {
          value: 3,
          label: "义工距离过近",
        },
        {
          value: 4,
          label: "进入禁止区域",
        },
        {
          value: 0,
          label: "抓拍高兴",
        },
      ],
      value: "",
      visible: false,
      dataForm: {
        id: 0,
        eventType: "",
        eventDate: "",
        roomNumber: "",
        description: "",
      },
      dataRule: {
        eventType: [
          { required: true, message: "事件类型不能为空", trigger: "blur" },
        ],
        eventDate: [
          {
            required: true,
            message: "事件发生的时间不能为空",
            trigger: "blur",
          },
        ],
        roomNumber: [
          { required: true, message: "事件发生地点不能为空", trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    roomselectchange(data) {
      console.log("被改变了" + this.dataForm.roomNumber);
      this.dataForm.roomNumber = data
    },
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/event/eventinfo/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.eventType = data.eventInfo.eventType;
              this.dataForm.eventDate = data.eventInfo.eventDate;
              this.dataForm.roomNumber = data.eventInfo.roomNumber;
              this.dataForm.description = data.eventInfo.description;
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
              `/event/eventinfo/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              eventType: this.dataForm.eventType,
              eventDate: this.dataForm.eventDate,
              roomNumber: this.dataForm.roomNumber,
              description: this.dataForm.description,
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
