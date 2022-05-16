title: How to Convert Python dict to object?
date: 2016-05-04 23:14:33
tags:
- Python
---

假如我们有如下的数据：
```python
d = {'a': 1, 'b': {'c': 2}, 'd': ["hi", {'foo': "bar"}]}
```

假如我需要访问`c`需要通过`d.get('b').get('c')`

如果能直接通过`d.b.c`去访问就好了

实现如下：
```python
class Struct:
  '''The recursive class for building and representing objects with.'''
  def __init__(self, obj):
    for k, v in obj.iteritems():
      if isinstance(v, dict):
        setattr(self, k, Struct(v))
      else:
        setattr(self, k, v)
  def __getitem__(self, val):
    return self.__dict__[val]
  def __repr__(self):
    return '{%s}' % str(', '.join('%s : %s' % (k, repr(v)) for
      (k, v) in self.__dict__.iteritems()))
```
使用：
```
s = Struct(d)
s.b.c
```