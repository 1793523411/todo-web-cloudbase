# 如何在 web 应用中使用数据库

随着云时代的到来，云开发有着独特的优势相对于传统开发，从数据库而言，cloudbase 提供的云数据库真的很方便，本文就以一个简单的 todolist 小例子来讲解一下如何在 web 应用中使用云开发数据库

## 构建简单的界面

下面的这个 todolist 只要包括的功能有：增加事情，删除事情，修改事情的完成状态，以及查询事情，所有的操作都是基于云数据库的

![image-20200616114939528](https://web-sql-1301545895.cos.ap-nanjing.myqcloud.com/image-20200616114939528.png)

该界面的构建主要用到了 Vue 和 bootstrap。使用 Vue 双向数据绑定可以更方便的操作数据，使用 bootstrap 来构建好看的界面

界面构建代码如下：

```html
<div id="app">
  <div class="panel panel-primary">
    <div class="panel-heading">
      <h3 class="panel-title">web应用中使用数据库</h3>
    </div>
    <div class="panel-body form-inline">
      <label>
        Name:
        <input type="text" class="form-control" v-model="name" />
      </label>

      <input
        type="button"
        value="添加"
        class="btn btn-primary"
        @click="add()"
      />

      <label>
        搜索名称关键字：
        <input type="text" class="form-control" v-model="keywords" />
      </label>

      <input
        type="button"
        value="查找"
        class="btn btn-primary"
        @click="search(keywords)"
      />
    </div>
  </div>

  <table class="table table-bordered table-hover table-striped">
    <thead>
      <tr>
        <th>things</th>
        <th>delete</th>
        <th>finsih</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="item in list" :key="item._id">
        <td :class="[item.complete?'active':'']" v-text="item.name"></td>
        <td>
          <a href="" @click.prevent="del(item._id)">删除</a>
        </td>
        <td>
          <a href="" @click.prevent="complete(item._id,item.complete)"
            >{{item.complete?'取消完成':'已完成'}}</a
          >
        </td>
      </tr>
    </tbody>
  </table>
</div>
```

记得引入第三方文件

```html
<script src="./lib/vue-2.4.0.js"></script>
<script src="https://imgcache.qq.com/qcloud/tcbjs/1.3.5/tcb.js"></script>
<link rel="stylesheet" href="./lib/bootstrap-3.3.7.css" />
```

其中第二个就是与云开发相关的第三方文件了

> 说明：这里的 vue 和 bootstrap 文件可以通过 npm 自行下载

## 开通云开发环境

进入控制台在云产品一栏中找到:云开发->云开发cloudbase，然后点击新建环境，填写基本信息向下面这样，然后点击立即开通

![image-20200616121925902](https://web-sql-1301545895.cos.ap-nanjing.myqcloud.com/image-20200616121925902.png)

环境初始化需要一段时间，初始化完成后就可以点击进入，会看到如下界面

![image-20200616122416190](https://web-sql-1301545895.cos.ap-nanjing.myqcloud.com/image-20200616122416190.png)

我们要用的就是左侧的数据库功能，点击数据库并创建所要使用的todo集合

![image-20200616122510561](https://web-sql-1301545895.cos.ap-nanjing.myqcloud.com/image-20200616122510561.png)

至此，环境开通完成，下面开始写代码

## 配置 web 应用

要想在 web 应用中使用云数据库，首先应该进行一些基本的配置，代码如下

```javascript
const app = tcb.init({
  env: "你的环境id",
});
app
  .auth()
  .signInAnonymously()
  .then(() => {
    // alert("登录云开发成功！");
  });
const db = app.database();
const collection = db.collection("todo");
```

上面操作实现了将你的 web 应用与你的云开发环境关联，并确定要连接的数据库集合，也就是上面所创建的 todo集合

## 从数据库中获取数据

```javascript
 mounted() {
                    console.log("mounted");
                    collection.get().then((res) => {
                        // console.log(res)
                        this.list = res.data;
                    });
                },
```

在页面加载完成时获取数据库中所有数据，然后用 vue 的数据绑定渲染到页面上

## 在数据库中进行查询

```javascript
 search(keywords) {
                        console.log(keywords);
                        collection
                            .where({
                                name: {
                                    $regex: keywords + ".*",
                                },
                            })
                            .get()
                            .then((res) => {
                                console.log(res);
                                this.list = res.data;
                            });
                    },
```

上面的代码实现了从数据库中通过关键字查询 name 这个属性的值，来改变要渲染的数据，这里用到了正则匹配来进行模糊查询

![image-20200616115648937](https://web-sql-1301545895.cos.ap-nanjing.myqcloud.com/image-20200616115648937.png)

## 向数据库中增加数据

```javascript
add() {
                        collection
                            .add({
                                name: this.name,
                                complete: false,
                            })
                            .then((res) => {
                                this.name = "";
                            });
                        collection.get().then((res) => {
                            this.list = res.data;
                            this.search("")
                        });
                    },
```

这段代码实现了向数据库中添加一条数据，添加的字段中，名字从用户输入中获取，完成情况默认为未完成，并且在添加完成后重新获取所数据，并调用 search 方法来让页面的数据实时的变化,进行添加操作云数据库还会默认添加\_id 和\_openid 属性

## 实现删除数据

```javascript
 del(id) {
                        console.log(id);
                        collection
                            .doc(id)
                            .remove()
                            .then((res) => {
                                console.log(res);
                            });
                        collection.get().then((res) => {
                            this.list = res.data;
                            this.search("")
                        });
                    },
```

上面的代码是实现了根据数据的\_id 通过传参的方式从数据库中删除一条数据，并即时的展现在页面上

## 改变完成状态

```javascript
complete(id,complete){
                        console.log(id)
                        comolete = !complete
                        collection
                        .doc(id)
                        .update({
                            complete:comolete
                        })
                        collection.get().then((res) => {
                        // console.log(res)
                        this.list = res.data;
                        this.search("")
                    });
                    }
```

最后一个功能，通过点击来改变单条数据的 complete 属性值来改变完成状态，并让 thing 对应不同的样式

## 部署该应用

### 方式一

既然这样一个应用写好了，那我们能不能利用云开发cloudbase的静态网网站托管功能来部署我们应用呢？答案是可以的，点击左侧的静态网站托管，并点击开始使用，然后等待期初始化完成

![image-20200616122918313](https://web-sql-1301545895.cos.ap-nanjing.myqcloud.com/image-20200616122918313.png)

初始化完成后，我们将刚刚所写的代码和所需要的依赖文件上传到静态网站托管，向下面这样

![image-20200616123553515](https://web-sql-1301545895.cos.ap-nanjing.myqcloud.com/image-20200616123553515.png)

![image-20200616123653706](https://web-sql-1301545895.cos.ap-nanjing.myqcloud.com/image-20200616123653706.png)

然后点击上面的基础配置就可以看见域名信息处有一个默认域名，点击该默认域名，就可以访问到刚刚所写的应用了

### 方式二

除了使用上面哪种方式部署外，还有一种更简单的，那就是使用云开始提供的cli工具，需要用npm安装cloudbase/cli

[具体教程参考这里](http://docs.cloudbase.net/cli/intro.html)

当你安装好了这个工具并进行了登录，就可以可以通过命令行的形式来部署应用了，只需要执行一条命令（前提是在云开发控制台开通了静态网站服务）：`tcb hosting:deploy ./todo / -e envId`

> todo是项目的目录名，/代表云端文件根路径

最后会出现下面这样的结果



![image-20200619152741345](https://web-sql-1301545895.cos.ap-nanjing.myqcloud.com/image-20200619152741345.png)

到此就部署完成了，如果想要查看静态网站的状态，访问域名等信息，可以执行`tcb hosting:detail -e envId`

![image-20200619153218537](https://web-sql-1301545895.cos.ap-nanjing.myqcloud.com/image-20200619153218537.png)

## 完整代码

最后附上完整的代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Document</title>
    <script src="./lib/vue-2.4.0.js"></script>
    <script src="https://imgcache.qq.com/qcloud/tcbjs/1.3.5/tcb.js"></script>
    <link rel="stylesheet" href="./lib/bootstrap-3.3.7.css" />
    <style>
      .active {
        text-decoration: line-through;
        color: blueviolet;
      }
    </style>
  </head>

  <body>
    <div id="app">
      <div class="panel panel-primary">
        <div class="panel-heading">
          <h3 class="panel-title">web应用中使用数据库</h3>
        </div>
        <div class="panel-body form-inline">
          <label>
            Thing:
            <input type="text" class="form-control" v-model="name" />
          </label>

          <input
            type="button"
            value="添加"
            class="btn btn-primary"
            @click="add()"
          />

          <label>
            搜索名称关键字：
            <input type="text" class="form-control" v-model="keywords" />
          </label>

          <input
            type="button"
            value="查找"
            class="btn btn-primary"
            @click="search(keywords)"
          />
        </div>
      </div>

      <table class="table table-bordered table-hover table-striped">
        <thead>
          <tr>
            <th>things</th>
            <th>delete</th>
            <th>finsih</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in list" :key="item._id">
            <td :class="[item.complete?'active':'']" v-text="item.name"></td>
            <td>
              <a href="" @click.prevent="del(item._id)">删除</a>
            </td>
            <td>
              <a href="" @click.prevent="complete(item._id,item.complete)"
                >{{item.complete?'取消完成':'已完成'}}</a
              >
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <script>
      const app = tcb.init({
        env: "xxx",
      });
      app
        .auth()
        .signInAnonymously()
        .then(() => {
          // alert("登录云开发成功！");
        });
      const db = app.database();
      const collection = db.collection("todo");
      var vm = new Vue({
        el: "#app",
        data: {
          name: "",
          keywords: "",
          list: [],
        },
        update() {
          this.search("");
        },
        mounted() {
          console.log("mounted");
          collection.get().then((res) => {
            // console.log(res)
            this.list = res.data;
          });
        },
        methods: {
          add() {
            collection
              .add({
                name: this.name,
                complete: false,
              })
              .then((res) => {
                this.name = "";
              });
            collection.get().then((res) => {
              this.list = res.data;
              this.search("");
            });
          },
          del(id) {
            console.log(id);
            collection
              .doc(id)
              .remove()
              .then((res) => {
                console.log(res);
              });
            collection.get().then((res) => {
              this.list = res.data;
              this.search("");
            });
          },
          search(keywords) {
            console.log(keywords);
            collection
              .where({
                name: {
                  $regex: keywords + ".*",
                },
              })
              .get()
              .then((res) => {
                console.log(res);
                this.list = res.data;
              });
          },
          complete(id, complete) {
            console.log(id);
            comolete = !complete;
            collection.doc(id).update({
              complete: comolete,
            });
            collection.get().then((res) => {
              // console.log(res)
              this.list = res.data;
              this.search("");
            });
          },
        },
      });
    </script>
  </body>
</html>
```

##
