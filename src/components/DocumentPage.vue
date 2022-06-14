<template>
  <div class="common-layout MinePage">
    <el-container>
      <el-header class="header">
        <div id="title">文件资源管理器</div></el-header
      >
      <el-main class="main">
        <div class="PageField">
          <el-table :data="cur_dir" style="width: 100%">
            <el-table-column  label="名称" width="300">
              <template #default="scope">
                <div style="display: flex; align-items: center">
                  <img :src="this.icon_urls[scope.row.type]" class="MineIcon">
                  <span style="margin-left: 10px">{{ scope.row.name }}</span>
      
                </div>
              </template>
            </el-table-column>

            <el-table-column
              prop="last_edit_time"
              label="修改日期"
              width="300"
            />
            <el-table-column prop="type" label="类型" width="300" />
            <el-table-column prop="size" label="大小" width="300" />
          </el-table>
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      icon_urls:[require("../assets/folder.png"),require("../assets/document.png")],
      icon:"../assets/folder.png",
      cur_dir: [
        {
          name: "AppStore",
          last_edit_time: "2022/3/5",
          type: 0,
          size: "",
        },
        {
          name: "App",
          last_edit_time: "2022/3/5",
          type: 1,
          size: "",
        },
      ],
    };
  },
};
</script>

<style scoped>
.MineIcon{
  width: 30px;
  height: 30px;
}
.MinePage {
  width: 90%;
  margin-left: 5%;
  box-shadow: rgb(172, 166, 166) 2px 2px 10px;
}
.header {
  background-color: #66b1ff;
  color: white;
}
.main {
  background-color: #e5eaf3;
  height: 600px;
}
.PageField {
  height: 100%;
  background-color: white;
  border-radius: 10px;
}
#title {
  padding-top: 10px;
}
.PageField >>> .el-table__row > td {
  border: none;
  height: 0;
}
</style>
