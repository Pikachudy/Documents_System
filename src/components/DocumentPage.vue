<template>
  <div class="common-layout MinePage">
    <!-- 磁盘侧栏 -->
    <el-drawer
      v-model="this.show_drawer"
      title="磁盘块使用情况"
      :with-header="true"
    >
      <el-table :data="this.physical_disk" style="width: 100%">
        <el-table-column prop="block_num" label="块号" width="130" />
        <el-table-column label="存放内容" width="180">
          <template #default="scope">
            <span style="margin-left: 10px">{{ scope.row.des_content }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="disk_next" label="下一块号" width="130" />
      </el-table>
    </el-drawer>
    <!-- 属性对话框 -->
    <el-dialog v-model="show_attr" title="属性">
      <table id="MineTable">
        <tr>
          <td>名称</td>

          <td>{{ this.cur_dir[this.selected_index].name }}</td>
        </tr>
        <tr>
          <td>最后修改时间</td>

          <td>{{ this.cur_dir[this.selected_index].last_edit_timestr }}</td>
        </tr>
        <tr>
          <td>类型</td>

          <td>
            {{
              this.cur_dir[this.selected_index].type == 0
                ? "文件夹"
                : this.cur_dir[this.selected_index].type == 1
                ? "文件"
                : "磁盘"
            }}
          </td>
        </tr>
        <tr v-if="this.cur_dir[this.selected_index].type == 1">
          <td>大小</td>

          <td>{{ this.cur_dir[this.selected_index].size + " KB" }}</td>
        </tr>
        <tr>
          <td>路径</td>

          <td>{{ this.cur_dir[this.selected_index].path }}</td>
        </tr>
        <tr v-if="this.cur_dir[this.selected_index].type == 1">
          <td>起始块号</td>

          <td>{{ this.cur_dir[this.selected_index].p_begin }}</td>
        </tr>
        <tr v-if="this.cur_dir[this.selected_index].type == 1">
          <td>终止块号</td>

          <td>{{ this.cur_dir[this.selected_index].p_end }}</td>
        </tr>
      </table>
    </el-dialog>
    <!-- 新建文件夹对话框 -->
    <el-dialog v-model="show_dialog" title="新建文件(夹)">
      <el-form :model="new_doc_name">
        <el-form-item label="请输入文件(夹)名称" :label-width="formLabelWidth">
          <el-input v-model="new_doc_name" autocomplete="off" />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="show_dialog = false">取消</el-button>
          <el-button type="primary" @click="addDoc">确认</el-button>
        </span>
      </template>
    </el-dialog>
    <!-- 删除文件夹/格式化对话框 -->
    <el-dialog v-model="show_delete" title="确认">
      <span>"确认要删除文件(夹)/格式化磁盘吗？此操作不可逆！"</span>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="show_delete = false">取消</el-button>
          <el-button type="primary" @click="delete_Check">确认</el-button>
        </span>
      </template>
    </el-dialog>
    <!-- 文件内容对话框 -->
    <el-dialog v-model="show_content" :title="this.open_doc_name" fullscreen>
      <el-input v-model="open_doc_content" :rows="28" type="textarea" />
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="show_content = false">退出</el-button>
          <el-button type="primary" @click="saveDoc">保存并退出</el-button>
        </span>
      </template>
    </el-dialog>
    <!-- 主页面内容 -->
    <el-container>
      <el-header class="header">
        <div id="title">
          <span>文件资源管理器</span>
        </div>
      </el-header>
      <el-main class="main">
        <div class="OperationField">
          <el-button-group class="ml-4 MineIconGroup" style="">
            <el-button
              type="primary"
              plain
              :disabled="this.cur_path.length == 0"
              @click="backForward"
              ><el-icon><Back /></el-icon
            ></el-button>
            <el-button
              type="primary"
              plain
              :disabled="this.cur_path.length == 0"
              @click="this.setNewFolder(1)"
              ><el-icon><DocumentAdd /></el-icon
            ></el-button>
            <el-button
              type="primary"
              plain
              :disabled="this.cur_path.length == 0"
              @click="this.setNewFolder(0)"
              ><el-icon><FolderAdd /></el-icon
            ></el-button>
            <el-button type="primary" plain @click="this.show_drawer = true"
              ><el-icon><Files /></el-icon
            ></el-button>
          </el-button-group>
          <span class="mt-4 MineSearch">
            <el-input v-model="search_path" class="input-with-select">
              <template #append>
                <el-button @click="this.toPath">
                  <el-icon><ArrowRightBold /></el-icon>
                </el-button>
              </template>
            </el-input>
          </span>
        </div>
        <div class="PageField">
          <el-table :data="cur_dir" style="width: 100%">
            <!-- <el-table :data="cur_dir" style="width: 100%" v-menus:right="this.menus"> -->
            <el-table-column label="名称" width="300">
              <template #default="scope">
                <div
                  style="display: flex; align-items: center"
                  @click.stop
                  @contextmenu="rightClick(scope.row.name, scope.row.type)"
                  @dblclick="doubleClick(scope.row.name, scope.row.type)"
                >
                  <img :src="this.icon_urls[scope.row.type]" class="MineIcon" />
                  <span style="margin-left: 10px">{{ scope.row.name }}</span>
                </div>
              </template>
            </el-table-column>

            <el-table-column
              prop="last_edit_timestr"
              label="修改日期"
              width="300"
            />
            <el-table-column label="类型" width="300">
              <template #default="scope">
                <span>{{
                  scope.row.type == 0
                    ? "文件夹"
                    : scope.row.type == 1
                    ? "文件"
                    : "磁盘"
                }}</span>
              </template>
            </el-table-column>
            <el-table-column prop="size" label="大小(KB)" width="300" />
          </el-table>
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import { menusEvent } from "vue3-menus";
import { ElNotification } from "element-plus";
import { ElMessage } from "element-plus";
export default {
  watch: {
    cur_path: {
      handler(val) {
        this.search_path = "";
        for (let i = 0; i < val.length; ++i) {
          this.search_path += val[i] + "\\";
        }
      },
      deep: true,
      immediate: true,
    },
  },
  data() {
    return {
      /*主要配置*/
      //当前所在路径
      cur_path: [],
      //当前所在目录
      cur_dir: [],
      //文件目录——采用树的数据结构
      totol_dir: [
        {
          name: "Data(D:)",
          last_edit_timestr: "2001/10/23",
          last_edit_time: -1,
          type: 2,
          size: "64",
          used_space: "",
          path: "",
          children: [],
          p_begin: -1, //非文件类型不需要设置此项
          p_end: -1, //非文件类型不需要设置此项
        },
      ],
      //一个磁盘64 KB，分为128块，每块大小为 512 B
      disk_bitmap: [
        {
          disk: "D",
          bolck_size: 512,
          block_free_num: 512,
          bitmap:
            "00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
        },
      ],
      //在此模拟外部的硬盘，在created钩子中初始化
      physical_disk: [],
      /*对话框配置 */
      //路径跳转
      search_path: "",
      //新建文件
      show_dialog: false,
      new_doc_name: "",
      new_doc_type: -1, //0-文件夹 1-文件
      new_doc_time: -1, //存放时间戳
      new_doc_timestr: "",
      //打开文件
      show_content: false,
      open_doc_name: "",
      open_doc_content: "",
      open_doc_index: -1,
      save_doc_time: -1,
      save_doc_timestr: -1,
      //确认删除
      show_delete: false,
      /*磁盘页面配置*/
      show_drawer: false,
      //属性页面
      show_attr: false,
      /*资源文件目录*/
      icon_urls: [
        require("../assets/folder.png"),
        require("../assets/document.png"),
        require("../assets/disk.png"),
      ],
      /*右键菜单配置*/
      selected_index: -1,
      menus: [
        {
          label: "属性",
          tip: "",
          click: () => {
            this.show_attr = true;
          },
        },
        {
          label: "删除/格式化",
          tip: "删除文件夹/格式化磁盘",
          click: () => {
            this.show_delete = true;
          },
        },
      ],
    };
  },
  methods: {
    /*资源管理器初始化配置*/
    //模拟的物理磁盘初始化——实际中并没有这一步
    physicalDiskInit() {
      for (let i = 0; i < 64; ++i) {
        this.physical_disk.push({
          block_num: i,
          content: "",
          des_content: "", //便于在侧栏展示内容，只展示前10个字符
          disk_next: -1,
        });
      }
    },
    /*文件管理操作*/

    //新建文件——0为文件夹 1为文件
    addDoc() {
      let dir = this.totol_dir; //保存当前层的情况
      //这里的i相当于深度
      for (let i = 0; i < this.cur_path.length - 1; ++i) {
        //这里的j是为了遍历每一层的子结点
        for (let j = 0; j < dir.length; ++j) {
          if (dir[j].name == this.cur_path[i] && dir[j].type != 1) {
            //一定可以找到一个结点

            dir = dir[j].children; //dir定位到前一层
          }
        }
      }
      //查询当前层哪个结点为要添加结点的父节点
      let index = -1;
      console.log(222222);
      console.log(dir);
      for (let i = 0; i < dir.length; ++i) {
        if (
          dir[i].name == this.cur_path[this.cur_path.length - 1] &&
          (dir[i].type == 2 || dir[i].type == 0)
        ) {
          index = i;
        }
      }
      if (index != -1) {
        this.new_doc_name = this.new_doc_name.trim();
        //检测名字合法性
        if (this.new_doc_name == "") {
          if (this.new_doc_type == 0) {
            ElMessage.error("文件夹名不能为空！");
          } else {
            ElMessage.error("文件名不能为空！");
          }
          return;
        }
        //检测是否重名
        for (let i = 0; i < dir[index].children.length; ++i) {
          if (
            dir[index].children[i].name == this.new_doc_name &&
            dir[index].children[i].type == this.new_doc_type
          ) {
            if (this.new_doc_type == 0) {
              ElMessage.error("已存在同名文件夹！");
            } else {
              ElMessage.error("已存在同名文件！");
            }
            this.new_doc_name = "";
            this.new_doc_type = -1;
            return;
          }
        }
        //生成文件路径
        let new_doc_path = "";
        for (let i = 0; i < this.cur_path.length; ++i) {
          new_doc_path += this.cur_path[i] + "\\";
        }
        let doc_size = 0;
        if (this.new_doc_type == 0) {
          doc_size = "-";
        }
        dir[index].children.push({
          name: this.new_doc_name,
          last_edit_timestr: this.new_doc_timestr,
          last_edit_time: this.new_doc_time,
          type: this.new_doc_type,
          size: doc_size,
          path: new_doc_path,
          children: [],
          p_begin: -1, //非文件类型不需要设置此项
          p_end: -1, //非文件类型不需要设置此项
        });
        dir[index].last_edit_timestr = this.new_doc_timestr;
        dir[index].last_edit_time = this.new_doc_time;
        this.cur_dir = [].concat(dir[index].children); //更新当前目录
      } else {
        alert("下标查找有问题");
        return;
      }
      this.new_doc_name = "";
      this.new_doc_type = -1;
      this.show_dialog = false;
      ElMessage({
        message: "创建成功！",
        type: "success",
      });
      return;
    },

    //输入新文件夹信息
    setNewFolder(type) {
      let a = new Date().toLocaleDateString();
      this.show_dialog = true;
      this.new_doc_type = type;
      this.new_doc_time = new Date(a) / 1000;
      this.new_doc_timestr = a;
    },
    //打开文件
    openFile(index) {
      this.open_doc_name = this.cur_dir[index].name;
      this.open_doc_index = index;
      this.show_content = true;
      this.open_doc_content = this.readDisk(
        this.cur_dir[index].p_begin,
        this.cur_dir[index].p_end
      );
    },
    //保存文件并退出
    saveDoc() {
      if (this.writeOutDisk()) {
        //存储至“磁盘”——若成功则返回true，空间不足则返回false
        this.cur_dir[this.open_doc_index].size =
          this.open_doc_content.length / 1024;
        let a = new Date().toLocaleDateString();
        this.cur_dir[this.open_doc_index].last_edit_time = new Date(a) / 1000;
        this.cur_dir[this.open_doc_index].last_edit_timestr = a;
        ElMessage({
          message: "更改已保存！",
          type: "success",
        });
        this.show_content = false;
      } else {
        ElMessage.error("磁盘空间不足！");
      }
    },
    //从磁盘中读出数据
    readDisk(p_begin, p_end) {
      if (p_begin == -1) {
        //说明为新文件
        return;
      }
      let content = "";

      let p_cur = p_begin;
      while (1) {
        content += this.physical_disk[p_cur].content;
        if (p_cur == p_end) {
          break;
        }
        p_cur = this.physical_disk[p_cur].disk_next;
      }
      return content;
    },
    //写出至磁盘，若完成返回true，空间不足返回false
    writeOutDisk() {
      if (this.cur_dir[this.open_doc_index].p_begin != -1) {
        //说明不是第一次写 则删除
        this.deleteFromDisk(
          this.cur_dir[this.open_doc_index].p_begin,
          this.cur_dir[this.open_doc_index].p_end
        );
        //修改目录项
        this.cur_dir[this.open_doc_index].p_begin = -1;
        this.cur_dir[this.open_doc_index].p_end = -1;
      }
      let size = this.open_doc_content.length; //假设1个字符占1个字节
      let block_size = this.disk_bitmap[0].bolck_size;
      let block_need_num = Math.ceil(size / block_size);
      var block_content_ary = []; //将字符串按块能存储的最大长度切割，子串存至数组中
      for (let i = 0; i < this.open_doc_content.length; i += block_size) {
        block_content_ary.push(this.open_doc_content.slice(i, i + block_size));
      }
      //判断空间是否充足
      if (this.disk_bitmap[0].block_free_num >= block_need_num) {
        this.disk_bitmap[0].block_free_num -= block_need_num; //修改空闲块数
        //分配空间
        let count = 0; //计数
        let last_block_index = -1; //记录上一个
        let bitmap_change_index = [];
        for (let i in this.disk_bitmap[0].bitmap) {
          if (count == block_need_num) {
            this.cur_dir[this.open_doc_index].p_end = last_block_index; //记录终止指针
            break;
          }
          if (this.disk_bitmap[0].bitmap[i] == "0") {
            if (count == 0) {
              this.cur_dir[this.open_doc_index].p_begin = i; //记录起始指针
              bitmap_change_index.push(i); //记录位图
              this.physical_disk[i].content = block_content_ary[count]; //存入外部磁盘
              this.physical_disk[i].des_content = block_content_ary[
                count
              ].slice(0, 10);
              count++;
              last_block_index = i; //记录
            } else {
              this.physical_disk[last_block_index].disk_next = i; //上一个磁盘块指向此块
              bitmap_change_index.push(i); //记录位图
              this.physical_disk[i].content = block_content_ary[count]; //存入外部磁盘
              this.physical_disk[i].des_content = block_content_ary[
                count
              ].slice(0, 10);
              count++;
              last_block_index = i; //记录
            }
          }
        }
        //修改位图
        let new_bitmap = "";
        for (let i = 0; i < this.disk_bitmap[0].bitmap.length; ++i) {
          if (bitmap_change_index.indexOf(i.toString()) != -1) {
            new_bitmap += "1";
          } else {
            new_bitmap += this.disk_bitmap[0].bitmap[i];
          }
        }

        this.disk_bitmap[0].bitmap = new_bitmap;

        return true;
      } else {
        return false;
      }
    },
    //删除文件/格式化磁盘
    deleteFile(type, delete_doc_path,delete_doc_name) {
      console.log(delete_doc_path + " " + type);
      /*确定delete_index，delete_cur_path,delete_cur_dir*/
      let delete_cur_path = delete_doc_path.split("\\"); //转为数组
      delete_cur_path.pop();
      let dir = this.totol_dir; //保存当前层的情况
      //这里的i相当于深度
      for (let i = 0; i < delete_cur_path.length - 1; ++i) {
        //这里的j是为了遍历每一层的子结点
        for (let j = 0; j < dir.length; ++j) {
          if (dir[j].name == delete_cur_path[i] && dir[j].type != 1) {
            //一定可以找到一个结点
            dir = dir[j].children; //dir定位到前一层
          }
        }
      }
      //查询当前层哪个结点为要删除结点的父节点
      let index = -1;
      for (let i = 0; i < dir.length; ++i) {
        if (
          dir[i].name == delete_cur_path[delete_cur_path.length - 1] &&
          dir[i].type != 1
        ) {
          index = i;
        }
      }
      let delete_cur_dir = [].concat(dir[index].children); //确定cur_dir
      if (index == -1) {
        alert(-1);
        return;
      }
      let delete_index = -1;
      //确定selected_index
      for (let i = 0; i < delete_cur_dir.length; ++i) {
        if (
          delete_cur_dir[i].name == delete_doc_name &&
          delete_cur_dir[i].type == type
        ) {
          delete_index = i;
          break;
        }
      }
      if (delete_index == -1) {
        alert("delete_index=" + "-1!");
      }
      /*开始删除文件*/
      if (type == 1) {
        if (delete_cur_dir[delete_index].p_begin != -1) {
          this.deleteFromDisk(
            delete_cur_dir[delete_index].p_begin,
            delete_cur_dir[delete_index].p_end
          );
        }
        for (let i = 0; i < dir[index].children.length; ++i) {
          if (
            dir[index].children[i].name == delete_cur_dir[delete_index].name &&
            dir[index].children[i].type == delete_cur_dir[delete_index].type
          ) {
            dir[index].children.splice(i, 1);
            this.cur_dir = [].concat(dir[index].children); //更新目录

            return;
          }
        }
      } else if (type == 0) {
        //删除文件夹
        let i = 0;
        while (i < delete_cur_dir[delete_index].children.length) {
          this.deleteFile(
            delete_cur_dir[delete_index].children[0].type,
            delete_cur_dir[delete_index].children[0].path,
            delete_cur_dir[delete_index].children[0].name
          );
        }
        for (let i = 0; i < dir[index].children.length; ++i) {
          if (
            dir[index].children[i].name == delete_doc_name &&
            dir[index].children[i].type == 0
          ) {
            dir[index].children.splice(i, 1);
            this.cur_dir = [].concat(dir[index].children);

            break;
          }
        }

        return;
      }
    },

    //从磁盘上删除数据、更改文件指针
    deleteFromDisk(p_begin, p_end) {
      if (p_begin == -1) {
        //说明在磁盘上无内容
        return;
      }
      let p_cur = p_begin;
      let bitmap_change_index = [];
      //修改物理块
      while (1) {
        bitmap_change_index.push(p_cur);
        this.physical_disk[p_cur].content = "";
        this.physical_disk[p_cur].des_content = "";
        if (p_cur == p_end) {
          break;
        }
        let p_next = this.physical_disk[p_cur].disk_next;
        this.physical_disk[p_cur].disk_next = -1;
        p_cur = p_next;
      }
      // //修改目录项
      // this.cur_dir[this.open_doc_index].p_begin = -1;
      // this.cur_dir[this.open_doc_index].p_end = -1;
      //改位图
      let new_bitmap = "";
      for (let i = 0; i < this.disk_bitmap[0].bitmap.length; ++i) {
        if (bitmap_change_index.indexOf(i.toString()) != -1) {
          new_bitmap += "0";
        } else {
          new_bitmap += this.disk_bitmap[0].bitmap[i];
        }
      }
      this.disk_bitmap[0].bitmap = new_bitmap;
      console.log(new_bitmap);
    },
    //磁盘格式化
    disk_Formmat() {
      (this.totol_dir = [
        {
          name: "Data(D:)",
          last_edit_timestr: "2001/10/23",
          last_edit_time: -1,
          type: 2,
          size: "64",
          used_space: "",
          path: "",
          children: [],
          p_begin: -1, //非文件类型不需要设置此项
          p_end: -1, //非文件类型不需要设置此项
        },
      ]),
        //一个磁盘64 KB，分为128块，每块大小为 512 B
        (this.disk_bitmap = [
          {
            disk: "D",
            bolck_size: 512,
            block_free_num: 512,
            bitmap:
              "00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
          },
        ]),
        (this.physical_disk = []);
      this.cur_dir = [];
      this.cur_dir.push(this.totol_dir[0]); //初始化当前目录
      this.cur_path = [];
      this.physicalDiskInit();
    },
    //删除文件对话框
    delete_Check() {
      if (this.cur_dir[this.selected_index].type == 2) {
        this.disk_Formmat();
        ElMessage({
          message: "磁盘已格式化！",
          type: "success",
        });
      } else {
        this.deleteFile(
          this.cur_dir[this.selected_index].type,
          this.cur_dir[this.selected_index].path,
          this.cur_dir[this.selected_index].name
        );
        ElMessage({
          message: "文件(夹)已删除！",
          type: "success",
        });
      }
      this.show_delete = false;
    },
    /*点击操作配置*/
    //双击操作配置
    doubleClick(name, type) {
      let index = -1; //所选文件夹（磁盘）在当前目录下的下标
      for (let i = 0; i < this.cur_dir.length; ++i) {
        if (this.cur_dir[i].name == name && this.cur_dir[i].type == type) {
          index = i;
          break;
        }
      }

      if (index != -1) {
        if (this.cur_dir[index].type == 0 || this.cur_dir[index].type == 2) {
          this.cur_path.push(name); //将当前路径更深一级
          this.cur_dir = [].concat(this.cur_dir[index].children); //将当前目录更新到选中的文件夹（磁盘）的子结点
        } else {
          this.openFile(index);
        }
      }
    },
    //右击展现菜单配置
    rightClick(name, type) {
      console.log(name);
      //查找相应下标
      for (let i = 0; i < this.cur_dir.length; ++i) {
        if (this.cur_dir[i].name == name && this.cur_dir[i].type == type) {
          this.selected_index = i;
        }
      }
      menusEvent(event, this.menus);
      event.preventDefault();
    },
    //后退到上一级
    backForward() {
      this.cur_path.pop(); //更新目录项,其中的是逐层递进
      let dir = [].concat(this.totol_dir); //保存当前层的情况
      //这里的i相当于深度
      for (let i = 0; i < this.cur_path.length; ++i) {
        //这里的j是为了遍历每一层的子结点
        for (let j = 0; j < dir.length; ++j) {
          if (dir[j].name == this.cur_path[i] && dir[j].type != 1) {
            //一定可以找到一个结点
            dir = [].concat(dir[j].children);
          }
        }
      }
      this.cur_dir = [].concat(dir);
    },
    //路径跳转
    toPath() {
      let to_path_Ary = this.search_path.split("\\");
      let dir = [].concat(this.totol_dir);
      to_path_Ary.pop();
      console.log(dir);

      console.log(to_path_Ary);
      for (let i = 0; i < to_path_Ary.length; ++i) {
        for (let j = 0; j < dir.length; ++j) {
          if (
            dir[j].name == to_path_Ary[i] &&
            (dir[j].type == 2 || dir[j].type == 0)
          ) {
            dir = [].concat(dir[j].children);
            if (i == to_path_Ary.length - 1) {
              this.cur_dir = [].concat(dir);
              this.cur_path = [].concat(to_path_Ary);
              return true;
            }
            break;
          }
        }
      }
      ElMessage.error("路径非法或不存在！(注意末尾需要加\\)");
      this.search_path = "";
      for (let i = 0; i < this.cur_path.length; ++i) {
        this.search_path += this.cur_path[i] + "\\";
      }
      return false;
    },
  },
  created() {
    this.physicalDiskInit(); //初始化硬盘
    if (localStorage.length != 0) {
      this.totol_dir = JSON.parse(localStorage.getItem("total_dir"));
      this.physical_disk = JSON.parse(localStorage.getItem("physical_disk"));
      this.disk_bitmap = JSON.parse(localStorage.getItem("disk_bitmap"));
    }
    this.cur_dir = [];
    this.cur_dir.push(this.totol_dir[0]); //初始化当前目录
  },
  mounted() {
    //浏览器退出时存储
    window.onbeforeunload = (e) => {
      e = e || window.event;
      if (e) {
        e.returnValue = "关闭提示";
      }
      localStorage.clear();
      let physical_disk_JSON = JSON.stringify(this.physical_disk);
      let totol_dir_JSON = JSON.stringify(this.totol_dir);
      let disk_bitmap_JSON = JSON.stringify(this.disk_bitmap);
      localStorage.setItem("physical_disk", physical_disk_JSON);
      localStorage.setItem("total_dir", totol_dir_JSON);
      localStorage.setItem("disk_bitmap", disk_bitmap_JSON);
      return "关闭提示";
    };
    ElNotification({
      title: "请使用同一浏览器",
      message:
        "使用同一浏览器打开，即可自动恢复上次编辑后的资源目录。关闭浏览器时无需在意浏览器提示，选择离开即可",
      type: "info",
      duration: 5000,
    });
    setTimeout(
      () =>
        ElNotification({
          title: "右击操作",
          message: "鼠标右击文件名称或图标可打开编辑菜单",
          type: "info",
          duration: 5000,
        }),
      1000
    );
    setTimeout(
      () =>
        ElNotification({
          title: "双击操作",
          message: "鼠标双击文件名称或图标可打开文件(夹)",
          type: "info",
          duration: 5000,
        }),
      2000
    );
  },
};
</script>

<style scoped>
.MineSearch {
  margin-left: 10px;
  margin-top: 4px;
  width: 80%;
}
.MineIconGroup {
  margin-left: 10px;
  margin-top: 4px;
}
.MineIcon {
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
  height: 43px;
}
.main {
  background-color: #e5eaf3;
  height: 650px;
  padding: 10px;
}
.PageField {
  height: 90%;
  background-color: white;
  border-radius: 10px;
}
.OperationField {
  display: flex;
  background-color: white;
  margin-bottom: 10px;
  height: 40px;
}
#title {
  padding-top: 10px;
}
.PageField >>> .el-table__row > td {
  border: none;
  height: 0;
}
#DrawerButton {
  right: 15px;
}
#MineTable td {
  text-align: left;
}
</style>
