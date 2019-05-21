Welcome to the FNLog wiki!  
# Introduction:  
[![Build Status](https://travis-ci.org/zsummer/FNLog.svg?branch=master)](https://travis-ci.org/zsummer/FNLog)
```
FNLog is an open source C++ lightweight & cross platform log library. It's a iteration version from log4z.
It provides in a C++ application log and trace debug function for 7*24h service program.  
Support 64/32 of windows/linux/mac/android/iOS.   
FNLog是一款开源的轻量级高性能的跨平台日志库, 从log4z迭代而来, 主要针对7*24小时服务器程序的日志输出与跟踪调试,   
支持64/32位的windows/linux/mac/android/iOS等操作系统.  
```
# 特性列表:  

- [x] **MIT开源授权 授权的限制非常小.**   
- [x] **跨平台支持linux & windows & mac, 仅头文件实现.**   
  > 通过merge_header.sh脚本可以合并为单头文件实现.  
- [x] **自动生命周期管理, 无需关心销毁问题.**    
- [x] **CHANNEL-DEVICE M:N组合方式的多管道-多输出端设计.**    
  > 对不同Channel的工作模式进行完全隔离, 提供高效并且灵活的组合使用方案.   
- [x] **灵活的过滤机制.**  
  > 支持Channel级别的优先级过滤和标签过滤.   
  > 支持Device终端输出的优先级过滤和标签过滤.   
- [x] **灵活的日志分流机制**  
  > 不同Channel的日志写入相互隔离,独立控制.   
  > 可以挂靠任意类型和任意数量的输出设备(文件/UDP/屏幕).   
  > 可以按照过滤策略单独定义每个输出端的内容 .    
- [x] **可在CHANNEL级别并行开启同步写入模式和异步写入模式.**  
- [x] **支持日志文件回滚, 支持屏显日志染色输出.**   
- [x] **C++ STREAM 输入风格, 类型安全.**  
- [x] **支持配置文件实时(延迟)热更新开关.**   
- [x] **高性能.**   
 > 文件写入可以达到200万行/秒, UDP 50万/秒. 
- [x] **自定义的配置解析器 简洁易用**      


# 配置文件示例:   

文件输出压测配置.  详见stress_test.cpp  
out_type改为udp即可成为udp输出的压测配置.

```   YAML
# 压测配表  
 # 0通道为多线程文件输出和一个CLS筛选的屏显输出 
 - channel: 0
    sync: null
    filter_level: trace
    filter_cls_begin: 0
    filter_cls_count: 0
    -device: 0
        disable: false
        out_type: file
        filter_level: trace
        filter_cls_begin: 0
        filter_cls_count: 0
        path: "./log/"
        file: "$PNAME_$YEAR$MON$DAY"
        rollback: 4
        limit_size: 100 m #only support M byte
    -device:1
        disable: false
        out_type: screen
        filter_cls_begin: 1
        filter_cls_count: 1
 # 1通道为多线程不挂任何输出端 
 - channel: 1

 # 2通道为单线程异步写文件(回环队列) 
 - channel: 2
    sync: ring #only support single thread
    -device: 0
        disable: false
        out_type: file
        file: "$PNAME_$YEAR_ring"
        rollback: 4
        limit_size: 100 m #only support M byte

 # 3通道为单线程异步无输出端(回环队列) 
 - channel: 3
    sync: ring #only support single thread

 # 4通道为单线程同步写文件 
 - channel: 4
    sync: sync #only support single thread
    -device: 0
        disable: false
        out_type: file
        file: "$PNAME_$YEAR"
        rollback: 4
        limit_size: 100 m #only support M byte

 # 5通道为单线程无输出端 
 - channel: 5
    sync: sync #only support single thread

```  

# 日志文件输出  
##### 文件名可配  
回滚日志列表  
```
stress_test_2019.log
stress_test_2019.log.1
stress_test_2019.log.2
stress_test_2019.log.3
```
##### 日志内容  
示例  
```
[20190514 16:47:20.536][ALARM] [15868]channel [0] start.

[20190514 16:47:20.548][DEBUG] [15868] FNLog\tests\simple_test.cpp:<40> main log init success
[20190514 16:47:20.548][DEBUG] [15868] FNLog\tests\simple_test.cpp:<42> main now time:1557823640;
[20190514 16:47:20.548][DEBUG] [15868] FNLog\tests\simple_test.cpp:<44> main hex text:
	[
	[0x7FF67B00B4B4:  f  n  l  o  g  . 
	[0x7FF67B00B4B4: 66 6E 6C 6F 67 00 
	]
	
[20190514 16:47:20.548][ALARM] [15868] FNLog\tests\simple_test.cpp:<46> main finish
```
# Example  Read Config 
配置文件   
``` yaml
# 配表文件  

 hot_update: true
 # 0通道为多线程文件输出和一个CLS筛选的屏显输出
 - channel: 0  
    filter_level: trace
    filter_cls_begin: 0
    filter_cls_count: 0
    -device: 0
        disable: false
        out_type: file
        filter_level: trace
        filter_cls_begin: 0
        filter_cls_count: 0
        path: "./log/"
        file: "$PNAME_$YEAR$MON$DAY"
        rollback: 4
        limit_size: 1000 m #only support M byte
    -device: 1
        disable: false
        out_type: screen
        filter_cls_begin: 0
        filter_cls_count: 0
 # 1通道为多线程不挂任何输出端
 - channel: 1

 # 2通道为单线程同步写文件
 - channel: 2
    sync_write: 1 #only support single thread
    -device: 0
        disable: false
        out_type: file

 # 3通道为单线程无输出端
 - channel: 3
    sync_write: 1 #only support single thread

```
代码   
```  C++ 
#include "fn_log.h"
#include "fn_load.h"

int main(int argc, char* argv[])
{
    int ret = FNLog::LoadAndStartDefaultLogger("./log.yaml");
    if (ret != 0)
    {
        return ret;
    }
    
    int limit_second = 50;
    while (limit_second > 0)
    {
        LOGD() << "default channel.";
        LOGCD(1, 0) << "channel:1, cls:0.";
        LOGCD(2, 0) << "channel:2, cls:0.";
        LOGCD(3, 0) << "channel:3, cls:0.";
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));
        limit_second--;
    }


    LOGA() << "finish";
    return 0;
}

```  
# FAST USE EXAMPLE  
``` C++
#include "fn_log.h"

int main(int argc, char* argv[])
{
    int ret = FNLog::FastStartDefaultLogger();
    if (ret != 0)
    {
        return ret;
    }

    LOGA() << "log init success";

    LOGD() << "now time:" << time(nullptr) << ";";
    
    LOGA() << "finish";

    return 0;
}
```


# How to compile  
直接嵌入头文件即可  

# How to test  
``` shell
mkdir build
cd build
cmake ..
make
cd ../bin
./buffer_test
./buffer_correct_test
./load_config_test
./simple_test
./stress_udp_test
./stress_test
./multi-thread_test
./multi-thread_write_file_test
```

# About The Author  
Author: YaweiZhang  
Mail: yawei.zhang@foxmail.com  
QQGROUP: 524700770  
GitHub: https://github.com/zsummer/FNLog  

