# R for Regression (using data.frame)
R 统计基础：回归分析

P.S. 由于数据量不大，本次课程都用data.frame，而不用data.table。data.table相关知识在以后介绍。

## Getting started
1. 下载本次课程的数据集[Rstatistics.zip](http://tutorials.iq.harvard.edu/R/Rstatistics.zip), 并解压到桌面上。

2. 然后将R的工作区设置为下载的数据集文件夹：
```r
setwd("C:/Users/niyayu/Desktop/Rstatistics")

getwd()
# 输出当前的工作空间
# [1] "C:/Users/niyayu/Desktop/Rstatistics"

list.files(".")
# 显示工作空间中的文件和文件夹
# [1] "dataSets"          "images"            "Rplots.pdf"        "Rstatistics.html"  "Rstatistics.ipynb" "Rstatistics.md"    "Rstatistics.org"   "Rstatistics.R"     "Rtips.pdf"

list.files("dataSets")
# 显示工作空间中dataSets文件夹中的所有文件
# [1] "Exam.rds"          "NatHealth2008MI"   "NatHealth2011.rds" "states.dta"        "states.rds"
```

3. 读取数据集：dataSets/states.rds
```r
states.data = readRDS("dataSets/states.rds") 

states.info = data.frame(attributes(states.data)[c("names", "var.labels")])
show(states.info)
# 显示数据集内容
# 可以看到，一共有21个数据集，names是数据集的名称，var.labels是数据集的介绍
#      names                      var.labels
# 1    state                           State
# 2   region             Geographical region
# 3      pop                 1990 population
# 4     area         Land area, square miles
# 5  density          People per square mile
# 6    metro Metropolitan area population, %
# 7    waste    Per capita solid waste, tons
# 8   energy Per capita energy consumed, Btu
# 9    miles    Per capita miles/year, 1,000
# 10   toxic Per capita toxics released, lbs
# 11   green Per capita greenhouse gas, tons
# 12   house    House '91 environ. voting, %
# 13  senate   Senate '91 environ. voting, %
# 14    csat        Mean composite SAT score
# 15    vsat           Mean verbal SAT score
# 16    msat             Mean math SAT score
# 17 percent       % HS graduates taking SAT
# 18 expense Per pupil expenditures prim&sec
# 19  income Median household income, $1,000
# 20    high             % adults HS diploma
# 21 college         % adults college degree
```

## Tutorials 1: SAT分数和学费之间的相关性

1. 通常，在做回归分析之前，都会对数据做一些初步的观察
```r
# 将expense和csat单独存放在data.frame中
sts.ex.sat <- subset(states.data, select = c("expense", "csat"))

summary(sts.ex.sat))
# summary函数，可以显示data.frame的大概分布
#     expense          csat       
#  Min.   :2960   Min.   : 832.0  
#  1st Qu.:4352   1st Qu.: 888.0  
#  Median :5000   Median : 926.0  
#  Mean   :5236   Mean   : 944.1  
#  3rd Qu.:5794   3rd Qu.: 997.0  
#  Max.   :9259   Max.   :1093.0  

cor(sts.ex.sat) 
# cor函数可以显示数据集之间的相关性
# 可以看到expense和csat的相关度是-0.46629778
#            expense       csat
# expense  1.0000000 -0.4662978
# csat    -0.4662978  1.0000000

plot(sts.ex.sat)
# plot函数可以将data.frame可视化出来，如下图
```
![alt text](pic1.png)

2. 计算线性回归: lm

用lm函数可以计算不同变量之间的线性回归。例如要计算expense和csat之间的线性回归：
```r
sat.mod <- lm(formula=csat ~ expense, data=states.data)
# lm函数有两个参数：
# csat ~ expense，表示要计算csat是因变量、expense是自变量的线性回归
# data=states.data，表示用states.data的数据来计算线性回归

show(sat.mod)
# 显示线性回归的结果
# Call:
# lm(formula = csat ~ expense, data = states.data)

# 意思是：csat = -0.02228 * expense + 1060.73244
# Coefficients:
# (Intercept)      expense  
#  1060.73244     -0.02228

summary(sat.mod)
# 显示线性回归的详细信息
# Call:
# lm(formula = csat ~ expense, data = states.data)

# 回归误差的大致分布
# Residuals:
#      Min       1Q   Median       3Q      Max 
# -131.811  -38.085    5.607   37.852  136.495 

# 回归系数说明
# 除了显示了expense的系数是-2.228e-02，截距是1.061e+03
# 和show(sat.mod)相比，多显示了三个数值：
#   Std. Error：该系数的标准差，也就是系数可能的上下浮动范围
#   t value：   t假设检验的值
#   Pr(>|t|)：  t假设检验不成立的概率
# 这三个值怎么来的，可以专门讲一节课
# Coefficients:
#               Estimate Std. Error t value Pr(>|t|)    
# (Intercept)  1.061e+03  3.270e+01   32.44  < 2e-16 ***
# expense     -2.228e-02  6.037e-03   -3.69 0.000563 ***
# ---
# Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

# 回归误差说明
# 这个的具体解释，也可以讲一节课
# Residual standard error: 59.81 on 49 degrees of freedom
# Multiple R-squared:  0.2174,    Adjusted R-squared:  0.2015 
# F-statistic: 13.61 on 1 and 49 DF,  p-value: 0.0005631

```

3. 回归模型对比

上面的结果显示，expense和csat竟然是负相关，显然是不符合常理的（学费越高，分数还越低）。

原因之一，回归模型考虑的变量个数太少了，导致其他因素的影响被混合进去。

这里，我们再加一个因变量：percent，即学生参加SAT考试的比例:
```r
sat.mod2 = lm(csat ~ expense + percent, data = states.data)
summary(sat.mod2)
# Call:
# lm(formula = csat ~ expense + percent, data = states.data)

# Residuals:
#     Min      1Q  Median      3Q     Max 
# -62.921 -24.318   1.741  15.502  75.623 

# 这里可以看到，expense和csat是正相关，但是相关显著度明显降低；percent和csat相关度极高，且为负相关
# Coefficients:
#               Estimate Std. Error t value Pr(>|t|)    
# (Intercept) 989.807403  18.395770  53.806  < 2e-16 ***
# expense       0.008604   0.00 4204   2.046   0.0462 *  
# percent      -2.537700   0.224912 -11.283 4.21e-15 ***
# ---
# Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

# Residual standard error: 31.62 on 48 degrees of freedom
# Multiple R-squared:  0.7857,    Adjusted R-squared:  0.7768 
# F-statistic: 88.01 on 2 and 48 DF,  p-value: < 2.2e-16

anova(sat.mod, sat.mod2)
# anova函数，对比两个线性回归结果
# Analysis of Variance Table

# 增加一个变量后，回归误差从175306下降到了47999
# 该新增变量的F检验为127.31，拒绝F假设的概率为4.206e-15，显著度非常高
# Model 1: csat ~ expense
# Model 2: csat ~ expense + percent
#   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
# 1     49 175306                                  
# 2     48  47999  1    127307 127.31 4.206e-15 ***
# ---
# Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

```
