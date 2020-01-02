### 项目初始化
#### koa初始化
安装koa
```
npm install -g koa-generator
```
node版本大于等于8.0，检查node版本
```
node -v
```
#### 用ejs初始化项目
该命令自动创建文件夹以及文件
```
koa2 -e koa2-weibo-code
```
进入目录初始化
```
npm i  or
yarn i
```
#### 启动程序
```
npm run dev
```
设置环境变量
```
npm i cross-env -D
```

#### ejs格式
变量格式
```
<%= title %>
```
循环语句格式
```
<div>
    <% if (isMe) { %>
    <a href="#">@ 提到我的(3)</a>
    <% } else { %>
    <button>关注</button>
    <% } %>
</div>
```
组建化
将小功能单独放在一个文件夹下，新建一个ejs文件，然后引用，引用需要传递两个值，引用路径，变量值(可能有多个)
```
将以下代码放入views/widgets/user-info.ejs中

<div>
    <% if (isMe) { %>
    <a href="#">@ 提到我的(3)</a>
    <% } else { %>
    <button>关注</button>
    <% } %>
</div>

然后在view/index.ejs中引用
<%- include('widgets/user-info',{
    isMe
  })%>
```