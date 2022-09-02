---
layout: post
title: 记一次Go slice内存泄露踩坑
date: 2022-09-01
author: opso
tags:
 - go
 - memory leak
cover: "/images/go_logo.png"
---

项目最近上了一个服务，内存占用一直会缓慢增加，为了不影响服务稳定，避免出现 `OOM`(Out of memory)，目前临时的解决方式是启用两组服务，一组服务暴露给用户，一组作为备用，内存过大的时候手动切换到备用组上，原先那组服务等没有用户连接之后，重启释放内存，然后另一边使用各种方式查找 `内存泄露`(Memory Leak)的原因。

<!--more-->

首先观察`Prometheus`拉取的内存上报数据如图，`heap_inuse` 是缓慢增长状态

![slice-memory](/images/slice_memory.png)

之后使用 `pprof` 工具进行heap分析，发现项目中有一段数据中，对象数量`inuse_objects`异常偏高，除了连接相关的对象，不应该有1w+的对象数，于是查看代码，发现最可能的就是这里的 `fr.rules` 没有删除成功了。

```go
type Rules struct {
	rules []*TheRule
	rwm   sync.RWMutex
}

func (fr *Rules) Del {
    fr.rwm.Lock()
    defer fr.rwm.Unlock()
    for i, rule := range fr.rules {
        if rule.Typ == typ {
            fr.rules = append(fr.rules[:i], fr.rules[i+1:]...)
            break
        }
    }
}
```

测试下发现每次调用`Del`方法，`fr.rules`下的元素是能正确清理的，那是什么原因会导致对象没有释放呢？是这里的`append`后，指针数据不能被`gc`回收吗？

想起来刚学`golang`的时候的知识，`slice`语言切片是对数组的抽象，是一种"动态数组"，当我们从`slice`里用`append`这样的方式删除数据的时候，原数组里的对象指针还是会存在，即使后续没有被引用，`gc`也不会回收它，要想内存能够被回收需要手动置为`nil`，于是加了两行如下

```go
func (fr *Rules) Del {
    fr.rwm.Lock()
    defer fr.rwm.Unlock()
    for i, rule := range fr.rules {
        if rule.Typ == typ {
            prev := fr.rules
            fr.rules = append(fr.rules[:i], fr.rules[i+1:]...)
            prev[len (prev)-1] = nil
            break
        }
    }
}
```

第二种方法也可以声明一个新的`slice`，使用`copy`将删除完数据的`slice`复制到这个新的`slice`上

```go
func (fr *Rules) Del {
    fr.rwm.Lock()
    defer fr.rwm.Unlock()
    for i, rule := range fr.rules {
        if rule.Typ == typ {
            fr.rules = append(fr.rules[:i], fr.rules[i+1:]...)
            tmp := make([]*TheRule, len(fr.rules))
            copy(tmp, fr.rules)
            break
        }
    }
}
```

下面的示例代码也就能解释了为什么每次从`slice`删除一个元素后，倒数第1个元素和倒数第2个元素是相同的

```go
func main() {
	var rules = [][]int{
		{1, 2, 3},
		{4, 5, 6},
		{7, 8, 9},
	}
	prev := rules
	fmt.Println("origin\t", rules)
	i := 1
	rules = append(rules[:i], rules[i+1:]...)
	fmt.Println("removed\t", rules)
	fmt.Println("slice\t", prev)
}

// OUTPUT
// origin	 [[1 2 3] [4 5 6] [7 8 9]]
// removed	 [[1 2 3] [7 8 9]]
// slice	 [[1 2 3] [7 8 9] [7 8 9]]
```