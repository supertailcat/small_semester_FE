<template>
  <div class="mod-demo-echarts">
    <el-select
      class="myselect"
      v-model="laorenvalue"
      placeholder="请选择"
      @change="laorenChange"
    >
      <el-option
        v-for="item in laorenoptions"
        :key="item.value"
        :label="item.label"
        :value="item.value"
      >
      </el-option>
    </el-select>
    <el-row :gutter="20">
      <el-col :span="24">
        <el-card>
          <div id="J_chartLineBox" class="chart-box"></div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import echarts from "echarts";
import moment from "moment";
export default {
  data() {
    return {
      xdata: [],
      pointdata: [],
      laorenoptions: [],
      chartLine: null,
      laorenvalue: "",
    };
  },
  mounted() {
    this.initChartLine();
  },
  activated() {
    // 由于给echart添加了resize事件, 在组件激活时需要重新resize绘画一次, 否则出现空白bug
    if (this.chartLine) {
      this.chartLine.resize();
    }
  },
  created() {
    this.getLaoren();
  },
  methods: {
    laorenChange(val) {
      this.xdata = new Array();
      this.pointdata = new Array();
      var obj = {};
      obj = this.laorenoptions.find((item) => {
        return item.value === val;
      });
      this.$http({
        url: this.$http.adornUrl("/laoren/health/list"),
        method: "get",
        params: this.$http.adornParams({"key":obj.label}),
      }).then(({ data }) => {
        if (data.page.list.length <= 7) {
          for (let i = 0; i < data.page.list.length; i++) {
            this.xdata.push(
              moment(data.page.list[i].recordTime).format("YYYY-MM-DD hh:mm")
            );
            this.pointdata.push(data.page.list[i].healthPoint);
          }
        } else {
          for (let i = data.page.list.length - 7 ; i < data.page.list.length; i++) {
            this.xdata.push(
              moment(data.page.list[i].recordTime).format("YYYY-MM-DD hh:mm")
            );
            this.pointdata.push(data.page.list[i].healthPoint);
          }
        }

        this.initChartLine();
      });
    },
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

    // 折线图
    initChartLine() {
      var option = {
        title: {
          text: "老人心情报表",
        },
        tooltip: {
          trigger: "axis",
        },
        legend: {},
        grid: {
          left: "3%",
          right: "4%",
          bottom: "3%",
          containLabel: true,
        },
        toolbox: {
          feature: {
            saveAsImage: {},
          },
        },
        xAxis: {
          type: "category",
          boundaryGap: false,
          data: this.xdata,
        },
        yAxis: {
          type: "value",
        },
        series: [
          {
            name: "心情评分",
            type: "line",
            stack: "分数",
            data: this.pointdata,
          },
        ],
      };
      this.chartLine = echarts.init(document.getElementById("J_chartLineBox"));
      this.chartLine.setOption(option);
      window.addEventListener("resize", () => {
        this.chartLine.resize();
      });
    },
  },
};
</script>

<style lang="scss">
.myselect {
  margin-bottom: 60px;
}

.mod-demo-echarts {
  > .el-row {
    margin-top: -10px;
    margin-bottom: -10px;
    .el-col {
      padding-top: 10px;
      padding-bottom: 10px;
    }
  }
  .chart-box {
    min-height: 400px;
  }
}
</style>
