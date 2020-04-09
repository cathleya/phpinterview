# PHP面试-PHP7新特性

1.类型的声明。
可以使用字符串(string), 整数 (int), 浮点数 (float), 以及布尔值 (bool)，来声明函数的参数类型与函数返回值。

2.新增操作符“<=>”

3.新增操作符“??”

如果变量存在且值不为NULL， 它就会返回自身的值，否则返回它的第二个操作数。

4.define() 定义常量数组

5.匿名函数

$anonymous_func = function(){return 'function';};
echo $anonymous_func(); // 输出function

6.命名空间引用优化

// PHP7以前语法的写法 
use FooLibrary\Bar\Baz\ClassA; 
use FooLibrary\Bar\Baz\ClassB; 
// PHP7新语法写法 
use FooLibrary\Bar\Baz\{ ClassA, ClassB};

7.异常处理

8.foreach

9.list

# redis的使用和数据

redis数据类型：String，Hash，List，Set，Sorted Set


# Pub/Sub，Transactions


# redis的队列使用案例
  
    商品秒杀与超卖问题：将要促销的商品数量以队列的方式存入redis中，每当用户抢到一件促销商品则从队列中删除一个数据，确保商品不会超卖，且效率极高。 把要秒杀的商品放入到队列中，因为pop操作是原子的，即使有很多用户同时到达，也是依次执行，文件锁和事务在高并发下性能下降很快，当然还要考虑其他方面的东西，比如抢购页面做成静态的，通过ajax调用接口，其中也可能会出现一个用户抢多次的情况，这时候需要再加上一个排队队列和抢购结果队列及库存队列。高并发情况下，将用户进入排队队列，用一个线程循环处理从排队队列取出一个用户，判断用户是否已在抢购结果队列，如果在，则已抢购，否则未抢购，库存减1，写数据库，将用户入结果队列。 

需要一个排队队列(比如：queue:1,以user_id为值的列表)和抢购结果队列(比如：order:1,以user_id为值的列表)及库存队列(比如上面的goods_store:1)。高并发情况，先将用户进入排队队列，用一个线程循环处理从排队队列取出一个用户，判断用户是否已在抢购结果队列，如果在则已抢购，否则未抢购，接着执行库存减1，写入数据库，将此user_id用户同时也进入结果队列。


