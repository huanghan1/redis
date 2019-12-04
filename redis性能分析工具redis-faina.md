redis性能分析工具redis-faina

1. redis-faina安装
    该工具是用python写的，不需要安装什么依赖包，只需要将redis-faina.py下载即可：
     git clone https://github.com/facebookarchive/redis-faina.git
 2. redis-faina使用
     使用也非常简单，help一下
    $ ./redis-faina.py -h
    usage: redis-faina.py [-h] [--prefix-delimiter PREFIX_DELIMITER]
                      [--redis-version REDIS_VERSION]
                      [input]

    positional arguments:
     input                 File to parse; will read from stdin otherwise

   optional arguments:
     -h, --help            show this help message and exit
     --prefix-delimiter PREFIX_DELIMITER
                        String to split on for delimiting prefix and rest of
                        key
    --redis-version REDIS_VERSION
                        Version of the redis server being monitored
                        
    其中--prefix-delimiter主要用于统计前缀的key的数据。
    可以通过redis MONITOR命令以及管道进行分析，比如：
    
    redis-cli -p 6379 MONITOR | head -n <NUMBER OF LINES TO ANALYZE> | ./redis-faina.py [options]
    
   $ redis-cli -p 6379 MONITOR > m.log
   
   $ ./redis-faina.py ./m.log
    
    输出如下
$ ./redis-faina.py ./m.log 
Overall Stats
========================================
Lines Processed     44          -- 总命令数
Commands/Sec        0.63        -- qps

Top Prefixes                    -- 前缀最多的数据
========================================
n/a

Top Keys                        -- 使用最多的key
========================================
userToken       21  (47.73%)
userSession     21  (47.73%)

Top Commands                    -- 使用的最多的命令
========================================
HGET        40  (90.91%)
HSET        2   (4.55%)
COMMAND     1   (2.27%)

Command Time (microsecs)        -- 请求的响应时间分布
========================================
Median      394010.0
75%         447155.0
90%         8218888.25
99%         16510561.75

Heaviest Commands (microsecs)   -- 总体耗时最多的命令
========================================
HGET        57269310.75
HSET        8219184.75
COMMAND     4601067.25

Slowest Calls                   -- 慢请求列表
========================================
16510561.75     "HGET" "userSession" "7b8a4f64e03c7e24586f2fb2d705b232"
14359969.25     "HGET" "userSession" "1dcb54f57c3d9775c713ea8f97ae0ebc"
9015870.0       "HGET" "userSession" "7b8a4f64e03c7e24586f2fb2d705b232"
8881043.0       "HGET" "userSession" "1dcb54f57c3d9775c713ea8f97ae0ebc"
8218888.25      "HSET" "userSession" "b52de307a793b35d79e914a1d7b26028" "1502348079"
4601067.25      "COMMAND"
1396710.25      "HGET" "userSession" "123"
1305825.25      "HGET" "userSession" "123"
    
    
    
  3. 注意 
  由于redis MONITOR输出的只有请求开始的时间，所以在一个非常繁忙的redis实例中，根据该请求的开始时间以及下一个请求的开始时间，可以大概估算出一个请求的执行时间。由此可以看出，redis-faina统计的时间并不是十分精确的，尤其在分析一个非常闲的redis实例时，分析的结果可能差的很多。


   https://blog.csdn.net/cjfeii/article/details/77069778
