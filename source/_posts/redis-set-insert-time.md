title: Redis set中sadd插入时间测试
date: 2015-11-25 15:38:28
tags:
- Redis
- unittest
- Python
---

今天老郑问我“一个redis的set里面，最多能放多少数据，而不会有显著的性能下降？”，由于Redis中读取
的时间复杂度为O(1)，增加的复杂度为O(n)，这个问题还是有测试的价值的。

所以我直接写了下面这个测试用例来测试这个问题：

```python
import unittest

import redis
import time


class RedisTest(unittest.TestCase):
    def setUp(self):
        self.redis_client = redis.StrictRedis()

    def test_add_to_set(self):
        for i in range(1000000):
            start = time.time()
            self.redis_client.sadd('test_set', i)
            end = time.time()

```

```
> python -m unittest tests/test_redis.py

... 省略了好多好多行
index: 999994, time: 0.00010895729064941406
index: 999995, time: 0.00014710426330566406
index: 999996, time: 0.00015497207641601562
index: 999997, time: 0.00015592575073242188
index: 999998, time: 0.00017213821411132812
index: 999999, time: 0.00010395050048828125
.
----------------------------------------------------------------------
Ran 1 test in 191.684s

OK
```

100W次插入，用时191.684s，平均每秒插入5216条数据，这个速度还是非常非常快的，所以放心用吧，
不用担心速度问题啦。
