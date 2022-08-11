---
title: Python单元测试
date: 2022-08-10 19:40:49
categories: 
    - 测试相关
tags: 
    - Python单元测试
---

## 1. PyTest安装:
```bash
>>> pip install pytest

# 常用插件安装
>>> pip install pytest-mock  # mocker插件
```

## 2. 基础用法
定义一个函数如下:

```python
def get_sum(a, b):
	print "calling get_sum function"
	return a + b
```

为了验证其功能，我们可以编写单测用例如下:
```python
import pytest

class TestTmpFunction(object):
	def test_sum(self):
		result = get_sum(1, 2)
		print result
		assert result == 3
```

运行用例:
```bash
>>> python -m pytest -v test_tmp.py -s
```

### 2.1 命令行参数
可以通过`pytest -help` 查看支持的参数。以下是一些常用的参数:
* `-v`: 输出更详细的用例执行信息, 不使用 -v 参数，运行时不会显示运行的具体测试用例名称；
* `-s`: 显示print内容 在运行测试用例时，为了调试或打印一些内容，我们会在代码中加一些print内容，但是这些内容默认不会显示出来。如果带上-s，就可以显示了。
* `-x`: 出现一条测试用例失败就退出测试。 
* `-m`: 用表达式指定多个标记名。 pytest 提供了一个装饰器 @pytest.mark.xxx，用于标记测试并分组，以便你快速选中并运行，各个分组直接用 and、or 来分割。



### 2.2 选择执行的测试用例(静态)
按文件夹执行
```shell
# 执行指定文件夹及子文件夹下的所有测试用例
pytest ../tests
```

按文件执行
```shell
# 运行test_tmp.py下的所有的测试用例
pytest test_tmp.py
```

按测试类执行
```shell
# pytest 文件名.py::测试类
pytest test_tmp.py::TestTmp
```

按测试方法执行
```shell
# pytest 文件名.py::测试类::测试方法
pytest test_tmp.py::TestTmpFunction::test_sum
```

### 选择执行的测试用例(动态)
如要使用动态指定测试用例的方式，首先需要给测试用例打标签（mark），比如在 `class`、`method` 上加上如下装饰器：
```python
@pytest.mark.dev_test
```
在运行时，可以根据标签来动态的选择哪些用例需要执行

```shell
# 同时选中带有这两个标签的所有测试用例运行
pytest -m "mark1 and mark2"

# 选中带有mark1的测试用例，不运行mark2的测试用例
pytest -m "mark1 and not mark2"

# 选中带有mark1或 mark2标签的所有测试用例
pytest -m "mark1 or mark2"
```

除此之外还提供了一种通过模糊匹配的方式选择测试用例的方式:
```shell
# -k 参数是按照文件名、类名、方法名、标签名来模糊匹配的
pytest -k xxxPattern
```

## 3. mock使用
pytest自带的unittest框架中默认集成了mock库，PyTest的mock支持是通过插件实现的。相对来讲PyTest使用起来更简单(PyTest的mocker是对原生mock的一个兼容，原生mock支持的功能mocker基本都可以支持)

### 3.1 基础用法
```python
def get_sum(a, b):
	print "calling get_sum function"
	return a + b


class TestTmpFunction(object):
	def test_sum_with_mock(self, mocker):
		mocker.patch('test_tmp.get_sum', return_value=3)
		result = get_sum(1, 2)
		print result
		assert result == 3
```
运行后可以发现，原本`get_sum`的print内容并没有被打印出来，我们通过`mocker.patch`方法屏蔽掉了原函数，转而直接返回我们指定的返回结果

### 3.2 其他用法
mocker.patch的函数定义
```python
unittest.mock.patch(target, new=DEFAULT, spec=None, create=False, spec_set=None, autospec=None, new_callable=None, **kwargs)
```
常用参数含义：
* `target`: 模拟对象的路径，参数必须是一个str,格式为'package.module.ClassName'，注意这里的格式一定要写对。如果对象和mock函数在同一个文件中，路径要加文件名
* `return_value`: 模拟函数返回的结果
* `side_effect`: 调用mock时的返回值，可以是函数，异常类，可迭代对象。当设置了该方法时，如果该方法返回值是DEFAULT，那么返回return_value的值，如果不是，则返回该方法的值。 return_value 和 side_effect 同时存在，side_effect会返回。(如果 side_effect 是异常类或实例时，调用模拟程序时将引发异常。如果 side_effect 是可迭代对象，则每次调用 mock 都将返回可迭代对象的下一个值。如果设置为函数时其具体表现会替换被mock函数）


## 4. MagicMock

## 5. 数据驱动