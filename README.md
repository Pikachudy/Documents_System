# 文件管理——文件系统管理

[TOC]


### 开发环境

开发语言：Javascript+html+css

开发框架：Vue.js 3.0+Element-plus

开发工具：Vue-cli、Vue-devtools、VScode、Edge

### 实现方法

引入Element-plus组件作为UI，采用Vue3框架进行组件化开发

采用 JS 在内存中开辟的空间作为文件资源管理器所需的内存部分，使用浏览器缓存（localStorage）作为外部磁盘，将文件写入的数据存至其中。

在退出资源管理器时（即关闭浏览器页面）将必要的目录文件结构（如位图、文件目录等）也一并存入浏览器缓存中，模拟关闭系统时文件存入磁盘。在下次访问时从浏览器缓存中读取目录文件结构数据，模拟进入系统后从磁盘中取出文件目录等操作。

### 提交文件说明

**dist.zip**: 项目构建后的发布版本

**DocumentsSystem**: 源代码，其中

+ ./src/App.vue

+ ./src/components/DocumentPage.vue

  为主要实现代码

### 项目浏览

+ 联网进入[[documents_system (pikachudy.github.io)](https://pikachudy.github.io/Documents_System/)](https://pikachudy.github.io/OS_MemoryScheduling/)在线浏览

+ 解压**dist**文件，在根目录下找到 **index.html** ,使用浏览器打开即可本地浏览

+ 解压**DocumentsSystem.zip**文件在根目录打开终端

  > 环境配置：
  >
  > 若未安装Node.js，需要先安装Node.js [下载 | Node.js (nodejs.org)](https://nodejs.org/zh-cn/download/)
  >
  > 若未安装Vue cli，在终端输入 `npm install -g @vue/cli` 

  运行 `npm install` 安装项目依赖包

  运行`npm run build` 构建发布版本

  在根目录下会出现**dist**文件夹，使用浏览器打开其中的 **index.html** 即可浏览

**请注意：**

+ 由于使用了浏览器缓存来存放数据，不同浏览器之间、同一浏览器不同域名下缓存不能共享，因此 **请使用同一浏览器打开项目**，以顺利读取上一次退出时保存的内容
+ 请确保浏览器**不处于无痕模式/安全模式**，以确保浏览器允许缓存写入
+ 本项目在Edge、FireFox、Chrom、Mac端Safari均进行过测试，可正常读写缓存

### 界面展示&UI说明

目录界面

![image-20220616191925859](C:\Users\焦佳宇\AppData\Roaming\Typora\typora-user-images\image-20220616191925859.png)



文件内容界面：

![image-20220616191957757](C:\Users\焦佳宇\AppData\Roaming\Typora\typora-user-images\image-20220616191957757.png)



右键菜单界面

![image-20220616192108771](C:\Users\焦佳宇\AppData\Roaming\Typora\typora-user-images\image-20220616192108771.png)



新建文件（夹）

![image-20220616192157441](C:\Users\焦佳宇\AppData\Roaming\Typora\typora-user-images\image-20220616192157441.png)



磁盘块使用详情界面：（存放内容只显示前10个字符）

![image-20220616192235397](C:\Users\焦佳宇\AppData\Roaming\Typora\typora-user-images\image-20220616192235397.png)

#### 页面UI说明

本项目主要模仿常见文件管理器进行功能设置和布局，因此不再特别说明UI功能



### 主要变量及数据结构介绍

实现功能的主要文件为**DocumentPage.vue**

#### 文件预览

| 文件名称         |         负责内容         | 子组件           |
| ---------------- | :----------------------: | ---------------- |
| App.vue          |        页面根组件        | DocumentPage.vue |
| DocumentPage.vue | 实现页面UI展示、整体逻辑 | null             |

#### 源码变量介绍

##### DocumentPage.vue

###### 维护变量

```javascript
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
          bitmap:   "00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
        },
      ],
      //在此模拟外部的硬盘，在created钩子中初始化
      physical_disk: [],
    ...
    };
  },
```

###### 基本介绍

组件内部维护了文件资源管理器所需的状态变量及必要数据结构，除了基本的状态参数外，使用了对象数组 **total_dir** 作为文件目录表，其中每个对象相当于一个 FCB ,  **disk_bitmp** 模拟使用位图来管理空闲块信息；**physical_disk** 模拟外部磁盘空间，存放文件内容

在退出系统前会将以上信息转换为 JSON 格式，存放入浏览器本地缓存中（使用 windows.localStorage 对象），在每次进入系统时将会读取缓存，恢复上次离开时的状态。

###### 主要对象属性介绍

1. **totol_dir** 使用**树的数据结构**组织 FCB 项，采用**链接结构管理文件存储空间**，其中每一项 FCB 主要包含以下属性：

   + name：文件名

   + last_edit_time:  最后修改时间的时间戳

   + type：文件类型——磁盘/文件夹/可读写文件

   + size：文件大小（以 KB 为单位）

   + path：文件路径

   + children：包含的子文件（夹）

   + p_begin：块指针，指向起始位置

   + p_end：块指针，指向终止位置




2. **disk_bitmp** 是一个数组，原本考虑到了多个磁盘（C、D、E、F），每个磁盘均有一个位图管理空闲空间，但时间原因最终只写了一个磁盘：

   + disk：磁盘号

   + block_size: 一个块的大小（单位为 B）
   + bitmap：由于 JS 无法直接操作比特位，因此使用字符串数组来模拟

   

3. **physical_disk** 是一个数组，模拟外部磁盘，其中的每一个对象元素为一个磁盘块，每个元素的属性为：

   + block_num：块号
   + content：存放内容
   + disk_next：采用链接结构管理文件存储空间，此处指向下一块块号

   由于块数比较多，不便于以字面量形式初始化，因此在 created 钩子中调用相应函数进行初始化，初始化代码如下：

```javascript
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
```



### 主要功能设计及实现

#### 管理方式

1. 文件目录——多级目录
2. 文件存储空间管理——链接结构
3. 空闲空间管理——位图

#### 新建文件

1. 由于文件目录以多级目录方式组织，因此直接遍历树找到当前目录下新建的文件所属的直接父文件夹（父结点）

``` javascript
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
```

2. 检查文件名是否冲突及合法性（同一目录下已存在同类型同名文件）
   1. 若非法则输出提示，等待新的文件名输入
   2. 若合法则进行下一步

```javascript
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
```

3. 生成文件其他信息，在文件目录下创建相应FCB

```javascript
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
```

4. 创建成功（P.S windows 11文件管理系统不会在创建文件时要求输入文本内容，因此此处也不要求，初始化文件大小均为 0 KB）

#### 打开文件

根据前端传入的打开文件名称，查找文件目录表，根据其对应的 p_begin，p_end 指针从”外部磁盘”中读取数据

```javascript
	openFile(index) {
      this.open_doc_name = this.cur_dir[index].name;
      this.open_doc_index = index;
      this.show_content = true;
      this.open_doc_content = this.readDisk(
        this.cur_dir[index].p_begin,
        this.cur_dir[index].p_end
      );
    },
```

由于打开文件事件由鼠标双击触发，因此此处传入的文件名称必然合法，且一定在 FCB 中存在（若不存在则不会显示在页面上，自然   	无法被点击），因此无需检测文件名

下面介绍 readDisk() 函数，其主要功能为根据文件块指针从“磁盘”读取数据

判断 p_begin 是否为 -1 ，若为 -1 则说明是一个空文件，直接返回即可；若不为 -1，则从起始块读数据，依据起始块中的指向下一块的指针找到下一块……直至读取完毕，即 p_begin 与 p_end 相等

```javascript
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
```

#### 保存文件

将文件数据存入磁盘，若磁盘有剩余空间则返回 true，记录相关信息并输出提示；若无剩余空间则输出提示，等待文本内容修改

```javascript
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
```

下面主要介绍下writeOutDisk()函数，其主要功能为根据文件块指针从“磁盘”读取数据

1. 查看 p_begin 是否为-1，若为-1则说明该文件之前在磁盘上没有存储的数据，则直接进行第 3 步
2. 若 p_begin 不为-1，则说明该文件之前在磁盘上存储有数据。此处设计为将磁盘上该文件的旧数据擦除，再存储新数据

``` javascript
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
```

3. 计算保存的数据所需要的磁盘块数，并将数据按块大小分隔成相应份

```javascript
	  let size = this.open_doc_content.length; //假设1个字符占1个字节
      let block_size = this.disk_bitmap[0].bolck_size;
      let block_need_num = Math.ceil(size / block_size);
      var block_content_ary = []; //将字符串按块能存储的最大长度切割，子串存至数组中
      for (let i = 0; i < this.open_doc_content.length; i += block_size) {
        block_content_ary.push(this.open_doc_content.slice(i, i + block_size));
      }
```

4. 通过位图磁盘是否有剩余空间，若不足，则返回 false 输出提示并等待文本内容修改；若充足则进行磁盘块空间分配。分配规则为通过位图寻找前N个（假设需要 N 块）空闲块，依次将数据存入，并建立块指针链接。完成后返回 true

```javascript
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
        ...
```

#### 删除文件

1. 由于文件目录以多级目录方式组织，因此直接遍历树找到当前目录下要删除的文件所属的直接父文件夹及删除过程中所需参数

``` javascript
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
```

2. 判断删除的文件类型

   若不为文件夹，且在磁盘内存有数据（p_begin 不为-1），则先根据文件目录表中 FCB 所记录的块指针，调用` deleteFromDisk(p_begin,p_end)`将磁盘对应块中的数据擦除，并取消块之间的链接

```javascript
		if (delete_cur_dir[delete_index].p_begin != -1) {
          this.deleteFromDisk(
            delete_cur_dir[delete_index].p_begin,
            delete_cur_dir[delete_index].p_end
          );
        }
```

​		在从对应的文件目录中将对应的 FCB 项去除，即从树中去除相应结点

```javascript
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
```

  		若为文件夹，则递归调用本函数，将所有的子文件删除后，再将其删除

```javascript
		...
		else if (type == 0) {
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
```

#### 数据记录/恢复

使用 **localStorage** 对象，其允许在浏览器中存储 key/value 对的数据，可长久保存整个网站的数据，保存的数据没有过期时间，直到手动去删除，所以很适合在本项目使用

##### 保存数据

在页面关闭/刷新/前进/后退时，将文件目录表、位图、用内存空间模拟的磁盘空间数据转换为 JSON 格式，存入缓存中。需要在 mounted 钩子中设置

```javascript
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
    ...
}
```

##### 读取数据

在页面打开后，变量创建完成但页面未渲染时，检测本地缓存中是否有相应数据，若有则读取数据，将读到的 JSON 格式转换回对象，完成相应状态的恢复。在created 钩子设置：

```javascript
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
```

