使用场景，用到Circos.ca网站分析Wos论文国家字段

1.在wos网站中找到要分析的论文，导出Tab分隔符格式

2.用excel中打开，选择分隔符模式

3.用下面代码处理该数据文件，得到共现矩阵的excel表

4.excel表保存为tab分隔符格式的txt文件

5.在txt文件开头输入字符，因为circos格式需要设置

```
import os
import output as output
import pandas as pd
import numpy as np
import networkx as nx
import xlwt
from pandas import DataFrame

os.chdir('/Users/weirdchun/Downloads')
data = pd.read_table("savedrecs1.txt",sep="    ",index_col=False)  #找到源文件
df_gj = (data["C1"] + '; [').str.findall("(\w*);\s\[")  # 从原始文件找到C1栏分出国家,现在是dataframe类型，可以导出excel        extractall/findall
#正则表达式， \w 表示匹配任意字母数字下划线 .\s 表示匹配任何空白字符
#上述表达为 单词 + ; + 空格 + [

print('一列多少条信息',len(df_gj))
df_gj.to_excel("/Users/weirdchun/Downloads/df_gj.xlsx")#导成Excel格式
print('查看类型：',type(df_gj)) #DataFrame类型

guojia_list = [] #创建一个新列表list类型

for i in range(len(df_gj) - 1):   #从0开始，遍历整个数据
    n = len(df_gj[i])  # 多少个国家
    print(df_gj[i])
    print(n)
    for j in range(n):
        c1_list = df_gj[i][j]
        guojia_list.append(c1_list)
        print(guojia_list)

#要算出现在有多少个国家，注意：有些文章没有国家信息，会导致错误,可以先在excel里面先处理空格，然后保存再运行代码

guojia_list_unique = list(set(guojia_list)) #去重复的学科，放在list列表里面
print("共现的gj类别数量有：",len(guojia_list_unique))
#web of science上边导出的学科包含二级学科，如果想用一级学科的话需要替换学科名，把逗号后边的二级删掉
guojia_frame = pd.DataFrame(np.zeros([len(guojia_list_unique),len(guojia_list_unique)]),columns=guojia_list_unique,index=guojia_list_unique)#构建了都是0的矩阵
guojia_frame.to_excel("/Users/weirdchun/Downloads/gjcooc.xlsx")#导成Excel格式
print(type(df_gj))
df_gj=DataFrame(df_gj) #把list变成DataFrame格式,才能继续下面的操作
for gj in df_gj.C1:   #从原始表里面遍历每列数据
    print(df_gj.C1[0])
    print(df_gj.C1[1])
    print(len(gj))
    print(gj)
    for i in range(len(gj)-1):     #第一遍，范围0到0，取i=0 ；第2遍，范围从0-1
        for j in range(i+1,len(gj)): #第一遍，从1到1，j=1；第2遍，范围从1-2
            guojia_frame.loc[[gj[i]],[gj[j]]]=guojia_frame.loc[[gj[i]],[gj[j]]]+1 #往矩阵加数据   定位位置loc[行，列] 然后 +1
guojia_frame.to_excel("/Users/weirdchun/Downloads/GJcooc.xlsx")#导成Excel格式
print("excel ok")
```
