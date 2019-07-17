https://stackoverflow.com/questions/41452314/static-analysis-tools-for-llvm-ir

# KLEE
这个编译起来非常复杂。我上次在mac上编译不成功。我参考的是这个 https://github.com/tum-i22/klee-install。只能用Ubuntu才能编译成功。好像依赖库uclibc缺乏头文件，提示我可以继续用KLEE，但是会有限制，应该是最大内存的限制。但是SAT都是开很大内存，几百万个参数一起算的，后面好像编译不成功是LLVM的版本太新了，我的用8，他配套的是RELEASE_342.有个数据结构变了。浪费我2天时候。我把主要的错误记录在这里了，https://github.com/sancao2/klee-install。
后来用docker，半个小时就搭建好了。
通过编译这个KLEE，了解了他为什么依赖这些库：
## klee-uclibc
uclibc is for embedded systems，MMU-less systems. 因为klee内存密集型，所以自己改了uclibc, https://www.uclibc.org/FAQ.html
## minisat
## STP
SMT solver aimed at bitvectors and arrays. 这个比较老了，最迟参加了2014 SMTCOMP。https://github.com/stp/stp
## z3
SMT solver from Microsoft Research, http://theory.stanford.edu/~nikolaj/programmingz3.html, 这个文档第一个架构图可以参考，整个文档都是很好的学习资料。

他的架构还是很经典的。后面的工程CLOVER不过是他的C++的“补丁”。

# 
