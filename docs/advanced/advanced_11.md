## 操作符

#### 汇总

|  函数   | 是否扩展函数 | 函数参数 | 返回值（调用本身、最后一行） |
| :-----: | :----------: | :------: | ---------------------------- |
|  with   |     不是     |   this   | 最后一行                     |
|  T.run  |      是      |   this   | 最后一行                     |
|  T.let  |      是      |    it    | 最后一行                     |
| T.also  |      是      |    it    | 调用本身                     |
| T.apply |      是      |   this   | 调用本身                     |
|         |              |          |                              |

