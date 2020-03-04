# Vue_Shop 项目笔记

框架使用 `element-ui`

## 20 实现分页效果

### 引入Pagination组件

### 添加pagesize 改变事件方法

1. size-change 属性绑定的函数回调函数返回的页容量，即pagesize
2. 在对应的方法中改变pagesize，再次请求

### 添加pagenum 改变事件方法

1. current-change 属性绑定的函数回调函数返回的页容量，即pagenum
2. 在对应的方法中改变pagenum，再次请求

## 21 修改用户转态

### 双向绑定单行数据

1. 使用 slot-scope=“scope”，得到单行数据
2. switch中双向绑定`mg_state`属性
3. 添加`change`方法，传递`scope.row`参数
4. 由于已经双向绑定数据，此时的`mg_state`已经变成的`switch`开关的状态值，利用API接口，发送数据到服务器，并获得服务器返回的状态码
5. 检查状态码，若为200，则向服务器修改数据成功，此时提示用户修改状态成功即可
6. 若状态码不为200，此时由于已经修改了本地的`mg_state`属性，需要将本地再设置为原状态，即取反操作，并提示用户修改失败

## 22 实现搜索功能

1. input 绑定 `queryInfo.query`属性
2. 搜索按钮绑定 `getUserList`方法
3. 优化清空
4. input 框添加`clearable`属性，并使`clear`事件绑定`getUserList` 方法

