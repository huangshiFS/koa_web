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

#### MySQL建表

#### sequelize
+ 数据表，用JS中的模型(class或对象)代替
+ 一条或多条记录，用JS中一个对象或数组代替
+ sql语句，用对象方法代替

```
// seq.js
// 建立sequelize对象
const Sequelize = require('sequelize')

const conf = {
    host:'localhost',
    dialect:'mysql'
}

const seq = new Sequelize('koa2_weibo_db','root','password',conf)

module.exports = seq

// model.js
// 建立模型
const Sequelize = require('sequelize')
const seq = require('./seq')

// 创建User模型，数据表的名字是 users
const User = seq.define('user',{
    // id 会自动创建，并设置为主键，自增
    userName:{
        type:Sequlize.STRING,   // varchar(255)
        allowNull:false
    },
    password:{
        type:Sequelize.STRING,
        allowNull:false
    },
    nickName:{
        type:Sequelize.STRING,
        comment:'昵称'
    }

})

// 创建 Blog 模型
const Blog = seq.define('blog',{
    title:{
        type:Sequelize.STRING,
        allowNull:false
    },
    content:{
        type:Sequelize.TEXT,
        allowNull:false
    },
    userId:{
        type:Sequelize.INTEGER,
        allowNull:false
    }
})

// 外键关联
Blog.belongsTo(User,{
    // 创建外键 Blog.userId -> User.id
    foreignKey:'userId'
})
User.hasMany(Blog,{
    // 创建外键 Blog.userId -> User.id
    foreignKey:'userId'
})
module.exports = {
    User
}



// sync.js
// 同步
const seq = require('./seq')
require('./model')

// 测试连接
seq.authenticate().then(()=>{
    console.log('auth ok')
}).catch(()=>{
    console.log('auth err')
})

// 执行同步
seq.sync({force:true}).then(()=>{
    console.log('sync ok')
    process.exit()
})


// create.js
// insert ... 语句

const { Blog,User } = require('./model')

!(async function(){
    // 创建用户
    const zhangsan = await User.create({
        userName:'zhangsan',
        password:'123',
        nickName:'张三'
    })
    // insert into users (...) values (...)
    console.log('zhangsan:',zhangsan.dataValues) 
    const zhangsanId = zhangsan.dataValues.id

    const lisi = await User.create({
        userName:'lisi',
        password:'123',
        nickName:'李四'
    })
    const lisiId = lisi.dataValues.id

    // 创建博客
    const blog1 = await Blog.create({
        title:'标题1',
        content:'内容1'
        userId:zhangsanId
    })
    console.log('blog1',blog1.dataValues)
})()


// select.js
const { Blog,User } = require('./model')

!(async function(){
    // 查询一条记录
    const zhangsan = await User.findOne({
        where:{
            userName:'zhangsan'
        }
    })
    console.log('zhangsan',zhangsan.dataValues)
    
    // 查询特定的列
    const zhangsanName = await User.findOne({
        attributes:['userName','nickName'],
        where:{
            userName:'zhangsan'
        }
    })
    console.log('zhangsanName',zhangsanName.dataValues)


    // 查询一个列表
    const zhangsanBlogList = await Blog.findAll({
        where:{
            userId:1
        }
        order:[
            ['id','desc'],
        ]
    })
    console.log('zhangsanBlogList',
    zhangsanBlogList.map(blog=>blog.dataValues))


    // 分页
    const blogPageList = await Blog.findAll({
        limit:2,  // 限制本次查询2条
        offset：0，// 跳过多少条
        order:[
            ['id','desc']
        ]
    })
    console.log(
        'blogPageList',
        zhangsanBlogList.map(blog=>blog.dataValues)
    )


    // 查询总数
    const blogListAndCount = await Blog.findAndCountAll({
        limit:2,  // 限制本次查询2条
        offset:0, // 跳过多少条
        order:[
            ['id','desc']
        ]
    })
    console.log(
        'blogListAndCount',
        blogListAndCount.count,     // 所有的总数，不考虑分页
        blogListAndCount.rows.map(blog=>blog.dataValues)
    )

    // 连表查询1
    const blogListWithUser = await Blog.findAndCountAll({
        order:[
            ['id','desc']
        ],
        include:[
            {
                model:User,
                attributes:['userName','nickName'],
                where:{
                    userName:'zhangsan'
                }
            }
        ]
    })
    console.log(
        'blogListWithUser',
        blogListWithUser.count,
        blogListWithUser.rows.map(blog=>{
            const blogVal = blog.dataValues
            blogVal.user = blogVal.user.dataValues
            return blogVal
        })
    )

    // 连表查询2
    const userListWithBlog = await User.findAndCountAll({
        attributes:['userName','nickName'],
        include:[
            {
                model:Blog
            }
        ]
    })
    console.log(
        'userListWithBlog',
        userListWithBlog.count.
        JSON.stringify(userListWithBlog.rows.map(user=>{
            const userVal = user.dataValues
            userVal.blog = userVal.blog.map(blog=>blog.data.Values)
            return userVal
        }))
    )
})()



// update.js
const { user } = require('./model')
!(async function(){
    const updateRes = await User.update({
        nickName:‘张三1'
    },{
        where:{
            userName:'zhangsan'
        }
    })
    console.log('updateRes...',updateRes[0]>0)
})()


// delete.js
const { User,Blog } = require('./model')

!(async function(){
    const delBlogRes = await Blog,destroy({
        where:{
            id:4
        }
    })
    console.log('delBlogRes',delBlogres)
})()
```

#### koa2开发环境
+ 配置eslint,以及pre-commit
+ inspect调试
+ 404页和错误页
+ mysql添加用户和密码
  ```
  -- 添加新用户
  create user 'one'@'localhost'identified by '789';
  -- 允许外部IP访问
  create user 'one'@'%'identified by '789';
  -- 为新用户分配权限授予用户通过外网IP对于该数据库的全部权限
  grant all privileges on `onedb`.* to 'one'@'%' identified by '789'; 
  -- 刷新权限
  flush privileges;
  ```

#### 知识点介绍 - jest
+ 单元测试，及其意义
  - 单个功能或接口，给定输入，得到输出。看输出是否符合要求
  - 需手动编写用例代码，然后统一执行
  - 意义：能一次性执行所有单测，短时间内验证所有功能是否正常
+ 使用jest
  - *.test.js文件
  - 常用的断言
  - 测试http接口
+ 测试接口
