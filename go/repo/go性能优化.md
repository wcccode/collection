1.避免[]byte转化成string
2.重用缓存或者对象，使用sync.Pool
3.预先分配slice
4.使用缓存减少io调用
5.减少使用无用string（如byte转化成整数时，可以实现byteToBase10函数）



##benchcmp##
http://lk4d4.darth.io/posts/bench/

##go-debugging-profiling-optimization##
http://tonybai.com/2015/08/25/go-debugging-profiling-optimization/

https://signalfx.com/blog/a-pattern-for-optimizing-go-2/
