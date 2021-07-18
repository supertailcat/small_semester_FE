<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
    >
      <el-form-item>
        <el-input
          v-model="dataForm.key"
          placeholder="参数名"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
          >批量删除</el-button
        >
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%"
    >
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50"
      >
      </el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="id"
      >
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="姓名"
      >
      </el-table-column>
      <el-table-column
        prop="gender"
        header-align="center"
        align="center"
        label="性别"
        :formatter="genderFormatter"
      >
      </el-table-column>
      <el-table-column
        prop="phone"
        header-align="center"
        align="center"
        label="电话"
      >
      </el-table-column>
      <el-table-column
        prop="birth"
        header-align="center"
        align="center"
        label="出生日期"
      >
        <template slot-scope="scope">
          <span>{{ scope.row.birth | datefmt("YYYY-MM-DD") }}</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="checkinDate"
        header-align="center"
        align="center"
        label="入院日期"
      >
        <template slot-scope="scope">
          <span>{{ scope.row.checkinDate | datefmt("YYYY-MM-DD") }}</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="checkoutDate"
        header-align="center"
        align="center"
        label="出院日期"
      >
        <template slot-scope="scope">
          <span>{{ scope.row.checkoutDate | datefmt("YYYY-MM-DD") }}</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="profilePhoto"
        header-align="center"
        align="center"
        label="头像"
      >
        <template slot-scope="scope">
          <!-- <el-image
              style="width: 100px; height: 80px"
              :src="scope.row.logo"
              fit="contain"
            ></el-image> -->
          <img
            :src="scope.row.profilePhoto"
            style="width: 100px; height: 80px"
          />
        </template>
      </el-table-column>
      <el-table-column
        prop="roomNumber"
        header-align="center"
        align="center"
        label="房间号"
        :formatter="eventroomfomt"
      >
      </el-table-column>
      <el-table-column
        prop="isVolunteer"
        header-align="center"
        align="center"
        label="是否需要义工"
      >
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.isVolunteer"
            active-color="#13ce66"
            inactive-color="#ff4949"
            :active-value="1"
            :inactive-value="0"
            @change="updateVolunteer(scope.row)"
          >
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作"
      >
        <template slot-scope="scope">
          <el-button
            type="text"
            size="small"
            @click="addOrUpdateHandle(scope.row.id)"
            >修改</el-button
          >
          <el-button type="text" size="small" @click="tianjia(scope.row.id)"
            >添加监护人</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="deleteHandle(scope.row.id)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    >
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @refreshDataList="getDataList"
    ></add-or-update>
    <el-dialog title="添加监护人" :visible.sync="dialogFormVisible">
      <el-form :model="form" :rules="rule" ref="form">
        <el-form-item
          label="监护人类别"
          :label-width="formLabelWidth"
          prop="jhtype"
        >
          <!-- <el-input v-model="dataForm.gender" placeholder="性别"></el-input> -->
          <el-radio v-model="form.jhtype" :label="0">子女</el-radio>
          <el-radio v-model="form.jhtype" :label="1">伴偶</el-radio>
          <el-radio v-model="form.jhtype" :label="2">其他</el-radio>
        </el-form-item>
        <el-form-item
          label="说明"
          v-if="form.jhtype == 2"
          :label-width="formLabelWidth"
          prop="shuoming"
        >
          <el-input v-model="form.shuoming" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="姓名" :label-width="formLabelWidth" prop="name">
          <el-input v-model="form.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="电话" :label-width="formLabelWidth" prop="phone">
          <el-input v-model="form.phone" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="微信" :label-width="formLabelWidth" prop="weixin">
          <el-input v-model="form.weixin" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitForm('form')">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import AddOrUpdate from "./laoreninfo-add-or-update";
import { isPhone } from "../../../utils/validate.js";
export default {
  data() {
    return {
      form: {
        laorenId: 0,
        jhtype: 0,
        shuoming: "",
        name: "",
        phone: "",
        weixin: "",
      },
      rule: {
        jhtype: [
          { required: true, message: "监护人类型不能为空", trigger: "blur" },
        ],
        shuoming: [
          { required: true, message: "说明不能为空", trigger: "blur" },
        ],
        name: [
          { required: true, message: "监护人姓名不能为空", trigger: "blur" },
        ],
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
        weixin: [
          { required: true, message: "监护人微信不能为空", trigger: "blur" },
        ],
      },
      formLabelWidth: "120px",
      dialogFormVisible: false,
      eventroom: new Map(),
      dataForm: {
        key: "",
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
    };
  },
  components: {
    AddOrUpdate,
  },
  activated() {
    this.getDataList();
  },
  created() {
    this.getEventRoom();
  },
  methods: {
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl("/laoren/laoreninfo/addJianhuren"),
            method: "post",
            data: this.$http.adornData(
              {
                laorenId: this.form.laorenId,
                jhtype: this.form.jhtype,
                shuoming: this.form.shuoming,
                name: this.form.name,
                phone: this.form.phone,
                weixin: this.form.weixin,
              },
              false
            ),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.dialogFormVisible = false;
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
    tianjia(id) {
      this.dialogFormVisible = true;
      this.form.laorenId = id;
    },
    getEventRoom() {
      this.$http({
        url: this.$http.adornUrl("/guanliyuan/room/list"),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        for (let i = 0; i < data.page.list.length; i++) {
          const { id, name } = data.page.list[i];
          console.log(id, name);
          this.eventroom.set(id, name);
          this.getDataList();
        }
      });
    },
    eventroomfomt(row, cloum) {
      return this.eventroom.get(parseInt(row.roomNumber));
    },
    updateVolunteer(data) {
      var { id, isVolunteer } = data;
      this.$http({
        url: this.$http.adornUrl("/laoren/laoreninfo/update/volubteer"),
        method: "post",
        data: this.$http.adornData({ id, isVolunteer }, false),
      }).then(({ data }) => {});
    },

    genderFormatter(row, cloum) {
      if (row.gender == 0) {
        return "女";
      } else if (row.gender == 1) {
        return "男";
      } else {
        return "未知";
      }
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/laoren/laoreninfo/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
        }),
      }).then(({ data }) => {
        console.log(data);
        if (data && data.code === 0) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map((item) => {
            return item.id;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/laoren/laoreninfo/delete"),
          method: "post",
          data: this.$http.adornData(ids, false),
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList();
              },
            });
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    },
  },
};
</script>
