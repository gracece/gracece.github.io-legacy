---
layout: post
title: "用python复习一下算法"
description: "和同学一起每天写一小段算法～"
category: python
tags: 
- python
- 算法
---
{% include JB/setup %}

最近是各种实习信息铺天盖地的节奏，到了这个时候才感觉到自己在算法方面还差得很远。另外几个同学也有这样的感觉，所以就
说来干点有意义的事，每天写一个小算法，简单也好，复杂也罢，总之是要保持算法的思维。因为开始都是比较简单的算法，就直接贴在
这下面了。他们用c++写，我用python写。

### 堆排序
```python
#!/usr/bin/python
#coding:utf-8
# glygo@qq.com
# 2014-04-08

def heap_adjust(data,i,size):
    left = 2*i
    right = 2*i+1
    largest = i
    if left<=size and data[left-1] > data[i-1]:
        largest = left
    if right<=size and data[right-1] > data[largest-1]:
        largest = right
    if largest <> i:
        data[i-1],data[largest-1] = data[largest-1],data[i-1]
        heap_adjust(data,largest,size)

def heap_sort(data):
    size = len(data)
    for i in range(size/2,0,-1):
        heap_adjust(data,i,size)

    for n in range(size,1,-1):
        data[1-1],data[n-1] = data[n-1],data[1-1]
        heap_adjust(data,1,n-1)

if __name__ == '__main__':
    data= [4,1,12,3,112,15,1121,44,3,44,666]
    heap_sort(data)
    print data
```
###归并排序
在写归并排序的时候就真真切切感受到了python切片功能的强大，可以少写好几个循环，同时代码可读性也不受影响。

```python
#!/usr/bin/python
#coding:utf-8
# glygo@qq.com

def merge(left,right):
    result = []
    i,j = 0,0
    while i<len(left) and j <len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i+=1
        else:
            result.append(right[j])
            j+=1
    if i<len(left):
        result += left[i:]
    if j<len(right):
        result += right[j:]
    return result

def mergesort(seq):
    if len(seq)<=1:
        return seq
    mid = int(len(seq)/2)
    left = mergesort(seq[:mid])
    right = mergesort(seq[mid:])
    return merge(left,right)

def main():
    seq = [3123,12,3,56,78,23,213234,11,66,34]
    print mergesort(seq)

if __name__ == '__main__':
    main()

```
###全排列
```python
#!/usr/bin/python
#coding:utf-8
# glygo@qq.com

def perm(list_info,k,m):
    if k == m-1:
        for i in range(m):
            print list_info[i],
        print
    else:
        for i in range(k,m):
            list_info[k],list_info[i]=list_info[i],list_info[k] 
            perm(list_info,k+1,m)
            list_info[k],list_info[i]=list_info[i],list_info[k]
if __name__ == "__main__":
    list_info = ['g','r','a','c','e']
    perm(list_info,0,5)

```
###快速排序
```python
#!/usr/bin/python
#coding:utf-8
# glygo@qq.com
# 2014-04-06


def partition(data,left,right,pivotIndex):
    pivotValue = data[pivotIndex]
    data[pivotIndex],data[right] = data[right],data[pivotIndex]
    storeIndex = left
    for i in range(left,right):
        if data[i] < pivotValue:
            data[storeIndex],data[i] = data[i],data[storeIndex]
            storeIndex = storeIndex + 1
    data[right],data[storeIndex] = data[storeIndex],data[right]
    return storeIndex

def quicksort(data,left,right):
    if left<right:
        pivotIndex = left
        pivotNewIndex = partition(data,left,right,pivotIndex)
        quicksort(data,left,pivotNewIndex-1)
        quicksort(data,pivotNewIndex+1,right)


if __name__ == '__main__':
    data = [33,41,2,4,2,3,234231,23,4,787,98,67,5634,45,6,8,9,55,322,1]
    size = len(data)
    print data
    quicksort(data,0,size-1)
    print data

```
###拓扑排序
```python
#!/usr/bin/python
#coding:utf-8
# glygo@qq.com

def toposort(neighbour,indegree):
    queue = []
    arcNum = len(neighbour)
    result = []
    for i in indegree:
        if indegree[i] == 0:
            queue.append(i)
    while queue :
        v = queue.pop(0)
        result.append(v)
        for j in range(0,arcNum):
            if neighbour[j][0]==v:
                w = neighbour[j][1]
                indegree[w] = indegree[w] - 1
                if indegree[w] == 0:
                    queue.append(w)
    return result

def main():
    arcNum = int(raw_input("Please input arcNum:"))
    neighbour = []
    indegree = {}
    for i in range(arcNum):
        v,w = raw_input("").split()
        v,w = int(v),int(w)
        if v not in indegree:
            indegree[v] = 0
        if w not in indegree:
            indegree[w] = 0
        neighbour.append([v,w])
        indegree[w] = indegree[w] + 1
    result = toposort(neighbour,indegree)
    print "the topologic sort result is ", result

if __name__ == '__main__':
    main()
```
