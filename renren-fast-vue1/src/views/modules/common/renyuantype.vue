<template>
  <el-select
    v-model="typevalue1"
    placeholder="请选择类型"
    @change="changezujian()"
  >
    <el-option
  
      v-for="item in options"
      :key="item.value"
      :label="item.label"
      :value="item.value"
    >
    </el-option>
  </el-select>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具 js，第三方插件 js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import 引入的组件需要注入到对象中才能使用
  components: {},
  props: ["typevalue1"],
  data() {
    //这里存放数据
    return {
         options: [{
             label:"老人",
             value:0
         },{
             label:"陌生人",
             value:1
         },{
             label:"义工",
             value:2
         },{
             label:"员工",
             value:3
         },{
             label:"管理员",
             value:4
         }],
    };
  },
  //计算属性 类似于 data 概念
  computed: {},
  //监控 data 中的数据变化
  watch: {
      typevalue1: (data) => {
      console.log(data);
      
    },
  },
  //方法集合
  methods: {
    changezujian(data) {
      console.log("改变了"  +  this.typevalue1);
      this.$emit("room-select-change", this.typevalue1);
    },
  },
  //生命周期 - 创建完成（可以访问当前 this 实例）
  created() {},
  //生命周期 - 挂载完成（可以访问 DOM 元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有 keep-alive 缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>