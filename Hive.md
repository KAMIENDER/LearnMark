# Hive

### 窗口函数

参考知乎：https://zhuanlan.zhihu.com/p/113245904

格式：

```mysql
函数(原有的某一列属性) over (partition by 属性、属性 order by 属性 {窗口子句 (rows between .. and ..)}) as 列名
```

常用函数：

* 聚合类

avg() sum() max() min()

* 排名类

row_number()排名不会重复

rank()相等时重复并且留出空位

dense_rank()相等时重复，不会留出空位

* 其他

lag(属性，往前的行数) 可以计算得到前多少次的结果

lead(属性，往后的行数) 与上一个相反

ntile(n) 均分成多少片，返回当前行所在的片编号



other：https://zhuanlan.zhihu.com/p/75550159