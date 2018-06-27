# Python 中的循环导入依赖问题

今天在阅读《Flask Web 开发》一书的 P69 时发现了“避免循环导入依赖”一词，在 google 之后初步搞懂这个问题，记录成此文。  

### 问题描述
这个问题简单来说，就是两个包产生了互相依赖之后导致 ImportError，举例说明最清晰明了。  
[foo1.py](https://github.com/zhou-zheng/technote/blob/master/python/MutuallyImportProblem/foo1.py)
``` Python
from bar1 import bar_var
foo_var = 1
```
[bar1.py](https://github.com/zhou-zheng/technote/blob/master/python/MutuallyImportProblem/bar1.py)
``` Python
from foo1 import foo_var
bar_var = 2
```
此时如果我们在 Python Shell 中导入 foo1.py 或 bar1.py 都会出错。  
``` Shell
>>> import foo1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "foo1.py", line 1, in <module>
    from bar1 import bar_var
  File "bar1.py", line 1, in <module>
    from foo1 import foo_var
ImportError: cannot import name 'foo_var'
>>>
>>> import bar1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "bar1.py", line 1, in <module>
    from foo1 import foo_var
  File "foo1.py", line 1, in <module>
    from bar1 import bar_var
ImportError: cannot import name 'bar_var'
```
---
### 问题分析
为什么会产生这样的 ImportError 呢？我们以 import foo1 来分析下整个流程就明白了。  
- Python Shell 导入 foo1
- 创建 foo1 的全局符号字典
- foo1 被 Python 解释器解释执行
- foo1 导入 bar1
- 创建 bar1 的全局符号字典
- bar1 被 Python 解释器解释执行
- bar1 导入 foo1（此时已经加载过模块 foo1）
- bar1 导入 foo1 的 foo_var，即 bar1.foo_var = foo1.foo_var

在最后一步，因为 foo1 刚开始解释执行就转去了 bar1，所以 foo1 中的 foo_var 还没有被 Python 解释器解释执行出来，必然出现 cannot import name 'foo_var'！

---
### 问题解决
找到了问题的原因，对症下药就可以了。一个简便取巧的做法就是调整代码顺序来打破这个循环依赖的怪圈。  
[foo2.py](https://github.com/zhou-zheng/technote/blob/master/python/MutuallyImportProblem/foo2.py)
``` Python
foo_var = 1
from bar2 import bar_var
```
[bar2.py](https://github.com/zhou-zheng/technote/blob/master/python/MutuallyImportProblem/bar2.py)
``` Python
bar_var = 1
from foo2 import foo_var
```
此时如果我们在 Python Shell 中导入 foo2.py 或 bar2.py 都会成功。
``` Shell
>>> import foo2
>>> import bar2
>>> foo2.foo_var, foo2.bar_var
(1, 2)
>>> bar2.foo_var, bar2.bar_var
(1, 2)
```
为什么成功了呢？同样地我们仍然以 import foo2 为例再对流程分析一遍：
- Python Shell 导入 foo2
- 创建 foo2 的全局符号字典
- foo2 被 Python 解释器解释执行
- 创建 foo2.foo_var 
- foo2 导入 bar2
- 创建 bar2 的全局符号字典
- bar2 被 Python 解释器解释执行
- 创建 bar2.bar_var
- bar2 导入 foo2（此时已经加载过模块 foo2）
- bar2 导入 foo2 的 foo_var，即 bar2.foo_var = foo2.foo_var
- foo2 继续导入 bar2 的 bar_var，即 foo2.bar_var = bar2.bar_var

至此全部成功！

问题引申
如果我们在 foo1/bar1 的基础上只修改 foo1 或 bar1 的话，会发生什么呢？  
[foo3.py](https://github.com/zhou-zheng/technote/blob/master/python/MutuallyImportProblem/foo3.py)
``` Python
foo_var = 1
from bar3 import bar_var
```
[bar3.py](https://github.com/zhou-zheng/technote/blob/master/python/MutuallyImportProblem/bar3.py)
``` Python
from foo3 import foo_var
bar_var = 2
```
Python Shell 导入结果
``` Shell
>>> import bar3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "bar3.py", line 1, in <module>
    from foo3 import foo_var
  File "foo3.py", line 2, in <module>
    from bar3 import bar_var
ImportError: cannot import name 'bar_var'
>>> import foo3
>>> import bar3
```
[foo4.py](https://github.com/zhou-zheng/technote/blob/master/python/MutuallyImportProblem/foo4.py)
``` Python
from bar4 import bar_var
foo_var = 1
```
[bar4.py](https://github.com/zhou-zheng/technote/blob/master/python/MutuallyImportProblem/bar4.py)
``` Python
bar_var = 2
from foo4 import foo_var
```
Python Shell 导入结果
``` Shell
>>> import foo4
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "foo4.py", line 1, in <module>
    from bar4 import bar_var
  File "bar4.py", line 2, in <module>
    from foo4 import foo_var
ImportError: cannot import name 'foo_var'
>>> import bar4
>>> import foo4
```
有没有发现执行的结果随着 import 的顺序的不同而不同了呢？具体原因相信只要搞懂了前两种分析的同学都会明白，我就不多啰嗦了。

---
参考文献：  
1. [How can I have modules that mutually import each other?](https://docs.python.org/2/faq/programming.html#how-can-i-have-modules-that-mutually-import-each-other)