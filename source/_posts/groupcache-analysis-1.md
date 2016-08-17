---
title: groupcache源码解析(1)
date: 2016-08-09 00:49:36
tags: GO
---
[groupcache](http://godoc.org/github.com/golang/groupcache)是memcached的作者Brad Fitzpatrick写的GO的版本，现用于dl.google.com，为google的静态资源文件服务，这个东西轻量，简洁，也容易理解，废话少说，边看源码边学习GO<!--more-->

### consistenthash.go
看文件名就能知道这是用来处理[一致性哈希](https://zh.wikipedia.org/wiki/一致哈希)的
```go
type Hash func(data []byte) uint32

type Map struct {
    hash     Hash
    replicas int
    keys     []int // Sorted
    hashMap  map[int]string
}
```
这里定义了两个结构，一个Hash方法，用于将key值来hash成一个uint32的方法，Map是用来保存Hash对应结果的结构，`Map.hashMap`里就是具体对应，`Map.keys`保存了哈希后的结果，并且是经过了排序，首位相接就形成一致性哈希的圆环，`Map.replicas`应该是使得各个对应相对分散一些，这个我们后面可以看到
```go
func New(replicas int, fn Hash) *Map {
    m := &Map{
        replicas: replicas,
        hash:     fn,
        hashMap:  make(map[int]string),
    }
    if m.hash == nil {
        m.hash = crc32.ChecksumIEEE
    }
    return m
}
```
初始化的方法使用`&Map`这样的方式，这主要得益于GO的内存处理，`return m`时会GO会把m从栈上移动到堆上，如果是C语言，这里会报错，`m.hash`为空时会默认使用`crc32.ChecksumIEEE`
```go
// Returns true if there are no items available.
func (m *Map) IsEmpty() bool {
    return len(m.keys) == 0
}
```
是否为空的判断直接返回了keys的长度，groupcache中直接使用了list，map这样的数据结构，简单直接
```go
// Adds some keys to the hash.
func (m *Map) Add(keys ...string) {
    for _, key := range keys {
        for i := 0; i < m.replicas; i++ {
            hash := int(m.hash([]byte(strconv.Itoa(i) + key)))
            m.keys = append(m.keys, hash)
            m.hashMap[hash] = key
        }
    }
    sort.Ints(m.keys)
}
```
这里是添加了多个哈希对应，这里的`keys ...string`参数可以使得输入时添加多个key，然后方法内部通过`range keys`来循环得到，比如`map.Add('abc')`或`map.Add('abc', 'bcd', 'cde')`，这里还添加了`m.replicas`的循环，可以分散各个keys的分部，通过内部方法`m.hash`来进行哈希，最后也用`sort.Ints`把所有key排序，保证一致性哈希的环
```go
// Gets the closest item in the hash to the provided key.
func (m *Map) Get(key string) string {
    if m.IsEmpty() {
        return ""
    }

    hash := int(m.hash([]byte(key)))

    // Binary search for appropriate replica.
    idx := sort.Search(len(m.keys), func(i int) bool { return m.keys[i] >= hash })

    // Means we have cycled back to the first replica.
    if idx == len(m.keys) {
        idx = 0
    }

    return m.hashMap[m.keys[idx]]
}
```
Get时先对key进行一次相同的哈希，之后通过`sort.Search`方法，查找`[0, len(m.keys))`这个区间的数，这样可以找到最小的满足`m.keys[i] >= hash`的值，如果没有找到那idx会被赋值为`len(m.keys)`，之后会把idx置为0，然后返回hashMap对应的值就可以，其实一致性哈希就是把要找的key和已设置的key进行一样的hash后进行结果聚合的结果
