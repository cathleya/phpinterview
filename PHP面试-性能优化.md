# PHP面试-性能优化

## 性能调优整体思路
1.空间换时间
对热点数据缓存，减少数据查询时间。
2.分而治之
将大任务切片，分开执行。
3.异步处理
若业务链中有某一环节耗时严重，则该环节将拉长业务链的整体耗时。可以将耗时业务采用消息队列异步化，从而缩短业务链耗时。
4.并行处理
采用多进程、多线程同时处理，提升处理速度。
5.离用户更近一点
如CDN技术，将静态资源放到离用户更近的地方，从而缩短请求静态资源的时间。
6.提升可扩展性
采用业务模块化、服务化的手段，提升系统的可扩展性，从而可根据业务需求实现弹性计算。

## 具体的优化方案

1. 数据库优化
    
    *mysql性能优化
    
    1.数据架构的优化

　　　　　　（1）主从复制
            mysql支持单向、异步复制，复制过程中一个服务器充当主服务器，而一个或多个其它服务器充当从服务器。
　　　　　　（2）读写分离
　　　　　　读写分离的实现原理就是在执行SQL语句的时候，判断到底是读操作还是写操作，把读的操作转向到读服务器上（从服务器，一般是多台），写的操作转到写的服务器上（主服务器，一般是一台，视数据量来看）。

　　　　　　（3）双主热备

　　　　　　（4）负载均衡　通过LVS的三种模式实现的、Mysql数据库中间件实现的
　　　　　　
　　2.数据库操作细节的优化
　　
　　尽量使用InnoDB存储引擎，因为它支持事务、外键、使用独立表空间、使用的是行级锁
    
    建立数据库索引，选择适当的字段
    避免使用select *
    用Where子句替换HAVING子句
    选取最适用的字段属性,应该尽量把字段设置为NOT NULL
    使用连接（JOIN）来代替子查询(Sub-Queries)
    使用联合(UNION)来代替手动创建的临时表
    尽量少使用 LIKE 关键字和通配符
    使用事务和外键
         
2. php-fpm优化
   
   1.调高linux内核打开文件数量，可以使用这些命令(必须是root帐号)(我是修改/etc/rc.local，加入ulimit -SHn 51200的)
   echo `ulimit -HSn 65536` >> /etc/profile
echo `ulimit -HSn 65536` >> /etc/rc.local
source /etc/profile 
　如果`ulimit -n`数量依旧不多(即上面配置没生效)的话, 可以在 /etc/security/limits.conf 文件最后加上
　* soft nofile 51200
  * hard nofile 51200

  2.与Nginx使用Unix域Socket通信(Nginx和php-fpm在同一台服务器)

    Unix域Socket因为不走网络，的确可以提高Nginx和php-fpm通信的性能，但在高并发时会不稳定。

　　Nginx会频繁报错：connect() to unix:/dev/shm/php-fcgi.sock failed (11: Resource temporarily unavailable) while connecting to upstream

　　可以通过下面两种方式提高稳定性：
　　1）调高nginx和php-fpm中的backlog
    　　 配置方法为：在nginx配置文件中这个域名的server下，在listen 80后面添加default backlog=1024。
     　　同时配置php-fpm.conf中的listen.backlog为1024，默认为128。
　　2）增加sock文件和php-fpm实例数
     　　再新建一个sock文件，在Nginx中通过upstream模块将请求负载均衡到两个sock文件背后的两套php-fpm实例上。
     　
 3.php-fpm进程管理方式
     　
     　1.pm = dynamic;表示php-fpm进程数是动态的，最开始是pm.start_servers指定的数量，如果请求较多，则会自动增加，保证空闲的进程数不小于pm.min_spare_servers，如果进程数较多，也会进行相应清理，保证多余的进程数不多于pm.max_spare_servers
        
     2.pm = static;表示php-fpm进程数是静态的, 进程数自始至终都是pm.max_children指定的数量，不再增加或减少
     
     pm.max_children = 300; 静态方式下开启的php-fpm进程数量
　　pm.start_servers = 20; 动态方式下的起始php-fpm进程数量
　　pm.min_spare_servers = 5; 动态方式下的最小php-fpm进程数量
　　pm.max_spare_servers = 35; 动态方式下的最大php-fpm进程数量
　　
　　如果pm为static, 那么其实只有pm.max_children这个参数生效。系统会开启设置数量的php-fpm进程

