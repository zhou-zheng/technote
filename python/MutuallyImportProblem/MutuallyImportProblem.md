# Python 中的循环导入依赖问题

今天在阅读《Flask Web 开发》一书的 P69 时发现了“避免循环导入依赖”一词，在 google 之后初步搞懂这个问题，记录成此文。  
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
Python 3.6.5 |Anaconda, Inc.| (default, Mar 29 2018, 13:32:41) [MSC v.1900 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
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