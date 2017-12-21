## 关键词
### *args **kw
> 1 *args是可变的positional arguments列表，**kwargs是可变的keyword arguments列表
> 2 *args必须位于**kwargs之前，因为positional arguments必须位于keyword arguments之前
> 3 用法
```  
for k, v in kwargs.items():
      print 'Optional argument %s (*kwargs): %s' % (k, v)
```
