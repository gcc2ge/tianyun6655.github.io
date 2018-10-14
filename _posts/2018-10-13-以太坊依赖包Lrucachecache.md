---
layout: post
title: 以太坊依赖包源码分析-Lru
subtitle: 理解Lru的Go版本
date: 2018-10-13
author: Tianyun Chen (OFBank)
header-img: img/lru_backgroud.png
catalog: true
tags:
   - 依赖包
   - Block Chain
   - 区块链
---

### 以太坊依赖库源码分析-LruCache


### 前言

最近突然想整理下，以太坊依赖包的源码。没错，对于学Go的小伙伴，Geth 和Docker 是明星级的源码阅读项目。 对于2个项目所用的依赖包，个人觉得 很有必要拿出来说说，因为他们支持了整套系统的运行。同时也能增加下比较基础的代码能力和类库的推荐选择。

----------------

### LRU

Lru 具体是什么呢， 英语就是（Least Recently Used）。最近最少的使用。如果一个数据在最近一段时间没有被访问到，那么在将来它被访问的可能性也很小。也就是说，当限定的空间已存满数据时，应当把最久没有被访问到的数据淘汰。Lru 大多数情况下会被在用缓存中。据我们所知的Redis 里面就有一个Lru的选项。不要再说面试的时候碰到Lru 不会写了，其实简单！！！

#### Eth中的依赖包和使用
Eth 中也确实用到了 Lru， 比如说区块的Totaldifficult 的存储里面就用到了Lru缓存。具体的依赖包为：  https://github.com/golang/groupcache/tree/master/lru。我特意去github上看了下。start 数有6757了，对于go的项目来说 确实挺高了。很有学习的价值


#### 包结构
![lru_package](https://github.com/tianyun6655/tianyun6655.github.io/blob/master/img/lru_package.png?raw=true)

包结构的还是比较简单的，包内又用了simplelru，那么我们先从simple 开始吧。

#### SimpleLru 源码分析
![simple_lru_package](https://github.com/tianyun6655/tianyun6655.github.io/blob/master/img/simple_lru_package.jpg)

看下具体的几个变量：
- items: 存放cache 内容的，大多数的cache 都是Key，value形式的，自然要用到Map 这种数据结构了
- size： 缓存的大小，如果超过这个大小自然就要进行策略删除在cache中的item了
- evictList：之前说过的Lru 是删除一直没用用的item，那么这个simplelru 用一个list 维护这样的关系（最前面表示最新，最后面表示最老）
- onEvict： 当发生删除策略时候的callback


具体的几个方法：
- Purge： 清空
- Add： 添加
- Get： 获取
- Contains： 是否包含
- Peek： 第一个元素
- Remove： 移除
- RemoveOldest： 移除最最老的元素，这里最最老的就是一直没用的（相当于手动调用策略）
- GetOldest: 获取最老的
- Keys： 获取所有Key

具体方法讲解：
```
 //构造法咯，不多解释!!注意Simple是线程非！！！非！！安全的，全部操作中没有带锁
func NewLRU(size int, onEvict EvictCallback) (*LRU, error) {
	//判断大小是否符合要求
    if size <= 0 {
		return nil, errors.New("Must provide a positive size")
	}
	c := &LRU{
		size:      size,
		evictList: list.New(),
		items:     make(map[interface{}]*list.Element),
		onEvict:   onEvict,
	}
	return c, nil
}

```

```
// Purge is used to completely clear the cache
//清空所有的缓存，重来来咯
func (c *LRU) Purge() {

	for k, v := range c.items {
	   //清楚的时候判断是不是有callback 方法，有的话，执行一下
		if c.onEvict != nil {
			c.onEvict(k, v.Value.(*entry).value)
		}
		delete(c.items, k)
	}
	c.evictList.Init()
}
```

```
// Add adds a value to the cache.  Returns true if an eviction occurred.
//添加元素
func (c *LRU) Add(key, value interface{}) bool {
	// Check for existing item
	//判断元素是不是存在了，如果存在说明，这个元素已经用过一次了，所以需要把它移动到List的最前面去（）
	//然后更新缓存内容（也就是说，add=add+update）
	if ent, ok := c.items[key]; ok {
		c.evictList.MoveToFront(ent)
		ent.Value.(*entry).value = value
		return false
	}

	// 添加元素 不多解释
	ent := &entry{key, value}
	entry := c.evictList.PushFront(ent)
	c.items[key] = entry
	//好了这里需要判断是不是超长了
	evict := c.evictList.Len() > c.size
	// Verify size not exceeded
	if evict {
		//如果超了那么就要删除了
		c.removeOldest()
	}
	return evict
}

```

```
//这里不多解释了，就这样，这个看不懂，那你要去补补Go了
// Get looks up a key's value from the cache.
//获取元素
func (c *LRU) Get(key interface{}) (value interface{}, ok bool) {
	//如果存在需要把其移动到最最前面哈
	if ent, ok := c.items[key]; ok {
		c.evictList.MoveToFront(ent)
		return ent.Value.(*entry).value, true
	}
	return
}

// Check if a key is in the cache, without updating the recent-ness
// or deleting it for being stale.
func (c *LRU) Contains(key interface{}) (ok bool) {
	_, ok = c.items[key]
	return ok
}

```
#### 其实很simple Lru 很简单就是一个map 和List的使用，主要是这个肯定不能用于线上环境，因为线程非安全绝对致命。

```
//更上一层其实就是对Simple lru的互斥锁的封装，每个方法加了互斥锁
func (c *Cache) Purge() {
	c.lock.Lock()
	c.lru.Purge()
	c.lock.Unlock()
}
```


