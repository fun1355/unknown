https://stackoverflow.com/questions/41452314/static-analysis-tools-for-llvm-ir
```
|----------|-----------|--------------|--------------|---------|-----------|---------|------|
| Tool     | LAV       | CBMC         | ESBMC        | KLEE    | LLBMC     | CALYSTO | PEX  |
|----------|-----------|--------------|--------------|---------|-----------|---------|------|
| Frontend | LLVM      | goto-cc(gcc) | goto-cc(gcc) | LLVM    | LLVM      | LLVM    | .NET |
|----------|-----------|--------------|--------------|---------|-----------|---------|------|
| Theories | -         | PL           | -            | -       | -         | -       | -    |
|          | LA        | -            | LA           | -       | -         | -       | LA   |
|          | BV        | -            | BV           | BV      | BV        | BV      | BV   |
|          | EUF       | -            | EUF          | -       | -         | -       | EUF  |
|          | ARR.      | -            | ARR.         | ARR.    | ARR.      | -       | ARR. |
|----------|-----------|--------------|--------------|---------|-----------|---------|------|
| Solvers  | MathSAT   | MathSAT      | MathSAT      | -       | -         | -       | -    |
|          | Boolector | Boolector    | Boolector    | -       | Boolector | -       | -    |
|          | Z3        | Z3           | Z3           | Z3      | Z3        | -       | Z3   |
|          | Yices     | Yices        | Yices        | -       | -         | -       | -    |
|          | -         | -            | -            | STP     | STP       | -       | -    |
|          | -         | MiniSAT2     | CVC          | MetaSAT | -         | Spear   | -    |
|----------|-----------|--------------|--------------|---------|-----------|---------|------|
Propositional Logic (PL), linear arithmetic (LA),  bit-vector arithmetic (BVA), 
theory of uninterpreted functions (EUF), theory of arrays (ARRAYS)。
goto-cc: a compiler from C and C++ into GOTO-programs
```
# KLEE
这个编译起来非常复杂。我上次在mac上编译不成功。我参考的是这个 https://github.com/tum-i22/klee-install, 只能用Ubuntu才能编译成功。好像依赖库uclibc缺乏头文件，提示我可以继续用KLEE，但是会有限制，应该是最大内存的限制。但是SAT都是开很大内存，几百万个参数一起算的，后面好像编译不成功是LLVM的版本太新了，我的用8，他配套的是RELEASE_342.有个数据结构变了。浪费我2天时候。我把主要的错误记录在这里了，https://github.com/sancao2/klee-install.
后来用docker，半个小时就搭建好了。
通过编译这个KLEE，了解了他为什么依赖这些库：
## klee-uclibc
uclibc is for embedded systems，MMU-less systems. 因为klee内存密集型，所以自己改了uclibc, https://www.uclibc.org/FAQ.html
## minisat

## STP
SMT solver aimed at bitvectors and arrays. 这个比较老了，最迟参加了2014 SMTCOMP。https://github.com/stp/stp
## z3
SMT solver from Microsoft Research, http://theory.stanford.edu/~nikolaj/programmingz3.html, 这个文档第一个架构图可以参考，整个文档都是很好的学习资料。

KLEE的架构还是很经典的。后面的工程CLOVER不过是他的C++的“补丁”。

# LAV
http://argo.matf.bg.ac.rs/lav/README.txt
lav 很具有前途，他可以用yices，boolector，mathsat，z3（其他SMT solver因为发行版权问题未采用。）作为后端的Solver，可以在configure的时候指定，如：
$./configure --with-llvm=<path/to/llvm> --enable-<solver1> [--enable-<solver2>] [--enable-<solver3>] [--enable-<solver4>]
他的代码结构就很值得研究了。

## yices
- Yices SMT solver (http://yices.csl.sri.com) 
这个sri.com的网站，我这两天上不去了。只能访问github，https://github.com/SRI-CSL/yices2 。我挺喜欢这个工程，自带lisp-like的语言的。他的menual很详细，github还持续更新。他的输入，支持SMT-LIB 1.2和SMT-LIB 2.5。代码目录有example，有命令行yices，yices-smt2，yices-smt等命令行工具。
我发现很多SMT之间都能互相调用，他们都有相似的"接口"，比如SMT-LIB、DIMACS等。
## boolector 
- Boolector SMT solver (http://fmv.jku.at/boolector/)
(BVA and ARRAYS)
## mathsat   
- MathSAT SMT solver (http://mathsat.itc.it/)
(LA, BVA, EUF)
## z3
- Z3 SMT solver (https://github.com/Z3Prover/z3/releases)
(LA, BVA, EUF, ARRAYS)

# KLOVER
http://www.cs.utah.edu/~ligd/publications/KLOVER-CAV11.pdf
这个项目是基于KLEE的C++的扩展。我找不到代码。
这篇论文提出了一些很好的问题，比如： How to extend a symbolic executor to handle C++ features? 但是没有说怎么实现。
只是举了添加字符串的长度限制的例子。
while in [5] we describe how to extend KLOVER for GPU programs。参考论文5 可以看看，如果要用GPU扩展的话。
compare symbolic execution with other symbolic methods [6] for concurrent programs. 参考论文6，多线程。


# SATzilla
我的论文是2012的，http://www.cs.ubc.ca/labs/beta/Projects/SATzilla/SATzilla2012final.pdf
看烦了，这论文写得太简略了.只有2页关于自己选择SAT的步骤和半页参考链接。
而且很有意思的，他的实现涉及MATLAB，JAVA，RUBY等：
Our SATzilla version used in [9] was based on classification models built in Matlab, and its execution required the installation of the free Matlab runtime environment (MRE). In order to avoid the need for installing MRE, we now con- verted our Matlab-built models to Java and provide Java code to make predictions using them. Thus, running SATzilla2012 now only requires the scripting lan- guage Ruby (which is used for running the SATzilla pipeline).
2017的代码，只有bin的脚本，jar包，以及例子，没有代码. 2012的代码也运行不起来。
http://www.cs.ubc.ca/labs/beta/Projects/SATzilla/Report_SAT_features.pdf
关于如何选择SAT的特性的。

# LLBMC
https://link.springer.com/content/pdf/10.1007%2F978-3-642-28756-5_44.pdf
这个项目是基于LLVM对C的Bound Model Checker（目标是C的内存错误）。它先把C代码转成LLVM的IR，再转成logical representation并利用重写规则简化，然后将简化的方程最终传到STP（a SMT solver）。
LLBMC requires llvm-gcc (version 2.9) in order to convert C input files to LLVM’s intermediate representation. 

# DIMACS
一种cnf（主要）的表示方式。举例：
```
c
c start with comments
c
c 
p cnf 5 3
1 -5 4 0
-1 5 3 4 0
-3 -4 0
```
- the preamble:
The file can start with comments, that is lines begining with the character c.
- the problem line:
```
p cnf nbvar nbclauses
```
- the clause sections:
Each clause is a sequence of distinct non-null numbers between -nbvar and nbvar ending with 0 on the same line; it cannot contain the opposite literals i and -i simultaneously. Positive numbers denote the corresponding variables. Negative numbers denote the negations of the corresponding variables.

