---
layout: post
title: "Python数据转换脚本"
date: 2018-01-31 09:00:00 +0800 
categories: 研究生涯
tag: Python
---
* content
{:toc}
为了同学们的毕设，数据是个问题。写个通用爬虫，再写个通用转换器吧。这次就是一个超简单的转换脚本。麻雀虽小，五脏俱全。
这次用的案例是地理位置转换的，物流专业嘛，可能很多数据都跟位置相关，，但其实很多位置是不用经纬度方式存放的，geohash是个很常用的方法。这次就是个解码脚本。
结构如下：
1，基本模块的引入，用什么引用什么（如用Excel相关的东西，就需要引入别的模块了）
2，文件记录行内字段转换函数
3，文件输入输出，批量替换开始的函数

**代码：**

```python
#coding:utf-8
import csv
import math

#geohash模块提取的
__base32 = '0123456789bcdefghjkmnpqrstuvwxyz'
__decodemap = { }
for i in range(len(__base32)):
    __decodemap[__base32[i]] = i
del i

#返回 精确的经纬度和误差
def decode_exactly(geohash):
    lat_interval, lon_interval = (-90.0, 90.0), (-180.0, 180.0)
    lat_err, lon_err = 90.0, 180.0
    is_even = True
    for c in geohash:
        cd = __decodemap[c]
        for mask in [16, 8, 4, 2, 1]:
            if is_even: # adds longitude info
                lon_err /= 2
                if cd & mask:
                    lon_interval = ((lon_interval[0]+lon_interval[1])/2, lon_interval[1])
                else:
                    lon_interval = (lon_interval[0], (lon_interval[0]+lon_interval[1])/2)
            else:      # adds latitude info
                lat_err /= 2
                if cd & mask:
                    lat_interval = ((lat_interval[0]+lat_interval[1])/2, lat_interval[1])
                else:
                    lat_interval = (lat_interval[0], (lat_interval[0]+lat_interval[1])/2)
            is_even = not is_even
    lat = (lat_interval[0] + lat_interval[1]) / 2
    lon = (lon_interval[0] + lon_interval[1]) / 2
    return lat, lon, lat_err, lon_err

#orderid	userid	bikeid	biketype	starttime	geohashed_start_loc	geohashed_end_loc
#1893973	451147	210617	2	2017/5/14 22:16	wx4snhx	wx4snhj
#4657992	1061133	465394	1	2017/5/14 22:16	wx4dr59	wx4dquz

#2002997 test    3214097 train

def replace_train(trainfile='./input/train.csv'):
    tr_output = open('./output/train_output.csv', 'w')
    tr = csv.DictReader(open(trainfile))
    
    i=0
    for rec in tr:
        rec['geohashed_start_loc'] = decode_exactly(rec['geohashed_start_loc']);
        rec['geohashed_end_loc'] = decode_exactly(rec['geohashed_end_loc']);
        tr_output.write(str(rec)+"\n")
        i=i+1
        if i == 1000000:
            break
        else:
            if i % 100000 == 0:
                print(i)
            else:
                continue

    tr_output.close()
    print('##########',i,'##########')

    replace_test()

def replace_test(testfile='./input/test.csv'):
    tr_output = open('./output/test_output.csv', 'w')
    tr = csv.DictReader(open(testfile))
    i=0
    for rec in tr:
        rec['geohashed_start_loc'] = decode_exactly(rec['geohashed_start_loc']);
        tr_output.write(str(rec)+"\n")
        i=i+1
        if i == 1000000:
            break
        else:
            if i % 10000 == 0:
                print(i)
            else:
                continue
    tr_output.close()
    print('##########',i,'##########')

if __name__ =="__main__":
    print('##########start##########')
    replace_train()
```