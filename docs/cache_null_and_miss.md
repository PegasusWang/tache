# Cache 空值与缓存穿透

当查询一个不存在的数据时，会得到一个空值。如果缓存层不缓存这个空值，那么查询就会穿透到
数据库，给数据库带来查询压力。当一个爬虫大量抓取不存在的数据时，这个问题尤为明显。
缓存穿透使得缓存层失去了保护后端存储的能力，如果后端存储层关联业务比较多的情况下，甚至可能引起雪崩。

但是缓存空值也会带来新问题：

- 其一， 浪费缓存存储空间。比起不缓存空值，会缓存更多的数据。但我们认为以空间换时间是值得的，对于
这类数据我们也会缓存更短的时间(在5 分钟，和正常过期时间的十分之一两者间去最小值)

- 其二， 业务层新增数据可能不能被正常取到。当缓存了一条不存在的数据后，在缓存过期前这条数据可能刚好
被创建了出来，这时就造成了一定时间内的数据不一致。业务层需要注意到这点，如果在新建数据后就立刻访问，
你需要 refresh 一下来刷新缓存。

对于访问量比较小的业务，可以直接禁掉缓存空值。`Tache.cached` 方法中提供一个 `should_cache_fn` 的参数，
接受一个 function, 传入被装饰函数执行的结果，返回一个布尔值来决定是否可以缓存。

```
    @cache.cached(should_cache_fn=lambda value: value is not None)
    def incr(by):
        ...
```
这个例子中的 `should_cache_fn` 接受的函数就表示，除了 `None` 外都缓存。

也可以很方便的禁用缓存，以便在开发阶段排除是否是缓存引起的 bug

```
    @cache.cached(should_cache_fn=lambda _: False)
    def incr(by):
        ...
```

`Tache.batch` 不支持此用法。