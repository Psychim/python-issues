python版本：3.7.0
# 问题

运行https://github.com/ssepulveda/RTGraph的demo时出现错误
```
TypeError: can't pickle weakref objects
```
经过进一步确定，问题出在Process子类A的成员含有另一个Process子类B的时候，如果先调用B.start()再调用A.start()就会出现该错误。  
测试代码
```
from multiprocessing import Process,Event
import time
class TestProcess(Process):
    def __init__(self):
        Process.__init__(self)
    def run(self):
        pass
class TestProcessB(TestProcess):
    def __init__(self,p):
        Process.__init__(self)
        self._test_process=p
if __name__=='__main__':
    tp=TestProcess()
    tpb=TestProcessB(tp)
    tp.start()
    tpb.start()
```
# 解答
官方对这个问题给出了解答：https://bugs.python.org/issue34034

>It could be many different things.  The bottom line here, though, is that the Process class is not designed to be picklable (how would it work?), which is probably why you're seeing this.

这是由于Process类本身在设计上是不支持序列化的。因此这种用法是错误的，只是在python3.4-3.6版本恰好可以。应该避免Process类成员包含其它Process类。