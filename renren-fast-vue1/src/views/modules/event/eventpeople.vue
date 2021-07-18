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
        prop="eventId"
        header-align="center"
        align="center"
        label="事件id"
      >
      </el-table-column>
      <el-table-column
        prop="renyuanType"
        header-align="center"
        align="center"
        label="人员类型"
        :formatter="renyuanformatter"
      >
      </el-table-column>
      <el-table-column
        prop="renyuanId"
        header-align="center"
        align="center"
        label="参与人员"
        :formatter="renyuanfomt"
      >
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
  </div>
</template>

<script>
import AddOrUpdate from "./eventpeople-add-or-update";
export default {
  data() {
    return {
      map1: new Map(),
      map2: new Map(),
      map3: new Map(),
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
    this.getallRenyuan();
  },
  methods: {
    getallRenyuan() {
      this.$http({
        url: this.$http.adornUrl("/laoren/laoreninfo/list"),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        for (let i = 0; i < data.page.list.length; i++) {
          const { id, name } = data.page.list[i];
          console.log(id, name);
          this.map1.set(id, name);
        }
        this.getDataList();
      });
      this.$http({
        url: this.$http.adornUrl("/yigong/yigonginfo/list"),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        for (let i = 0; i < data.page.list.length; i++) {
          const { id, name } = data.page.list[i];
          console.log(id, name);
          this.map2.set(id, name);
        }
        this.getDataList();
      });
      this.$http({
        url: this.$http.adornUrl("/yuangong/yuangonginfo/list"),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        for (let i = 0; i < data.page.list.length; i++) {
          const { id, name } = data.page.list[i];
          console.log(id, name);
          this.map3.set(id, name);
        }
        this.getDataList();
      });
    },
    renyuanfomt(row, cloum) {
      if (row.renyuanType == 0) {
        return this.map1.get(row.renyuanId);
      } else if (row.renyuanType == 2) {
        return this.map2.get(row.renyuanId);
      } else if (row.renyuanType == 3) {
        return this.map3.get(row.renyuanId);
      }
      return "";
    },
    renyuanformatter(row, cloum) {
      if (row.renyuanType == 0) {
        return "老人";
      } else if (row.renyuanType == 1) {
        return "陌生人";
      } else if (row.renyuanType == 2) {
        return "义工";
      } else if (row.renyuanType == 3) {
        return "员工";
      } else if (row.renyuanType == 4) {
        return "管理员";
      }
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/event/eventpeople/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
        }),
      }).then(({ data }) => {
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
          url: this.$http.adornUrl("/event/eventpeople/delete"),
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
