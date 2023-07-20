# 国家共现
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
data = pd.read_table("savedrecs.txt",sep="	",index_col=False)  #找到源文件
df_gj = (data["C1"] + '; [').str.findall("(\w*);\s\[")  # 从原始文件找到C1栏分出国家,现在是dataframe类型，可以导出excel        extractall/findall
#正则表达式， \w 表示匹配任意字母数字下划线 .\s 表示匹配任何空白字符
#上述表达为 单词 + ; + 空格 + [

print('一列多少条信息',len(df_gj))
df_gj.to_excel("/Users/weirdchun/Downloads/df_gj.xlsx")#导成Excel格式

print('查看类型：',type(df_gj)) #DataFrame类型
guojia_list = [] #创建一个新列表list类型
print("21",df_gj[21])
df_gj=df_gj.fillna('null') #将空值，赋值为null

null="null"
print("null长度",len(df_gj[21]))
for i in range(len(df_gj)):   #从0开始，遍历整个数据
    if df_gj[i] != null:
        n = len(df_gj[i])  # 计算这一列有几个国家
        print(n)
        for j in range(n):
         c1_list = df_gj[i][j]
         guojia_list.append(c1_list)

print(guojia_list)

#要算出现在有多少个国家！ 注意：有些文章没有国家信息，会导致错误

guojia_list_unique = list(set(guojia_list)) #去重复的学科，放在list列表里面
print("共现的gj类别数量有：",len(guojia_list_unique))
#web of science上边导出的学科包含二级学科，如果想用一级学科的话需要替换学科名，把逗号后边的二级删掉
guojia_frame = pd.DataFrame(np.zeros([len(guojia_list_unique),len(guojia_list_unique)]),columns=guojia_list_unique,index=guojia_list_unique)#构建了行名和列名为国家但是数值为0的矩阵
print(type(df_gj))
print(df_gj[1])
df_gj=DataFrame(df_gj) #把list变成DataFrame格式,才能继续下面的操作
for gj in df_gj.C1:   #从原始表里面遍历每列数据  第2遍的时候，gj:['Australia','Australia']
    if gj != null:  #排除null元素，否则要报错
        for i in range(len(gj)-1):     #如，gj:['Australia','Australia','Japan'],即"i"范围为[0,3),取0，1，2；即重复3次
            for j in range(i+1,len(gj)): #如，gj:['Australia','Australia','Japan'],即"j"范围为[1,3),取1，2；即重复2次
                guojia_frame.loc[[gj[i]],[gj[j]]]=guojia_frame.loc[[gj[i]],[gj[j]]]+1 #往矩阵加数据
                # 如，gj:['Australia','Australia','Japan']，即guojia_frame.loc[[gj[0]],[gj[1]]]=guojia_frame.loc[[gj[0]],[gj[1]]]+1
                #实际上就是说明，Australia与Australia共现了，在dataframe表上找到行为Australia，列为Australia的位置，然后加1
                #然后变成guojia_frame.loc[[gj[0]],[gj[2]]]=guojia_frame.loc[[gj[0]],[gj[2]]]+1
                #再然后变成guojia_frame.loc[[gj[1]],[gj[1]]]=guojia_frame.loc[[gj[1]],[gj[1]]]+1，以此类推
                #[0,1],[0,2],[0,3]遍历完后再遍历[1,1],[1,2],[1,3],以此类推来实现共现

guojia_frame.to_excel("/Users/weirdchun/Downloads/GJcooc.xlsx")#导成Excel格式
print("excel ok")
```
# 学科共现
```
import os
import pandas as pd
import numpy as np
import networkx as nx
#import matplotlib.pyplot as plt
os.chdir('D:/software/0install package/VOSviewer/WOS_co_occurrence0304')

data = pd.read_table("WOS tab0304_utf8.txt",sep="	",index_col=False)
df_wc = data[data.WC.str.contains('; ')] #筛选包含分隔符的数据，web of science是分号+空格,只有有分割符才代表是出现了两个及以上学科
print(len(data))
print(len(df_wc))#对比两个数字查看比例，大部分论文(接近50%)只有一个学科，所以学科共现不一定有意义

xueke_list = []
for wc in df_wc.WC:
    wc_list = wc.split("; ")
    xueke_list.extend(wc_list)
xueke_list_unique = list(set(xueke_list))
print(len(xueke_list_unique))

xueke_list_unique#web of science上边导出的学科包含二级学科，如果想用一级学科的话需要替换学科名，把逗号后边的二级删掉

xueke_frame = pd.DataFrame(np.zeros([len(xueke_list_unique),len(xueke_list_unique)]),columns=xueke_list_unique,index=xueke_list_unique)
for wc in df_wc.WC:
    wc_list = wc.split('; ')
    for i in range(len(wc_list)-1):
        #xueke_frame.loc[wc_list[i]][wc_list[i+1]]=xueke_frame.loc[wc_list[i]][wc_list[i+1]]+1
        for j in range(i+1,len(wc_list)):
            xueke_frame.loc[wc_list[i]][wc_list[j]]=xueke_frame.loc[wc_list[i]][wc_list[j]]+1

for k in range(len(xueke_frame)):#对角线都为0
    xueke_frame.iloc[k,k]=0

g = nx.Graph()
for i in range(len(xueke_list_unique)-1):
    xueke = xueke_list_unique[i]
    for j in range(i+1,len(xueke_list_unique)):
        w=0
        xueke2 = xueke_list_unique[j]
        w = xueke_frame.loc[xueke][xueke2]+xueke_frame.loc[xueke2][xueke]
        g.add_edge(xueke,xueke2,weight=w)
nx.write_pajek(g,"D:/software/0install package/VOSviewer/web tab.net")


