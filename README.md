其他语言：[English](README_en.md)

# PHP-ChildProcess

一个CLI下易用简单的管理进程的库

# 安装

添加 `"pagon/childprocess": "*"` 到 [`composer.json`](http://getcomposer.org):

```
composer.phar install
```

# 目录

- [使用ChildProcess](#childprocess-manager)
- [创建子进程](#create-child-process)
  - [平行运行](#parallel-works)
    - [自动运行](#automatic-run)
    - [手动运行](#manually-run)
    - [手动运行等待](#manually-join)
  - [通过PHP文件Fork](#fork-php-file)
  - [发送消息](#send-message)
  - [Spawn命令](#spawn-the-command)
  - [高级用法](#advance-usage)
- [事件](#events)
  - [ChildProcess](#manager-events)
  - [Process](#process-events)

# Usage

## ChildProcess Manager

控制当前进程

```php
declare(ticks = 1) ;

$manager = new ChildProcess();

$manager->on('exit', function () use ($master) {
    error_log('exit');
    exit;
});

// 做其他事情或等待
```

## Create Child Process

### Parallel Works

在子平行空间运行闭包函数

#### Automatic run

```php
declare(ticks = 1) ;

$manager = new ChildProcess();

$child = $manager->parallel(function () {
    sleep(10);
    // 做其他事情
});
```

> 如果子进程正在工作，主进程退出，那将没法来handle事件

#### Manually Run

```php
declare(ticks = 1) ;

$manager = new ChildProcess();

$child = $manager->parallel(function () {
    // to do something
    sleep(10);
    error_log('child execute');
}, false);

$child->on('exit', function ($status) {
    error_log('child exit ' . $status);
});

// Will run but don't wait the child exit
$child->run()

while(1) { /*to do something */}
```

#### Manually Join


```php
declare(ticks = 1) ;

$manager = new ChildProcess();

$child = $manager->parallel(function () {
    // to do something
    sleep(10);
    error_log('child execute');
}, false);

$child->on('exit', function ($status) {
    error_log('child exit ' . $status);
});

// Will wait the child exit
$child->join();
```

### Fork PHP file

    Run the PHP file in parallel child process space

主进程：

```php
declare(ticks = 1) ;

$manager = new ChildProcess();

$child = $manager->fork(__DIR__ . '/worker.php', false);

$child->on('exit', function ($status) {
    error_log('child exit ' . $status);
});

$child->join();
```

PHP文件：

```php
$master // The parent process
$child  // Current process
// Some thing to do in child process
```

### Send message

父子进程可以使用消息来通信

```php
declare(ticks = 1) ;

$manager = new ChildProcess();

$manager->listen();

$child = $manager->parallel(function (Process $master, Process $child) {
    $child->listen();

    $master->on('message', function ($msg) {
        error_log('child revive message: ' . $msg);
    });

    $master->send('hello master');

    error_log('child execute');
}, false);

$child->on('message', function ($msg) {
    error_log('parent receive message: ' . $msg);
});

$child->send('hi child');

$child->join();
```

### Spawn the command

在子进程运行命令，并且捕获输出

```php
declare(ticks = 1) ;

$manager = new ChildProcess();

$child = $manager->spawn('/usr/sbin/netstat');

$child->on('stdout', function ($data) {
    error_log('receive stdout data: '  . $data);
    // to save data or process it
});

$child->on('stderr', function ($data) {
    error_log('receive stderr data: '  . $data);
    // to save data or process something
});

$child->join();
```

### Advance usage

设置选项

当前支持的选项列表：

```php
array(
    'cwd'      => false,        // Current working deirectory
    'user'     => false,        // Startup user
    'env'      => array(),      // Enviroments
    'timeout'  => 0,            // Timeout
    'init'     => false,        // Init callback
    'callback' => false         // Child startup callback
)
```

一些用法：

```php
declare(ticks = 1) ;

$manager = new ChildProcess();

$child = $manager->spawn('/usr/sbin/netstat', array(
    'timeout' => 60 // Will wait 60 seconds
    'callback' => function(){ error_log('netstat start'); }
));

$child->on('stdout', function ($data) {
    echo $data;
    // to save data or process it
});

$child->join();
```

## Events

### Register Events

```
$manager = new ChildProcess();

$manager->on('tick', function(){
    // Check something
});
```

### Manager Events

- `tick`      每个tick都会触发，主要用于监控一些行为来及时反馈到管理器
- `listen`    监听消息队列
- `exit`      进程退出
- `quit`      收到SIGQUIT信号
- `signal`    收到任何的信号

### Process Events

- `listen`    当管理器开始监听队列时触发
- `exit`      当退出时
- `run`       当手动运行时
- `init`      当子进程创建完成时
- `fork`      当fork时

# License

[MIT](./LICENSE)