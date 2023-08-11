# JVM

# Class Loader
## Calss Order
### loader 
### verify
### link
### init
## Bootstrap, Application, Extension


# GC
SafePoint 就是一个安全点，可以理解为用户线程执行过程中的一些特殊位置。SafePoint 保存了当前线程的 上下文，当线程执行到这些位置的时候，说明线程当前的状态是 
确
定
的
，线程有哪些对象、使用了哪些内存。

因此，只有用户线程处于 SafePoint 的时候，线程才可以安全阻塞。

这意味着，Stop-The-World 需要所有的用户线程处于 SafePoint。
test
