# R-data.table
R data.table 高级教程： .SD

.SD是data.table中的一个属性参数，意思就是**S**ubset-of-**D**ata.table，data.table的子集。

基本用法如下：
```r
# 创建data.table
set.seed(45L)
dat = data.table(V1=c("张三", "李四", "王五"), V2=round(rnorm(12), 4), V3=LETTERS[1:2])
print(dat)
#       V1      V2 V3
#  1: 张三  0.3408  A
#  2: 李四 -0.7033  B
#  3: 王五 -0.3795  A
#  4: 张三 -0.7460  B
#  5: 李四 -0.8981  A
#  6: 王五 -0.3348  B
#  7: 张三 -0.5014  A
#  8: 李四 -0.1745  B
#  9: 王五  1.8090  A
# 10: 张三 -0.2301  B
# 11: 李四 -1.1304  A
# 12: 王五  0.2160  B

# 获取V2的平均值
dat[, .(average_V2=mean(V2))]
#    average_V2
# 1: -0.2276917

# 【上次讲的写法】获取张三、李四、王五，各自的V2平均值
dat[, .(average_V2=mean(V2)), by=V1]
#      V1 average_V2
# 1: 张三  -0.284175
# 2: 李四  -0.726575
# 3: 王五   0.327675

# 【用.SD的写法】获取张三、李四、王五，各自的V2平均值
dat[, .SD[, .(average_V2=mean(V2))], by=V1]
#      V1 average_V2
# 1: 张三  -0.284175
# 2: 李四  -0.726575
# 3: 王五   0.327675
```
这两种写法输出结果一样，而且.SD还更复杂。

我们比较一下两种写法的思路：
```r
# 第一种，在后面添加了by=V1即可
dat[, .(average_V2=mean(V2))]
dat[, .(average_V2=mean(V2)), by=V1]

# 第二种，就是把dat换成了.SD，然后放在dat[, * ,by=V1]里面*的位置
# 也就是说，在dat[, * ,by=V1]语法的*的位置，.SD指代dat按照by=V1分组后的子集
      dat[, .(average_V2=mean(V2))]
dat[, .SD[, .(average_V2=mean(V2))], by=V1]
```

第二种写法虽然复杂，但是用处更广。.SD的作用就是，任何对整个data.table的操作（例如上次讲的所有操作），都可以作用到其子集上。
下面举一些例子：
```r
      print(dat)            # 打印整个data.table
dat[, print(.SD), by=V1]    # 分别打印按不同V1分组后的data.table子集
#         V2 V3
# 1:  0.3408  A
# 2: -0.7460  B
# 3: -0.5014  A
# 4: -0.2301  B
#         V2 V3
# 1: -0.7033  B
# 2: -0.8981  A
# 3: -0.1745  B
# 4: -1.1304  A
#         V2 V3
# 1: -0.3795  A
# 2: -0.3348  B
# 3:  1.8090  A
# 4:  0.2160  B
# Empty data.table (0 rows) of 1 col: V1

      dat[c(1,.N)]          # 获取dat的第一行和最后一行
dat[, .SD[c(1,.N)], by=V1]  # 获取每个分组的第一行和最后一行
#      V1      V2 V3
# 1: 张三  0.3408  A
# 2: 张三 -0.2301  B
# 3: 李四 -0.7033  B
# 4: 李四 -1.1304  A
# 5: 王五 -0.3795  A
# 6: 王五  0.2160  B

      lapply(dat[,.(V2, V3)], max)                        # 求V2和V3中最大的值
dat[, lapply(.SD[,.(V2, V3)], max), by=V1]                # 对每个子集求V2和V3中最大的值
dat[, lapply(.SD, max), by=V1, .SDcols=c("V2","V3")]      # 简化的写法
#      V1      V2 V3
# 1: 张三  0.3408  B
# 2: 李四 -0.1745  B
# 3: 王五  1.8090  B
```

但是，也有例外。.SD不能用于赋值操作，否则会报错
```r
      dat[, V2:=V2+1]
dat[, .SD[, V2:=V2+1], by=V1]
# Error in `[.data.table`(.SD, , `:=`(V2, V2 + 1)) : 
#  .SD is locked. Using := in .SD's j is reserved for possible future use; a tortuously flexible way to modify by group. Use := in j directly to modify by group by reference.
```

最后，当.SD不和by=一起用的时候，.SD就是原来的整个data.table
```r
print(dat)
print(dat[, .SD])
# 输出结果相同
identical(dat, dat[, .SD])
# identical函数，对比两个变量是不是相等
# [1] TRUE
```
