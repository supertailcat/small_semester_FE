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
        prop="beginDate"
        header-align="center"
        align="center"
        label="来访时间"
      >
        <template slot-scope="scope">
          <span>{{
            scope.row.beginDate | datefmt("YYYY-MM-DD hh:mm:ss")
          }}</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="endTime"
        header-align="center"
        align="center"
        label="离开时间"
      >
        <template slot-scope="scope">
          <span>{{ scope.row.endTime | datefmt("YYYY-MM-DD hh:mm:ss") }}</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="laorenId"
        header-align="center"
        align="center"
        label="老人"
        :formatter="laorenfomt"
      >
      </el-table-column>
      <el-table-column
        prop="yigongId"
        header-align="center"
        align="center"
        label="义工"
        :formatter="yigongfomt"
      >
      </el-table-column>
      <el-table-column
        prop="laifangId"
        header-align="center"
        align="center"
        label="来访id"
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
import AddOrUpdate from "./laorenyigong-add-or-update";
export default {
  data() {
    return {
      laorenMap: new Map,
      yigongMap: new Map,
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
  created(){
    this.getRenyuan()
  },
  methods: {
    laorenfomt(row, cloum) {
      return this.laorenMap.get(row.laorenId);
    },
    yigongfomt(row, cloum) {
      return this.yigongMap.get(row.yigongId);
    },
    getRenyuan(){
       this.$http({
        url: this.$http.adornUrl("/laoren/laoreninfo/list"),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        for (let i = 0; i < data.page.list.length; i++) {
          const { id, name } = data.page.list[i];
          this.laorenMap.set(id, name);
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
          this.yigongMap.set(id, name);
        }
        this.getDataList();
      });
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/yigong/laorenyigong/list"),
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
          url: this.$http.adornUrl("/yigong/laorenyigong/delete"),
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
