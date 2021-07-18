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
      label-width="100px"
    >
      <el-form-item label="监护人类型" prop="jianhurenType">
        <!-- <el-input v-model="dataForm.jianhurenType" placeholder="监护人类型---0子女，1伴偶，2其他"></el-input> -->
        <el-radio v-model="dataForm.jianhurenType" :label="0">子女</el-radio>
        <el-radio v-model="dataForm.jianhurenType" :label="1">伴偶</el-radio>
        <el-radio v-model="dataForm.jianhurenType" :label="2">其他</el-radio>
      </el-form-item>
      <el-form-item label="监护人" prop="jianhurenId">
        <!-- <el-input
          v-model="dataForm.jianhurenId"
          placeholder="监护人id"
        ></el-input> -->
        <el-select v-model="dataForm.jianhurenId" placeholder="请选择监护人">
          <el-option
            v-for="item in jianhurenoptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
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
      <el-form-item
        label="监护人说明"
        prop="description"
        v-if="dataForm.jianhurenType == 2"
      >
        <el-input
          v-model="dataForm.description"
          placeholder="监护人说明（当选定其他监护人的时候才可以填写）"
        ></el-input>
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
      jianhurenoptions: [],
      laorenoptions: [],
      visible: false,
      dataForm: {
        id: 0,
        jianhurenType: 0,
        jianhurenId: "",
        laorenId: "",
        description: "",
      },
      dataRule: {
        jianhurenType: [
          {
            required: true,
            message: "监护人类型---0子女，1伴偶，2其他不能为空",
            trigger: "blur",
          },
        ],
        jianhurenId: [
          { required: true, message: "监护人id不能为空", trigger: "blur" },
        ],
        laorenId: [
          { required: true, message: "老人id不能为空", trigger: "blur" },
        ],
        description: [
          {
            required: true,
            message: "监护人说明（当选定其他监护人的时候才可以填写）不能为空",
            trigger: "blur",
          },
        ],
      },
    };
  },
  created() {
    this.getjianhuren();
  },
  methods: {
    getjianhuren() {
      this.$http({
        url: this.$http.adornUrl("/laoren/jianhuren/list"),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        for (let i = 0; i < data.page.list.length; i++) {
          const { id, name } = data.page.list[i];
          console.log(id, name);
          this.jianhurenoptions.push({ label: name, value: id });
        }
      });
      this.$http({
        url: this.$http.adornUrl("/laoren/laoreninfo/list"),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        for (let i = 0; i < data.page.list.length; i++) {
          const { id, name } = data.page.list[i];
          console.log(id, name);
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
              `/laoren/laorenjianhuren/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.jianhurenType = data.laorenJianhuren.jianhurenType;
              this.dataForm.jianhurenId = data.laorenJianhuren.jianhurenId;
              this.dataForm.laorenId = data.laorenJianhuren.laorenId;
              this.dataForm.description = data.laorenJianhuren.description;
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
              `/laoren/laorenjianhuren/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              jianhurenType: this.dataForm.jianhurenType,
              jianhurenId: this.dataForm.jianhurenId,
              laorenId: this.dataForm.laorenId,
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
