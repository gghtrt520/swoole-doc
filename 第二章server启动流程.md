### Swoole暴露给php用于启动的类是swoole\_server

###### 其中核心的三个方法是

1. PHP\_METHOD\(swoole\_server, set\) 用于设置启动参数,e.g worker\_num设置worker进程数量

2. PHP\_METHOD\(swoole\_server, on\) 设置事件的回调函数 最主要的是onReceive事件的回调，其实就是接受到客户端请求swoole所调用的php代码

3. PHP\_METHOD\(swoole\_server, start\)  启动服务，创建worker进程，绑定端口，创建reactor线程等操作让服务器完成启动

### swoole启动流程

1 .配置好启动参数与回调函数后，在php执行启动代码

```php
$this->sw->start();
```

这将会调用扩展中的 PHP\_METHOD\(swoole\_server, start\) 方法

接下来是

```c
int swServer_start(swServer *serv)
```

这个方法中会先启动manager进程，然后由manager进程派生多个worker进程

```c
 //factory start
    if (factory->start(factory) < 0)
    {
        return SW_ERR;
    }
 //实际会调用到
 static int swFactoryProcess_start(swFactory *factory)
```

之后 swServer\_start 中会调用swServer\_start\_proxy，用于启动reactor线程，以及主进程进入端口监听循环

主进程先创建main\_reactor,实际为创建epoll,并监听端口描述符，并设置连接处理函数swServer\_master\_onAccept，

之后创建多个reactor线程，在这些线程中简单知道worker进程的管道

