---
weight: 6
title: "co/log"
---

include: [co/log.h](https://github.com/idealvin/co/blob/master/include/co/log.h).


## Basic introduction


`co/log` is a C++ streaming log library similar to [google glog](https://github.com/google/glog), which prints logs like this:


```cpp
LOG << "hello world" << 23;
```


co/log divides logs into five levels: debug, info, warning, error, fatal, and provides a series of macros for printing logs of different levels. **Printing the fatal level log will terminate the program**, and co/log will also print the stack information before the program exits.


co/log uses an asynchronous implementation. The log is first written to the cache, and after a certain amount or more than a certain period of time, the background thread writes all data in the cache to the file at a time. The performance is improved by about 20 to 150 times compared with glog on different platforms. The following table shows the test results of printing 1 million info logs (about 50 bytes each) by a single-thread on different platforms:

| log vs glog | google glog | co/log |
| --- | --- | --- |
| win2012 HHD | 1.6MB/s | 180MB/s |
| win10 SSD | 3.7MB/s | 560MB/s |
| mac SSD | 17MB/s | 450MB/s |
| linux SSD | 54MB/s | 1023MB/s |





## Initialize and close the log system


### log::init


```cpp
void init();
```


- This function initializes the log library and starts the log thread. It needs to be called once at the beginning of the main function.
- This function adds multi-thread protection internally, and it is safe to call this function multiple times.
- co/log depends on [co/flag](../flag/), you need to call `flag::init()` before calling this function.



- Example



```cpp
#include "co/flag.h"
#include "co/log.h"

int main(int argc, char** argv) {
    flag::init(argc, argv);
    log::init();
}
```




### log::exit


```cpp
void exit();
```


- The above 2 functions are equal, which write logs in the cache to a file and exit the log thread.
- When the program exits normally, co/log will automatically call this function.
- It is safe to call this function multiple times.
- co/log internally captures signals such as `SIGINT, SIGTERM, SIGQUIT`, and calls this function before the program exits.




### log::close

```cpp
void close();
```

- The same as `log::exit()`.



### log::set_write_cb

```cpp
void set_write_cb(const std::function<void(const void*, size_t)>& cb);
```

- By default, co/log writes logs to a local file. Users can set a callback to write logs to different destinations through this API.
- The callback has 2 parameters, a pointer to the log buffer and its length. The buffer may contain more than one logs.
- Once a callback is set, co/log will stop writing logs to the local file, which can be changed by setting the config item `also_log_to_local` to true.




### log::set_single_write_cb
```cpp
void set_single_write_cb(const std::function<void(const void*, size_t)>& cb);
```

- Similar to `log::set_write_cb()`, but only writes a single log each time.
- It may be useful when users want to send logs by UDP.





## Printing logs


### Basic usages


```cpp
#define DLOG  if (FLG_min_log_level <= log::xx::debug)   _DLOG_STREAM
#define LOG   if (FLG_min_log_level <= log::xx::info)     _LOG_STREAM
#define WLOG  if (FLG_min_log_level <= log::xx::warning) _WLOG_STREAM
#define ELOG  if (FLG_min_log_level <= log::xx::error)   _ELOG_STREAM
#define FLOG  _FLOG_STREAM << "fatal error! "
```


- The above 5 macros DLOG, LOG, WLOG, ELOG, FLOG are used to print 5 levels of logs respectively, they are **thread safe**.
- These macros are actually references to [fastream](../fastream/), so any type supported by `fastream::operator<<` can be printed.
- These macros will automatically add a '\n' at the end of each log, and users do not need to manually enter a newline character.
- The first 4 types will only print the log when the `FLG_min_log_level` is not greater than the current log level. The user can set FLG_min_log_level to a larger value to disable low-level logs.
- Print the fatal level log, which means that the program has a fatal error. The log library will print the function call stack information of the current thread and terminate the program.



- Example



```cpp
DLOG << "this is DEBUG log "<< 23;
LOG << "this is INFO log "<< 23;
WLOG << "this is WARNING log "<< 23;
ELOG << "this is ERROR log "<< 23;
FLOG << "this is FATAL log "<< 23;
```




### Condition Log


```cpp
#define DLOG_IF(cond) if (cond) DLOG
#define LOG_IF(cond) if (cond) LOG
#define WLOG_IF(cond) if (cond) WLOG
#define ELOG_IF(cond) if (cond) ELOG
#define FLOG_IF(cond) if (cond) FLOG
```


- The above 5 macros accept a conditional parameter cond, and only print the log when cond is true.
- The parameter cond can be any expression whose value is of type bool.
- Since the condition is judged in the first place, even if the log of the corresponding level is disabled, these macros will ensure that the cond expression is executed.



- Example



```cpp
int s = socket();
DLOG_IF(s != -1) << "create socket ok: "<< s;
 LOG_IF(s != -1) << "create socket ok: "<< s;
WLOG_IF(s == -1) << "create socket ko: "<< s;
ELOG_IF(s == -1) << "create socket ko: "<< s;
FLOG_IF(s == -1) << "create socket ko: "<< s;
```




### Print log every N entries


```cpp
#define DLOG_EVERY_N(n) _LOG_EVERY_N(n, DLOG)
#define LOG_EVERY_N(n) _LOG_EVERY_N(n, LOG)
#define WLOG_EVERY_N(n) _LOG_EVERY_N(n, WLOG)
#define ELOG_EVERY_N(n) _LOG_EVERY_N(n, ELOG)
```


- The above macro prints the log once every n entries, internally counted by [atomic operation](../atomic/), which is **thread safe**.
- The parameter n must be an integer greater than 0, and generally should not exceed the maximum value of the int type.
- When the parameter n is a power of 2, the log will be printed exactly once every n entries, otherwise there may be very few cases when it is not printed once every n entries.
- The first log will always be printed.
- The program will terminate as soon as the fatal log is printed, so FLOG_EVERY_N is not provided.



- Example



```cpp
// Print every 32 items (1,33,65...)
DLOG_EVERY_N(32) << "this is DEBUG log "<< 23;
LOG_EVERY_N(32) << "this is INFO log "<< 23;
WLOG_EVERY_N(32) << "this is WARNING log "<< 23;
ELOG_EVERY_N(32) << "this is ERROR log "<< 23;
```




### Print the first N logs


```cpp
#define DLOG_FIRST_N(n) _LOG_FIRST_N(n, DLOG)
#define LOG_FIRST_N(n) _LOG_FIRST_N(n, LOG)
#define WLOG_FIRST_N(n) _LOG_FIRST_N(n, WLOG)
#define ELOG_FIRST_N(n) _LOG_FIRST_N(n, ELOG)
```


- The above macro prints the first n logs, internally counted by [atomic operation](../atomic/), which is **thread safe**.
- The parameter n is an integer not less than 0 (no log will be printed when it is equal to 0). Generally, it should not exceed the maximum value of the int type.
- In general, do not use complex expressions for the parameter n.
- The program will terminate as soon as the fatal log is printed, so FLOG_FIRST_N is not provided.



- Example



```cpp
// print the first 10 logs
DLOG_FIRST_N(10) << "this is DEBUG log "<< 23;
LOG_FIRST_N(10) << "this is INFO log "<< 23;
WLOG_FIRST_N(10) << "this is WARNING log "<< 23;
ELOG_FIRST_N(10) << "this is ERROR log "<< 23;
```






## CHECK Assertion


```cpp
#define CHECK(cond) \
    if (!(cond)) _FLOG_STREAM << "check failed: "#cond "!"

#define CHECK_NOTNULL(p) \
    if ((p) == 0) _FLOG_STREAM << "check failed: "#p" mustn't be NULL! "

#define CHECK_EQ(a, b) _CHECK_OP(a, b, ==)
#define CHECK_NE(a, b) _CHECK_OP(a, b, !=)
#define CHECK_GE(a, b) _CHECK_OP(a, b, >=)
#define CHECK_LE(a, b) _CHECK_OP(a, b, <=)
#define CHECK_GT(a, b) _CHECK_OP(a, b, >)
#define CHECK_LT(a, b) _CHECK_OP(a, b, <)
```


- The above macros can be regarded as an enhanced version of assert, and they will not be cleared in DEBUG mode.
- These macros are similar to `FLOG` and can print fatal level logs.
- `CHECK` asserts that the condition cond is true, and cond can be any expression with a value of type bool.
- `CHECK_NOTNULL` asserts that the pointer is not NULL.
- `CHECK_EQ` asserts `a == b`.
- `CHECK_NE` asserts `a != b`.
- `CHECK_GE` asserts `a >= b`.
- `CHECK_LE` asserts `a <= b`.
- `CHECK_GT` asserts `a > b`.
- `CHECK_LT` asserts `a < b`.
- It is generally recommended to use `CHECK_XX(a, b)` first, they provide more information than `CHECK(cond)`, and will print out the values of parameters a and b.
- Types not supported by `fastream::operator<<`, such as iterator type of STL containers, cannot use the `CHECK_XX(a, b)` macros.
- When the assertion fails, the log library first calls `log::close()`, then prints the function call stack information of the current thread, and then exits the program.



- Example



```cpp
int s = socket();
CHECK(s != -1);
CHECK(s != -1) << "create socket failed";
CHECK_NE(s, -1) << "create socket failed"; // s != -1
CHECK_GE(s, 0) << "create socket failed";  // s >= 0
CHECK_GT(s, -1) << "create socket failed"; // s > -1

std::map<int, int> m;
auto it = m.find(3);
CHECK(it != m.end()); // Cannot use CHECK_NE(it, m.end()), the compiler will report an error
```




## Stack trace

`co/log` will print the function call stack when `CHECK` assertion failed, or an abnormal signal like `SIGSEGV` was caught. See details below:

![stack](https://idealvin.github.io/images/stack.png)(https://asciinema.org/a/435894)

On linux and macosx, [libbacktrace](https://github.com/ianlancetaylor/libbacktrace) is required, make sure you have installed it on your system. On linux, `libbacktrace` may have been installed within a newer version of gcc. You may find it in a directory like `/usr/lib/gcc/x86_64-linux-gnu/9`. Otherwise, you can install it by yourself as follow:

```sh
git clone https://github.com/ianlancetaylor/libbacktrace.git
cd libbacktrace-master
./configure
make -j8
sudo make install
```




## Configuration


### log_dir


```cpp
DEF_string(log_dir, "logs", "#0 log dir, will be created if not exists");
```


- Specify the log directory. The default is the `logs` directory under the current directory. If it does not exist, it will be created automatically.
- log_dir can be an absolute path or a relative path, and the path separator can be either '/' or '\'. It is generally recommended to use '/'.
- When the program starts, make sure that the current user has sufficient permissions, otherwise the creation of the log directory may fail.



### log_file_name


```cpp
DEF_string(log_file_name, "", "#0 name of log file, using exename if empty");
```


- Specify the log file name (without path), the default is empty, use the program name (`.exe` at the end will be removed), for example, the log file name corresponding to program xx or xx.exe is xx .log.
- If the log file name does not end with `.log`, co/log automatically adds `.log` to the end of it.



### min_log_level


```cpp
DEF_int32(min_log_level, 0, "#0 write logs at or above this level, 0-4 (debug|info|warning|error|fatal)");
```


- Specify the minimum level of logs to be printed, which can be used to disable low-level logs, the default is 0, and all levels of logs are printed.



### max_log_size
```cpp
DEF_int32(max_log_size, 4096, "#0 max size of a single log");
```

- Specify the maximum size of a single log, the default is 4k. A log will be truncated if its size is larger than this value.
- This value cannot exceed half of **max_log_buffer_size**.



### max_log_file_size


```cpp
DEF_int64(max_log_file_size, 256 << 20, "#0 max size of log file, default: 256MB");
```


- Specify the maximum size of a log file. The default is 256M. If this size is exceeded, a new log file will be generated, and the old log file will be renamed.



### max_log_file_num


```cpp
DEF_uint32(max_log_file_num, 8, "#0 max number of log files");
```


- Specify the maximum number of log files. The default is 8. If this value is exceeded, old log files will be deleted.



### max_log_buffer_size


```cpp
DEF_uint32(max_log_buffer_size, 32 << 20, "#0 max size of log buffer, default: 32MB");
```


- Specify the maximum size of the log cache. The default is 32M. If this value is exceeded, about half of the logs will be lost.



### log_flush_ms


```cpp
DEF_uint32(log_flush_ms, 128, "#0 flush the log buffer every n ms");
```


- The time interval for the background thread to flush the log cache to the file, in milliseconds.



### cout


```cpp
DEF_bool(cout, false, "#0 also logging to terminal");
```


- Terminal log switch, the default is false. If true, logs will also be printed to the terminal.



### also_log_to_local
```cpp
DEF_bool(also_log_to_local, false, "#0 if true, also log to local file when write-cb is set");
```

- If the value is true, also write logs to a local file when a write_cb has been set.






## Log file




### Log Organization


co/log will record all levels of logs in the same file. By default, the program name is used as the log file name. For example, the log file of process xx is xx.log. When the log file reaches the maximum size (FLG_max_log_file_size), co/log will rename the log file and generate a new file. The log directory may contain the following files:


```bash
xx.log  xx_1.log  xx_2.log  xx_3.log
```


`xx.log` is always the latest log file. For other files, the smaller the number in the file name, the newer the log. When the number in the file name reaches the maximum number of files (FLG_max_log_file_num), co/log will delete the file. 


`fatal` level logs will be additionally recorded in the `xx.fatal` file, **co/log will not rename or delete the fatal log file**.




### Log format


```bash
I0514 11:15:30.123 1045 test/xx.cc:11] hello world
D0514 11:15:30.123 1045 test/xx.cc:12] hello world
W0514 11:15:30.123 1045 test/xx.cc:13] hello world
E0514 11:15:30.123 1045 test/xx.cc:14] hello world
F0514 11:15:30.123 1045 test/xx.cc:15] hello world
```


- In the above example, each line corresponds to one log.
- The first letter of each log is the log level, `I` means info, `D` means debug, `W` means warning, `E` means error, and `F` means fatal.
- After the level is the time, from month to milliseconds. The year is not printed. The time of the log is not the time when it is generated, but the time when it is written to the cache, as we must ensure that the logs in the log file are strictly sorted by time.
- After the time is the thread id, 1045 above is the thread id.
- The thread id is followed by the file and line number of the log code.
- After the line number is `] `, that is, a space after `]`.
- following the `] `，is the log content by the user.





### View logs


On linux or mac, `grep`, `tail` and other commands can be used to view the logs.


```bash
grep ^E xx.log
tail -F xx.log
tail -F xx.log | grep ^E
```


- The first line uses `grep` to filter out the error logs in the file, `^E` means starts with the letter `E`.
- The second line uses the `tail -F` command to dynamically track the log file, here we should use the uppercase `F`, because xx.log may be renamed, and then generate a new xx.log file, `-F` make sure to follow the latest file by the name.
- In line 3, use `tail -F` in conjunction with `grep` to dynamically track the error logs in the log file.







## Build and run the co/log test program


```bash
xmake -b log                    # build log or log.exe
xmake r log                     # run log or log.exe
xmake r log -cout               # also log to terminal
xmake r log -min_log_level=1    # 0-4: debug,info,warning,error,fatal
xmake r log -perf               # performance test
```


- Run `xmake -b log` in the co root directory to compile [test/log.cc](https://github.com/idealvin/co/blob/master/test/log.cc), and a binary program named log or log.exe will be generated.
