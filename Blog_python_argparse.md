#理解python中argparse类
===
###导语：	
* python 3.2之前用optparse：[python doc optparse](https://docs.python.org/3.5/library/optparse.html)
* python 3.2以后用argparse: [python doc argparse](https://docs.python.org/3/library/argparse.html#module-argparse)
* 本文主要介绍argparse, 参考翻译: [argparse tutorial](https://docs.python.org/3/howto/argparse.html#id1)
* getopt也可以实现同样的功能，argparse是基于optparse写的，是optparse的升级版

###参数概览:
*我们先学习以下Unix一个命令行程序ls的设计, ls的基本用法是：

	$ ls
	cpython  devguide  prog.py  pypy  rm-unused-function.patch
	$ ls pypy
	ctypes_configure  demo  dotviewer  include  lib_pypy  lib-python ...
	$ ls -l
	total 20
	drwxr-xr-x 19 wena wena 4096 Feb 18 18:51 cpython
	drwxr-xr-x  4 wena wena 4096 Feb  8 12:04 devguide
	-rwxr-xr-x  1 wena wena  535 Feb 19 00:05 prog.py
	drwxr-xr-x 14 wena wena 4096 Feb  7 00:59 pypy
	-rw-r--r--  1 wena wena  741 Feb 18 01:01 rm-unused-function.patch
	$ ls --help	
	Usage: ls [OPTION]... [FILE]...
	List information about the FILEs (the current directory by default).
	Sort entries alphabetically if none of -cftuvSUX nor --sort is specified.
	...
	
从上面四个ls命令的实现我们可以学习到输入参数四个基本需求：

* ls命令不加任何参数可以直接运行，运行结果是列出当前目录下文件/文件夹
* 如果我们需要得到更详细的制定信息，比如想得到制定目录下的文件，就提供目录名 ls pypy，还有基于位置的参数，比如我们用cp [src] [des],第一个参数就是源文件地址，第二个参数就是目的地址
* 我们可以改变程序的默认输出，这里我们用-l，ls就会输出文件的详细信息
* ls --help,提供帮助文档，让从没用过的用户可以按照帮助来正确的使用

###基本用法:
新建一个prog.py, 看一下argparse的最基本用法:

	import argparse
	parser = argparse.ArgumentParser()
	parser.parse_args()
	
运行这个脚本:

	$ python3 prog.py
	$ python3 prog.py --help
	usage: prog.py [-h]

	optional arguments:
	-h, --help  show this help message and exit
	$ python3 prog.py --verbose
	usage: prog.py [-h]
	prog.py: error: unrecognized arguments: --verbose
	$ python3 prog.py foo
	usage: prog.py [-h]
	prog.py: error: unrecognized arguments: foo
	
让我们看看都发生了什么:

*  不带参数直接运行程序没有任何输出，没什么用
*  第二个命令显示了argparse的功能--我们什么都没有写，却得到了help功能
*  --help还可以缩写为-h

###位置参数
例程 proxy.py：

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("echo")
	args = parser.parse_args()
	print(args.echo)
	
run一下试试:

	$ python3 prog.py
	usage: prog.py [-h] echo
	prog.py: error: the following arguments are required: echo
	$ python3 prog.py --help
	usage: prog.py [-h] echo
	
	positional arguments:
	  echo
	
	optional arguments:
	  -h, --help  show this help message and exit
	$ python3 prog.py foo
	foo
	
让我们看看都发生了什么:

*  我们添加了add_argument()方法，添加的命令行选项是echo
*  现在调用程序需要指定echo选项，否则提示error: the following...
*  parse_args()获取选项并放入args中
*  argparse自动防止选项，parse_arges以后直接用args.echo就可以调用选项

但是这还不够，现在的help只告诉我们arguments是echo，但我们还是不知道他是怎么用的,argparse用 help=解决了这个问题，让我们再修改一下prog.py：

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("echo", help="echo the string you use here")
	args = parser.parse_args()
	print(args.echo)
	
run一下试试:

	$ python3 prog.py -h
	usage: prog.py [-h] echo
	
	positional arguments:
 	 echo        echo the string you use here
	
	optional arguments:
  	-h, --help  show this help message and exit
  	
现在help友好多了，让我们试一个有点儿用的例子吧，乘方:

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("square", help="display a square of a given number")
	args = parser.parse_args()
	print(args.square**2)
	
run一下的结果是:

	$ python3 prog.py 4
	Traceback (most recent call last):
	  File "prog.py", line 5, in <module>
	    print(args.square**2)
	TypeError: unsupported operand type(s) for ** or pow(): 'str' and 'int'
	
失败了。。。因为argparse默认认为输入值是string类型，我们需要告诉他这是int类型，加一个type=int就行了:

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("square", help="display a square of a given number",
	                    type=int)
	args = parser.parse_args()
	print(args.square**2)
	
现在再run一下:

	$ python3 prog.py 4
	16
	$ python3 prog.py four
	usage: prog.py [-h] square
	prog.py: error: argument square: invalid int value: 'four'
	
很好了，现在我们还有了输入检查。 位置参数就介绍到这里，下面介绍一下选项参数。

###选项参数
先看看怎么添加选项参数,还是用prog.py:

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("--verbosity", help="increase output verbosity")
	args = parser.parse_args()
	if args.verbosity:
    	print("verbosity turned on")
    
run一下看输出:

	$ python3 prog.py --verbosity 1
	verbosity turned on
	$ python3 prog.py
	$ python3 prog.py --help
	usage: prog.py [-h] [--verbosity VERBOSITY]
	
	optional arguments:
 	 -h, --help            show this help message and exit
 	 --verbosity VERBOSITY
 	                       increase output verbosity
	$ python3 prog.py --verbosity
	usage: prog.py [-h] [--verbosity VERBOSITY]
	prog.py: error: argument --verbosity: expected one argument
	
让我们看看发生了什么:

* 期望的结果是，如果输入参数有--verbosity就输出一些信息，如果没有就不输出
* 这个参数是可选的，也就是说不指定这个参数不会有任何错误出现
* 使用--verbosity的时需要指定值

现在的问题是，--verbosity 只有两个值TRUE和FALSE是有用的，现在程序接收所有的整数，我们来修改一下:

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("--verbose", help="increase output verbosity",
	                    action="store_true")
	args = parser.parse_args()
	if args.verbose:
 	   print("verbosity turned on")
 	   
再看看输出:

	$ python3 prog.py --verbose
	verbosity turned on
	$ python3 prog.py --verbose 1
	usage: prog.py [-h] [--verbose]
	prog.py: error: unrecognized arguments: 1
	$ python3 prog.py --help
	usage: prog.py [-h] [--verbose]
	
	optional arguments:
	  -h, --help  show this help message and exit
	  --verbose   increase output verbosity
	  
这次我们加了一个新的关键字:action，给它一个值store_true,所有当我们只给了--verbose，没有给值的时候，默认berbos是TRUE，if 中的print打印成功
	  

### 快捷选项
Unix 的命令行选项几乎都有快捷方式，比如--verbose可以直接用-v来代替，用argparse实现非常简单:

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("-v", "--verbose", help="increase output verbosity",
	                    action="store_true")
	args = parser.parse_args()
	if args.verbose:
	    print("verbosity turned on")
	    
run一下试试:

	$ python3 prog.py -v
	verbosity turned on
	$ python3 prog.py --help
	usage: prog.py [-h] [-v]
	
	optional arguments:
 	 -h, --help     show this help message and exit
 	 -v, --verbose  increase output verbosity
 	 
再add_argument里加一个-v就可以了，help也有了相应的变化

### 整合位置参数和选项参数
让我们把prog.py写得再充实一些:

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("square", type=int,
	                    help="display a square of a given number")
	parser.add_argument("-v", "--verbose", action="store_true",
	                    help="increase output verbosity")
	args = parser.parse_args()
	answer = args.square**2
	if args.verbose:
 	    print("the square of {} equals {}".format(args.square, answer))
	else:
	    print(answer)
	    
现在的输出更友好了:

	$ python3 prog.py
	usage: prog.py [-h] [-v] square
	prog.py: error: the following arguments are required: square
	$ python3 prog.py 4
	16
	$ python3 prog.py 4 --verbose
	the square of 4 equals 16
	$ python3 prog.py --verbose 4
	the square of 4 equals 16
	
* 我们把位置参数加回来了，注意-v 和 4的顺序可以随意放置，不影响结果

现在我们把verbose 再写详细一些:

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("square", type=int,
	                    help="display a square of a given number")
	parser.add_argument("-v", "--verbosity", type=int,
	                    help="increase output verbosity")
	args = parser.parse_args()
	answer = args.square**2
	if args.verbosity == 2:
	    print("the square of {} equals {}".format(args.square, answer))
	elif args.verbosity == 1:
	    print("{}^2 == {}".format(args.square, answer))
	else:
	    print(answer)
	    
看一下输出:

	$ python3 prog.py 4
	16
	$ python3 prog.py 4 -v
	usage: prog.py [-h] [-v VERBOSITY] square
	prog.py: error: argument -v/--verbosity: expected one argument
	$ python3 prog.py 4 -v 1
	4^2 == 16
	$ python3 prog.py 4 -v 2
	the square of 4 equals 16
	$ python3 prog.py 4 -v 3
	16
	
除了最后一行，其他都很不错，-v 3是我们程序的bug，让我们来处理一下,不接受0，1，2以外的值，add_argument里只需要加个choice就可以了:

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("square", type=int,
	                    help="display a square of a given number")
	parser.add_argument("-v", "--verbosity", type=int, choices=[0, 1, 2],
	                    help="increase output verbosity")
	args = parser.parse_args()
	answer = args.square**2
	if args.verbosity == 2:
	    print("the square of {} equals {}".format(args.square, answer))
	elif args.verbosity == 1:
	    print("{}^2 == {}".format(args.square, answer))
	else:
	    print(answer)
	    
再run一下:

	$ python3 prog.py 4 -v 3
	usage: prog.py [-h] [-v {0,1,2}] square
	prog.py: error: argument -v/--verbosity: invalid choice: 3 (choose from 0, 1, 2)
	$ python3 prog.py 4 -h
	usage: prog.py [-h] [-v {0,1,2}] square
	
	positional arguments:
	  square                display a square of a given number
	
	optional arguments:
	  -h, --help            show this help message and exit
	  -v {0,1,2}, --verbosity {0,1,2}
	                        increase output verbosity
	                        
* 错误信息和help信息都自动变化了

现在再改一下我们的程序,让它跟cpython有类似的效果，加一个action=“count”:

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("square", type=int,
	                    help="display the square of a given number")
	parser.add_argument("-v", "--verbosity", action="count",
	                    help="increase output verbosity")
	args = parser.parse_args()
	answer = args.square**2
	if args.verbosity == 2:
	    print("the square of {} equals {}".format(args.square, answer))
	elif args.verbosity == 1:
	    print("{}^2 == {}".format(args.square, answer))
	else:
	    print(answer) 
	    
count用来统计option出现的个数,args.verbosity获取的就是出现个数：

	$ python3 prog.py 4
	16
	$ python3 prog.py 4 -v
	4^2 == 16
	$ python3 prog.py 4 -vv
	the square of 4 equals 16
	$ python3 prog.py 4 --verbosity --verbosity
	the square of 4 equals 16
	$ python3 prog.py 4 -v 1
	usage: prog.py [-h] [-v] square
	prog.py: error: unrecognized arguments: 1
	$ python3 prog.py 4 -h
	usage: prog.py [-h] [-v] square
	
	positional arguments:
	  square           display a square of a given number
	
	optional arguments:
	  -h, --help       show this help message and exit
	  -v, --verbosity  increase output verbosity
	$ python3 prog.py 4 -vvv
	16
	
能看出来， count是和store_true类似的action，现在我们的prog.py的帮助还不够只能，最后一行-vvv的输出是一个bug，我们先fix了它,很简单，==替换成>=就好了：

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("square", type=int,
	                    help="display a square of a given number")
	parser.add_argument("-v", "--verbosity", action="count",
	                   help="increase output verbosity")
	args = parser.parse_args()
	answer = args.square**2
	
	# bugfix: replace == with >=
	if args.verbosity >= 2:
	    print("the square of {} equals {}".format(args.square, answer))
	elif args.verbosity >= 1:
	    print("{}^2 == {}".format(args.square, answer))
	else:
	    print(answer)
	    
现在的输出：

	$ python3 prog.py 4 -vvv
	the square of 4 equals 16
	$ python3 prog.py 4 -vvvv
	the square of 4 equals 16
	$ python3 prog.py 4
	Traceback (most recent call last):
	  File "prog.py", line 11, in <module>
 	   if args.verbosity >= 2:
	TypeError: unorderable types: NoneType() >= int()
	
第三个命令的输出仍然有问题，我们来fix它，加一个default=0：

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("square", type=int,
	                    help="display a square of a given number")
	parser.add_argument("-v", "--verbosity", action="count", default=0,
	                    help="increase output verbosity")
	args = parser.parse_args()
	answer = args.square**2
	if args.verbosity >= 2:
	    print("the square of {} equals {}".format(args.square, answer))
	elif args.verbosity >= 1:
	    print("{}^2 == {}".format(args.square, answer))
	else:
	    print(answer)
	    
到这里我们已经把argparse的基本用法介绍完了，在结束这篇文章之前，我们把prog.py稍微增强一下：

###argparse进阶

把prog.py改成阶乘，不仅仅是平方：

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("x", type=int, help="the base")
	parser.add_argument("y", type=int, help="the exponent")
	parser.add_argument("-v", "--verbosity", action="count", default=0)
	args = parser.parse_args()
	answer = args.x**args.y
	if args.verbosity >= 2:
	    print("{} to the power {} equals {}".format(args.x, args.y, answer))
	elif args.verbosity >= 1:
	    print("{}^{} == {}".format(args.x, args.y, answer))
	else:
	    print(answer)
	    
试试：

	$ python3 prog.py
	usage: prog.py [-h] [-v] x y
	prog.py: error: the following arguments are required: x, y
	$ python3 prog.py -h
	usage: prog.py [-h] [-v] x y
	
	positional arguments:
	  x                the base
	  y                the exponent
	
	optional arguments:
	  -h, --help       show this help message and exit
	  -v, --verbosity
	$ python3 prog.py 4 2 -v
	4^2 == 16
	
前面我们一直用verbosity level修改显示字符，下面的例子会显示更多的信息：

	import argparse
	parser = argparse.ArgumentParser()
	parser.add_argument("x", type=int, help="the base")
	parser.add_argument("y", type=int, help="the exponent")
	parser.add_argument("-v", "--verbosity", action="count", default=0)
	args = parser.parse_args()
	answer = args.x**args.y
	if args.verbosity >= 2:
	    print("Running '{}'".format(__file__))
	if args.verbosity >= 1:
	    print("{}^{} == ".format(args.x, args.y), end="")
	print(answer)	
	
输出:

	$ python3 prog.py 4 2
	16
	$ python3 prog.py 4 2 -v
	4^2 == 16
	$ python3 prog.py 4 2 -vv
	Running 'prog.py'
	4^2 == 16
	
###冲突选项

到目前为止我们尝试了argparse.ArgumentParser()里的两种方法，现在我们试试第三种，add__mutually_exclusive_group(). 他允许我们添加相互冲突的选项，我们把程序改一下，加一个新的选项--quiet,和--verbose相反的选项：

	import argparse
	
	parser = argparse.ArgumentParser()
	group = parser.add_mutually_exclusive_group()
	group.add_argument("-v", "--verbose", action="store_true")
	group.add_argument("-q", "--quiet", action="store_true")
	parser.add_argument("x", type=int, help="the base")
	parser.add_argument("y", type=int, help="the exponent")
	args = parser.parse_args()
	answer = args.x**args.y
	
	if args.quiet:
	    print(answer)
	elif args.verbose:
	    print("{} to the power {} equals {}".format(args.x, args.y, answer))
	else:
	    print("{}^{} == {}".format(args.x, args.y, answer))
	    
现在代码更简单，我们只是为了掩饰一下冲突的选项，run一下试试：

	$ python3 prog.py 4 2
	4^2 == 16
	$ python3 prog.py 4 2 -q
	16
	$ python3 prog.py 4 2 -v
	4 to the power 2 equals 16
	$ python3 prog.py 4 2 -vq
	usage: prog.py [-h] [-v | -q] x y
	prog.py: error: argument -q/--quiet: not allowed with argument -v/--verbose
	$ python3 prog.py 4 2 -v --quiet
	usage: prog.py [-h] [-v | -q] x y
	prog.py: error: argument -q/--quiet: not allowed with argument -v/--verbose
              
Argparse 已经帮我们处理了，不管是-v 还是--quite，很强大是不是？

最后，我们给程序添加一个介绍：

	import argparse
	
	parser = argparse.ArgumentParser(description="calculate X to the power of Y")
	group = parser.add_mutually_exclusive_group()
	group.add_argument("-v", "--verbose", action="store_true")
	group.add_argument("-q", "--quiet", action="store_true")
	parser.add_argument("x", type=int, help="the base")
	parser.add_argument("y", type=int, help="the exponent")
	args = parser.parse_args()
	answer = args.x**args.y
	
	if args.quiet:
	    print(answer)
	elif args.verbose:
	    print("{} to the power {} equals {}".format(args.x, args.y, answer))
	else:
	    print("{}^{} == {}".format(args.x, args.y, answer))

试试：
	
	$ python3 prog.py --help
	usage: prog.py [-h] [-v | -q] x y
	
	calculate X to the power of Y
	
	positional arguments:
	  x              the base
	  y              the exponent
	
	optional arguments:
	  -h, --help     show this help message and exit
	  -v, --verbose
	  -q, --quiet
	  
能看出来argparse默认给冲突选项用了[-v | -q],提示两个选项只能用其中之一。

###总结

[Argparse](https://docs.python.org/3/library/argparse.html#module-argparse) 使用很方便很强大，文档和例子也很全.照着这个文档做完，你应该已经可以熟练使用它了。







