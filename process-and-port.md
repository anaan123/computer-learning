一、进程与端口基础概念

1. 进程（Process）

* * *

**进程 = 正在运行的程序实例**

例如电脑运行的程序：

* Chrome
  
* VSCode
  
* Node
  
* Go 后端
  
* MySQL
  

当程序运行时，操作系统会给它分配一个 **PID（Process ID）**。

示例：

    main.exe      PID 13284  
    chrome.exe    PID 22111  
    node.exe      PID 34567

结论：

    进程 = 程序运行后的实例  
    PID = 进程的唯一编号

* * *

2. **端口（Port）**

* * *

端口是 **网络通信入口**。

可以用一个类比理解：

    电脑 = 一栋楼  
    端口 = 房间号  
    进程 = 房间里的工作人员

常见服务端口：

| 端口  | 用途  |
| --- | --- |
| 8080 | 后端 API |
| 5173 | Vite 前端 |
| 3000 | React dev server |
| 3306 | MySQL |
| 6379 | Redis |

例如访问：

    http://localhost:8080

系统会：

    找到监听 8080 端口的进程  
    把请求交给这个进程处理

结论：

    端口 = 网络入口  
    进程 = 处理请求的程序

* * *

二、进程与端口的关系

一个进程可以监听一个端口。

例如：

    Go 后端  
    PID = 13284  
    监听端口 = 8080

使用命令查看：

    netstat -ano | findstr :8080

返回：

    0.0.0.0:8080   LISTENING   13284

表示：

    8080端口 -> PID 13284

其中：

| 状态  | 含义  |
| --- | --- |
| LISTENING | 进程正在监听端口（服务器） |
| ESTABLISHED | 已建立连接 |
| TIME_WAIT | 连接关闭等待 |

* * *

三、Go 后端端口占用排查流程

当运行：

    go run main.go

出现错误：

    listen tcp 0.0.0.0:8080: bind  
    Only one usage of each socket address is normally permitted

说明：

    8080 端口已经被其他进程占用

* * *

标准排查流程

### 1 查看端口占用

    netstat -ano | findstr :8080

返回示例：

    TCP   0.0.0.0:8080   0.0.0.0:0   LISTENING   13284

说明：

    PID 13284 正在监听 8080 端口

* * *

### 2 确认进程

建议先确认进程：

    tasklist | findstr 13284

例如：

    main.exe   13284

说明是 Go 后端程序。

* * *

### 3 结束进程（管理员权限）

    taskkill /PID 13284 /F

如果出现：

    拒绝访问

需要：

    使用管理员 PowerShell

* * *

### 4 重新启动服务

    go run main.go

* * *

四、为什么不会误杀前端

如果执行：

    netstat -ano | findstr :8080

只会找到：

    使用 8080 端口的进程

例如：

    Go 后端 → 8080  
    Vite 前端 → 5173  
    MySQL → 3306

因为端口不同：

    5173 ≠ 8080

所以：

    杀 8080 的 PID  
    不会影响 5173 的前端

* * *

五、什么时候可能误杀

只有一种情况：

如果前端也运行在 **8080 端口**。

例如：

    React dev server  
    http://localhost:8080

这时：

    netstat -ano | findstr :8080

可能返回：

    node.exe  PID 20000

如果执行：

    taskkill /PID 20000 /F

就会把 **前端 dev server** 杀掉。

* * *

六、安全操作建议（开发者习惯）

在结束进程前建议先确认：

    tasklist | findstr PID

例如：

    tasklist | findstr 13284

确认进程类型：

    main.exe  
    node.exe  
    mysql.exe

再执行：

    taskkill /PID xxxx /F

* * *

七、开发时最常见端口

| 端口  | 用途  |
| --- | --- |
| 3000 | React |
| 5173 | Vite |
| 8080 | 后端 API |
| 3306 | MySQL |
| 6379 | Redis |
| 9200 | Elasticsearch |

* * *

八、核心总结

    进程 = 运行中的程序  
    端口 = 网络通信入口  
    PID = 进程唯一编号  
    
    一个端口只能被一个进程监听  
    端口冲突时，需要找到占用端口的 PID 并结束进程
    

常用排查命令：

    netstat -ano | findstr :端口  
    tasklist | findstr PID  
    taskkill /PID PID /F
