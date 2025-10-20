# MyLog 日志系统使用说明

## 简介

MyLog 是一个基于C++实现的高性能日志系统，参考了Log4J的设计理念。它支持多级别的日志记录、灵活的输出方式（控制台/文件）以及通过YAML配置文件进行配置管理。

## 功能特性

1. **多级别日志**：支持DEBUG、INFO、WARN、ERROR、FATAL五个日志级别
2. **灵活输出**：支持控制台和文件两种输出方式
3. **自定义格式**：通过格式化字符串自定义日志输出格式
4. **YAML配置**：通过YAML文件配置日志系统
5. **线程安全**：支持多线程环境下的日志记录
6. **动态配置**：支持运行时动态修改配置

## 安装依赖

日志系统使用 [yaml-cpp](https://github.com/jbeder/yaml-cpp) 解析配置，需要提前安装：

```bash
git clone https://github.com/jbeder/yaml-cpp.git
cd yaml-cpp
mkdir build
cd build
cmake ..
make

# 全局安装
sudo make install
```

## 快速开始

### 1. 基本使用

```cpp
#include <iostream>
#include "log.h"

int main() {
    // 获取名为"test_log"的日志记录器
    mylog::Logger::ptr test_log = MYLOG_LOG_NAME("test_log");
    
    // 记录不同级别的日志
    MYLOG_LOG_DEBUG(test_log) << "DEBUG log";
    MYLOG_LOG_INFO(test_log) << "INFO log";
    MYLOG_LOG_WARN(test_log) << "WARN log";
    MYLOG_LOG_ERROR(test_log) << "ERROR log";
    MYLOG_LOG_FATAL(test_log) << "FATAL log";
    
    return 0;
}
```

### 2. 编译和运行

```bash
# 编译（假设相关头文件和库文件已在正确位置）
g++ -o test test.cpp -L. -L/usr/local/lib -lmylog -lyaml-cpp -I. -Wl,-rpath,./

# 运行
./test
```

## 配置说明

### YAML配置文件格式

在 `bin/conf/log.yml` 中配置日志系统：

```yaml
logs:
    - name: root
      level: info
      formatter: "%d%T%m%n"
      appenders:
          - type: FileLogAppender
            file: root_log.txt
          - type: StdoutLogAppender
    - name: system
      level: debug
      formatter: "%d%T%m%n"
      appenders:
          - type: FileLogAppender
            file: system_log.txt
            formatter: "%d%T[%p]%T%m%n"
          - type: StdoutLogAppender
```

### 配置项说明

- **name**: 日志记录器名称
- **level**: 日志级别（debug/info/warn/error/fatal）
- **formatter**: 日志格式化字符串
- **appenders**: 日志输出器列表
  - **type**: 输出器类型（FileLogAppender/StdoutLogAppender）
  - **file**: 文件路径（仅FileLogAppender需要）
  - **formatter**: 特定于该输出器的格式化字符串（可选）

## 日志格式化

### 格式化符号

- `%d`: 时间戳，可指定格式如 `%d{%Y-%m-%d %H:%M:%S}`
- `%p`: 日志级别
- `%t`: 线程ID
- `%F`: 协程ID
- `%c`: 日志记录器名称
- `%f`: 文件名
- `%l`: 行号
- `%m`: 日志消息内容
- `%n`: 换行符
- `%T`: 制表符(Tab)

### 示例格式

```
%d%T%p%T%t%m%n
# 输出示例：2024-02-25 19:34:51     [INFO]  0       Hello world!

%d{%Y-%m-%d %H:%M:%S}%T[%p]%T[%c]%T%m%n
# 输出示例：2024-02-25 19:34:51     [INFO]  [test_log]      Hello world!
```

## API参考

### 日志记录宏

#### 流式日志记录
```cpp
MYLOG_LOG_DEBUG(logger) << "Debug message";
MYLOG_LOG_INFO(logger) << "Info message";
MYLOG_LOG_WARN(logger) << "Warning message";
MYLOG_LOG_ERROR(logger) << "Error message";
MYLOG_LOG_FATAL(logger) << "Fatal message";
```

#### 格式化日志记录
```cpp
MYLOG_LOG_FMT_DEBUG(logger, "Debug message: %s", msg);
MYLOG_LOG_FMT_INFO(logger, "Info message: %d", value);
MYLOG_LOG_FMT_WARN(logger, "Warning message: %f", flost_val);
MYLOG_LOG_FMT_ERROR(logger, "Error message: %s", error_msg);
MYLOG_LOG_FMT_FATAL(logger, "Fatal message: %s", fatal_msg);
```

### 获取日志记录器

```cpp
// 获取根日志记录器
mylog::Logger::ptr root = MYLOG_LOG_ROOT();

// 获取指定名称的日志记录器
mylog::Logger::ptr logger = MYLOG_LOG_NAME("logger_name");
```

## 高级用法

### 自定义日志输出器

可以通过继承 [LogAppender](file:///home/william/projects/serverproject/mylog/mylog/src/log.h#L163-L178) 类实现自定义的日志输出器：

```cpp
class CustomLogAppender : public mylog::LogAppender {
public:
    typedef std::shared_ptr<CustomLogAppender> ptr;
    
    void log(mylog::Logger::ptr logger, mylog::LogLevel::Level level, 
             mylog::LogEvent::ptr event) override {
        // 自定义日志输出逻辑
    }
    
    std::string toYamlString() override {
        // 返回YAML格式的配置字符串
    }
};
```

### 自定义格式化项

可以通过继承 [LogFormatter::FormatItem](file:///home/william/projects/serverproject/mylog/mylog/src/log.h#L142-L149) 类实现自定义的格式化项：

```cpp
class CustomFormatItem : public mylog::LogFormatter::FormatItem {
public:
    CustomFormatItem(const std::string& str = "") {}
    
    virtual void format(std::ostream& os, mylog::Logger::ptr logger, 
                        mylog::LogLevel::Level level, 
                        mylog::LogEvent::ptr event) override {
        // 自定义格式化逻辑
    }
};
```

## 常见问题

### 1. 如何在项目中集成MyLog？

确保已安装yaml-cpp依赖，然后将以下文件复制到您的项目中：
- libmylog.so（动态库文件）
- log.h
- singleton.h
- util.h

在代码中包含头文件并使用相应的API即可。

### 2. 日志没有输出怎么办？

检查以下几点：
1. 日志级别设置是否正确
2. 是否添加了日志输出器（Appender）
3. 配置文件路径是否正确
4. 文件输出器是否有写入权限

### 3. 如何动态修改日志配置？

MyLog支持运行时动态修改配置。修改 `bin/conf/log.yml` 文件后，系统会自动应用新的配置。

## 性能优化建议

1. 合理设置日志级别，避免输出过多调试信息到生产环境
2. 在高性能要求的场景中，考虑使用异步日志输出
3. 合理配置日志文件大小和滚动策略，避免单个日志文件过大
4. 在多线程环境中，注意日志记录的线程安全性