　　　　如果pm为dynamic, 那么pm.max_children参数失效，后面3个参数生效。系统会在php-fpm运行开始的时候启动pm.start_servers个php-fpm进程，然后根据系统的需求动态在pm.min_spare_servers和pm.max_spare_servers之间调整php-fpm进程数
　　　　
　　　　内存小的建议用动态（pm = dynamic），内存大的建议用静态（pm = static）。
　　　　
　　静态的max_children设置的数值推荐：
　　  8GB内存可以设置为100，那么php-fpm耗费的内存就能控制在 2G-3G的样子。
　　  
　　  php-fpm 单个进程需要20～30m内存
　　  
　　  3.pm = ondemand模式，这种模式和pm = dynamic相反，把内存放在第一位，他的工作模式很简单，每个闲置进程，在持续闲置了pm.process_idle_timeout秒后就会被杀掉，有了这个模式，到了服务器低峰期内存自然会降下来，如果服务器长时间没有请求，就只会有一个php-fpm主进程，当然弊端是，遇到高峰期或者如果pm.process_idle_timeout的值太短的话，无法避免服务器频繁创建进程的问题，因此pm = dynamic和pm = ondemand谁更适合视实际情况而定。


4.pm.max_requests = 10240;该值的意思是发送多少个请求后会重启该线程，我们需要适当降低这个值，用以让php-fpm自动的释放内存。
这样实际上的内存消耗是max_children*max_requests*每个请求使用内存

5.request_terminate_timeout = 30;

　　　　最大执行时间, 在php.ini中也可以进行配置(max_execution_time)

　　  
6.配置php慢日志，用于监控

request_slowlog_timeout = 10s

slowlog = log/$pool.log.slow  

rlimit_files = 1024; 增加php-fpm打开文件描述符的限制　　      

pm.status_path = /status
　
   
3. PHP优化

    1.少安装PHP模块, 费内存
    
    2.开启Opcache
    zend_extension=opcache.so 
    opcache.enable=1 
    opcache.enable_cli=1
    
    3.使用GCC 4.8以上进行编译
    
只有GCC 4.8以上PHP才会开启Global Register for opline and execute_data支持, 这个会带来5%左右的性能提升(Wordpres的QPS角度衡量)

    4.开启HugePage （根据系统内存决定）
    
    sudo sysctl vm.nr_hugepages=512
    
    分配512个预留的大页内存：
    
    cat /proc/meminfo |grep Huge
    
    AnonHugePages: 106496 kb
    HugePages_Total: 512
    HugePages_Free: 504
    HugePages_Rsvd: 27
    HugePages_Surp: 0
    HugePagesize: 2048 kb
        
    然后在php.ini中加入：
    opcache.huge_code_pages=1
    
    这样一来，PHP会把自身的text段，以及内存分配中的huge都采用大页内存来保存，减少TLB miss,从而提高性能
    
    5.PGO (Profile Guided Optimization）
    第一次编译成功后，用项目代码去训练PHP，会产生一些profile信息，最后根据这些信息第二次gcc编译PHP就可以得到量身定做的PHP7

需要选择在你要优化的场景中: 访问量最大的, 耗时最多的, 资源消耗最重的一个页面.

4. nginx优化

worker_processes
nginx运行工作进程个数，一般设置cpu的核心或者核心数x2，如：worker_processes 4;
worker_cpu_affinity
运行CPU亲和力，与worker_processes对应，如：worker_cpu_affinity 0001 0010 0100 1000;
worker_rlimit_nofile
Nginx最多可以打开文件数，与ulimit -n保持一致，如：worker_rlimit_nofile 65535;
events
事件处理模型。如：
events {
  use epoll;
  worker_connections 65535;
  multi_accept on;
}

multi_accept：小开大关，有利于内存

giz压缩
防盗链
fastcgi调优
连接超时时间
负载均衡，反向代理
访问限流

#限制用户连接数来预防DOS攻击
limit_conn_zone $binary_remote_addr zone=perip:10m;
limit_conn_zone $server_name zone=perserver:10m;
#限制同一客户端ip最大并发连接数
limit_conn perip 2;
#限制同一server最大并发连接数
limit_conn perserver 20;
#限制下载速度，根据自身服务器带宽配置
limit_rate 300k; 



## 分布式结构
3.1 微服务
在分布式结构中，将业务进行服务化。
所谓服务化，就是将一个完整的应用，根据逻辑功能拆分成多个子应用，每个应用都有各自独立的war包，部署在不同的服务器上。
服务化有如下好处：

系统逻辑清晰、耦合度低。
可以根据服务的业务量合理分配计算资源。
问题更容易排查。

3.2 分布式数据库
分布式数据库是对数据库进行分库分表，将数据分片存储在不同数据库的不同表中，并在数据库存储层之上增加了数据访问层，可通过hash找到数据所处的库与表。
很多公司都基于Mariadb开发自己的分布式数据库，该数据库之上建立数据访问层，用来实现分库分表，并对上层透明。

3.3 注册中心 Zookeeper
注册中心用来管理所有的分布式服务。当A服务需要请求B服务时，A服务首先会从注册中心获取B服务的IP，然后向该IP发起请求。
此外，当服务不可用、或增加新的服务时，配置中心就会相应地在服务列表中删除、增加该项服务。

   




