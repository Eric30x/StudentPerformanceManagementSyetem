 学生成绩管理系统（C++ HTTP 服务器）

> 基于原生 Windows Socket（Winsock2）从零构建的 HTTP 服务器，用于学生成绩的增删查改管理。

[![Language](https://img.shields.io/badge/language-C%2B%2B-blue)](https://isocpp.org/)
[![Platform](https://img.shields.io/badge/platform-Windows-lightgrey)](https://www.microsoft.com/windows/)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

## 项目简介

本项目是一个**练手/学习项目**，用纯 C++ + Winsock2 从 Socket 层手动构建了一个 HTTP 服务器，不依赖任何第三方 HTTP 库，实现了完整的 RESTful API，配合前端页面完成学生成绩的管理。

**核心特点：**

- 零依赖 HTTP：手工解析原始 HTTP 请求（Method、Path、Headers、Body）
- 原生 Windows 网络编程：基于 Winsock2 API
- 文件系统持久化：数据存储在 `data/students.txt`
- 支持 CORS 跨域，前后端分离

---

## 项目结构

```
├── main.cpp           # 主程序：HTTP 服务器 + 路由分发 + 请求处理
├── server.h           # 服务器类声明
├── database.h         # 数据库类声明
├── database.cpp       # 数据持久化（文件读写）
├── student.h          # 学生类声明
├── student.cpp        # 学生模型（学号、姓名、成绩、JSON 序列化）
├── index.html         # 前端管理页面（单页应用）
├── main.py            # Python 脚本存根
└── data/
    └── students.txt   # 学生数据（运行时自动生成）
```

---

## 快速开始

### 环境要求

- **操作系统**：Windows
- **编译器**：支持 C++11 及以上（MSVC 或 MinGW）
- **依赖**：`ws2_32.lib`（Windows 系统自带，无需额外安装）

### 编译 & 运行

```bash
# MSVC 编译
cl main.cpp student.cpp database.cpp /Fe:server.exe ws2_32.lib

# MinGW / g++ 编译
g++ main.cpp student.cpp database.cpp -o server.exe -lws2_32

# 启动服务器
server.exe
```

启动成功后会打印：

```
=================================
服务器启动成功！
访问地址: http://localhost:8080
=================================
```

在浏览器中打开 **http://localhost:8080** 即可使用。

---

## API 文档

### 接口一览

| 方法 | 路径 | 说明 |
|------|------|------|
| `GET` | `/` 或 `/index.html` | 返回前端管理页面 |
| `GET` | `/api/students` | 获取所有学生列表（JSON） |
| `POST` | `/api/students` | 添加新学生 |
| `DELETE` | `/api/students/{id}` | 删除指定学号的学生 |

### GET /api/students

获取全部学生数据。

**响应示例：**

```json
[
  {
    "id": "2024001",
    "name": "张三",
    "scores": [90, 85, 95],
    "total": 270,
    "average": 90.0
  },
  {
    "id": "2024002",
    "name": "李四",
    "scores": [88, 92, 87],
    "total": 267,
    "average": 89.0
  }
]
```

### POST /api/students

添加一名新学生。请求体格式为 URL-encoded form data。

**请求示例：**

```
POST /api/students HTTP/1.1
Content-Type: application/x-www-form-urlencoded

id=2024003&name=王五&score1=78&score2=82&score3=91
```

**响应示例：**

```json
{ "success": true }
```

### DELETE /api/students/{id}

根据学号删除学生。

**请求示例：**

```
DELETE /api/students/2024001 HTTP/1.1
```

**响应示例：**

```json
{ "success": true }
```

---

## 技术要点

### 1. 手工 HTTP 解析

不依赖任何 HTTP 库，直接从 TCP Socket 读取原始字节流，手工解析：

- 请求行（Method + Path + Version）
- 请求头
- 请求体（Body）

```cpp
// 示例：判断请求类型
if (request.find("GET /api/students") != std::string::npos) { ... }
if (request.find("POST /api/students") != std::string::npos) { ... }
```

### 2. Winsock2 网络编程五步曲

```cpp
WSAStartup()    // 1. 初始化 Winsock
socket()        // 2. 创建 Socket
bind()          // 3. 绑定地址和端口
listen()        // 4. 开始监听
accept()        // 5. 接受客户端连接
recv() / send() //    收发数据
closesocket()   //    关闭连接
WSACleanup()    //    清理
```

### 3. 阻塞式单线程模型

当前采用简单的一次处理一个请求的阻塞模型：

```
while (true) {
    client = accept();     // 阻塞等待连接
    recv(client, ...);     // 接收请求
    response = handle();   // 处理请求
    send(client, ...);     // 返回响应
    close(client);         // 关闭连接
}
```

>  **注意**：此为学习用途的简化实现，生产环境应使用多线程或 IO 多路复用。

### 4. 数据持久化

学生数据以空格分隔的文本格式存储在文件中：

```
2024001 张三 90 85 95
2024002 李四 88 92 87
```

每次 API 操作前加载数据（`loadData()`），修改后写回文件（`saveData()`）。

---

## 待改进

- [ ] **多线程支持** — 当前为单线程阻塞模型，无法并发处理请求
- [ ] **JSON 库替换** — 当前为手工字符串拼接，应使用 nlohmann/json 等成熟库
- [ ] **URL 解码** — 处理中文姓名等特殊字符的编解码
- [ ] **错误处理** — 增强边界条件和异常情况的处理
- [ ] **跨平台支持** — 使用 POSIX socket 适配 Linux/macOS
- [ ] **单元测试** — 添加测试用例

---

## 📄 License

MIT © [Eric30x](https://github.com/Eric30x)

