# PHP面试-框架laravel

1. laravvel的特性：优雅，简洁，工程化；使用了非常多的设计模式，包括依赖注入，Ioc模式,中间件，门脸模式；

2. laravel的社区
    [官网](https://laravel.com)
    [中文社区](https://laravel-china.org)
    [5.4中文文档](https://d.laravel-china.org/docs/5.4)
    [laravel源码地址](https://github.com/laravel/laravel)
    
3. 包含的功能
    队列，搜索，数据迁移，定时脚本

4. laravl的核心思想
    服务容器    控制反转ioc
    服务提供者 【框架中写死的，配置文件中】boot,register,defer
    门脸模式 
       
5. laravel有那些特点
    1.强大的rest router:用简单的回调函数就可以调用,快速绑定controller和router

    2.artisan:命令行工具,很多手动的工作都自动化、composer

    3.blade模板可继承,简化view的开发和管理,且渲染速度更快

    4.ORM操作数据库

    5.migration:管理数据库和版本控制
 
# laravel 性能优化  

路由缓存

php artisan route:cache

配置缓存

php artisan config:cache

类缓存

php artisan optimize

优化sql在页面上的重复查询

with/load

找出比较慢的sql查询

\DB::Listen()


#后端架构的套路

负载均衡，动静分离，读写分离，缓存，分布式

cdn分片：http://mengkang.net/641.html

