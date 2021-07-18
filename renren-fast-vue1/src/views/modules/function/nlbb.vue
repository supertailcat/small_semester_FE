<template>
  <div class="mod-demo-echarts">
    <el-row :gutter="20">
      <el-col :span="24">
        <el-card>
          <div id="J_chartBarBox" class="chart-box"></div>
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
      yuangongdata: [],
      yigongdata: [],
      laorendata: [],
      chartBar: null,
    };
  },
  mounted() {
    this.initChartBar();
  },
  activated() {
    // 由于给echart添加了resize事件, 在组件激活时需要重新resize绘画一次, 否则出现空白bug
    if (this.chartBar) {
      this.chartBar.resize();
    }
  },
  created() {
    this.getRenyuan();
  },
  methods: {
    getRenyuan() {
      this.getRenyuanNl("/laoren/laoreninfo/list", 1);
      this.getRenyuanNl("/yigong/yigonginfo/list", 2);
      this.getRenyuanNl("/yuangong/yuangonginfo/list", 3);
    },
    getRenyuanNl(url, val) {
      this.$http({
        url: this.$http.adornUrl(url),
        method: "get",
        params: this.$http.adornParams({}),
      }).then(({ data }) => {
        var age1 = 0;
        var age2 = 0;
        var age3 = 0;
        var age4 = 0;
        var age5 = 0;
        var age6 = 0;
        for (let i = 0; i < data.page.list.length; i++) {
          var age = moment(data.page.list[i].birth).format("yyyy");
          var now = moment(new Date()).format("yyyy");

          console.log(now - age);

          if (now - age >= 20 && now - age < 40) {
            age1++;
          } else if (now - age >= 40 && now - age < 50) {
            age2++;
          } else if (now - age >= 50 && now - age < 60) {
            age3++;
          } else if (now - age >= 60 && now - age < 70) {
            age4++;
          } else if (now - age >= 70 && now - age < 80) {
            age5++;
          } else if (now - age >= 80) {
            age6++;
          }
        }
        switch (val) {
          case 1:
            this.laorendata.push(age1);
            this.laorendata.push(age2);
            this.laorendata.push(age3);
            this.laorendata.push(age4);
            this.laorendata.push(age5);
            this.laorendata.push(age6);
            break;
          case 2:
            this.yigongdata.push(age1);
            this.yigongdata.push(age2);
            this.yigongdata.push(age3);
            this.yigongdata.push(age4);
            this.yigongdata.push(age5);
            this.yigongdata.push(age6);
            break;
          case 3:
            this.yuangongdata.push(age1);
            this.yuangongdata.push(age2);
            this.yuangongdata.push(age3);
            this.yuangongdata.push(age4);
            this.yuangongdata.push(age5);
            this.yuangongdata.push(age6);
            break;
        }

        this.initChartBar();
      });
    },
    // 柱状图
    initChartBar() {
      var option = {
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "shadow",
          },
        },
        legend: {
          data: ["老人", "义工", "员工"],
        },
        grid: {
          left: "3%",
          right: "4%",
          bottom: "3%",
          containLabel: true,
        },
        xAxis: [
          {
            type: "category",
            data: ["20-40", "40-50", "50-60", "60-70", "70-80", "80以上"],
          },
        ],
        yAxis: [
          {
            type: "value",
          },
        ],
        series: [
          {
            name: "老人",
            type: "bar",
            data: this.laorendata,
          },
          {
            name: "义工",
            type: "bar",
            stack: "广告",
            data: this.yigongdata,
          },
          {
            name: "员工",
            type: "bar",
            stack: "广告",
            data: this.yuangongdata,
          },
        ],
      };
      this.chartBar = echarts.init(document.getElementById("J_chartBarBox"));
      this.chartBar.setOption(option);
      window.addEventListener("resize", () => {
        this.chartBar.resize();
      });
    },
  },
};
</script>

<style lang="scss">
.mod-demo-echarts {
  > .el-alert {
    margin-bottom: 10px;
  }
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
