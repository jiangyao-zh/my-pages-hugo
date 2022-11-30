---
title: 'Go增加Redis缓存机制'
date: '2022-11-08'
tags: ["Go","Redis","Singleflight"]
categories: ["WEB"]
draft: false
---
## Go增加Redis缓存机制方案

### 增加缓存机制

- 缓存基于redis，以数据集为单位组织存储。
- 根据查询条件序列化生成单独的key，查询数据作为内容，组成缓存的最小单元。
- 数据查询时先进行redis缓存查询，如果命中，直接取缓存内容返回，未命中则继续执行后续程序，从数据库中查询内容，查询成功后存入缓存，以待下次使用。

#### 缓存模块提供方法

```Go
 //存储缓存方法
 func (c *Cache) HSet(ctx context.Context, datasetId int, key string, value interface{})
 
 //获取缓存方法
 func (c *Cache) HGet(ctx context.Context, datasetId int, key string) string
 
 //删除缓存，以数据集为单位
 func (c *Cache) Del(ctx context.Context, datasetId int)
 ```

#### 缓存击穿与并发解决方案

- 为应对高并发和缓存击穿风险，采用singleflight技术。singleflght使用协程技术，可以保证相同的查询在同一时间只有一个进程可以访问查询程序，其余的进程则会在查询完成后共享查询结果。

- singleflght可以在缓存查询前调用，以减缓高并发对缓存造成的压力，同样也可以减少缓存击穿时对数据库造成的冲击。

- singleflight代码：

```Go

import "sync"

type call struct {
    wg  sync.WaitGroup
    val interface{}
    err error
}

type Group struct {
    mu sync.Mutex      
    m  map[string]*call
}

func (g *Group) Do(key string, fn func() (interface{}, error)) (interface{}, error) {
    g.mu.Lock()
    if g.m == nil {
        g.m = make(map[string]*call)
    }
    if c, ok := g.m[key]; ok {
        g.mu.Unlock()
        c.wg.Wait()
        return c.val, c.err
    }
    c := new(call)
    c.wg.Add(1)
    g.m[key] = c
    g.mu.Unlock()

    c.val, c.err = fn()
    c.wg.Done()

    g.mu.Lock()
    delete(g.m, key)
    g.mu.Unlock()

    return c.val, c.err
}
```

- 使用示例：

```Go
import (
    "testing"
)

func TestDo(t *testing.T) {
    var g Group
    v, err := g.Do("key", func() (interface{}, error) {
        return "bar", nil
    })

    if v != "bar" || err != nil {
        t.Errorf("Do v = %v, error = %v", v, err)
    }
}

```
