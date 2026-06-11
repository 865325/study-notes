### 内存管理：对象树 (Object Tree) 机制

在 C++ 中，你可能习惯了 `new` 必须对应 `delete`，但在 Qt 中情况有所不同：

- **父子关系**：在构造函数 `Widget(QWidget *parent)` 中，`parent` 参数非常关键。
- **自动析构**：当一个对象被创建时指定了 `parent`（如 `new QProcess(this)`），它会被添加到父对象的子对象列表中。**当父对象（Widget）被销毁时，它会自动调用所有子对象的析构函数。**
- **UI 指针**：`ui` 成员通过 `delete ui;` 手动释放，因为它不是一个继承自 `QObject` 的子部件，而是由 `Ui::Widget` 生成的辅助类。



### 信号与槽 (Signals & Slots)

这是 Qt 替代“回调函数”的方案，用于对象间的通信

`SIGNAL` 和 `SLOT` 是**旧版宏语法**（Qt 4）。它不进行编译时类型检查，容易写错。

**新版函数指针语法**（Qt 5+）： `connect(发送者, &类名::信号, 接收者, &类名::槽函数);` 这种方式在编译时就能发现函数名写错或参数不匹配的问题。

```c++
connect(ui->cmdLineEdit, &QLineEdit::returnPressed, this, &Widget::handleCancelClicked);
```

其中&类名::槽函数可以用Lambda 表达式



###  Qt 原生定时器 (Timer Event) 核心笔记

在 Qt 中，`startTimer` 和 `timerEvent` 是最底层的定时器实现方案，适用于对性能要求极高或需要管理大量定时器的场景。

1. 开启定时器：`startTimer()`

- 函数原型：`int QObject::startTimer(int interval, Qt::TimerType timerType = Qt::CoarseTimer)`。
- 返回值：返回一个唯一的整数 ID（`myTimerId`），用于标识该定时器。
- 参数：`interval` 单位为毫秒（ms）。
- 特点：只要不调用 `killTimer`，它会每隔设定的时间触发一次 `timerEvent`。

2. 事件处理：`timerEvent()`

- 机制：这是一个虚函数（Virtual Function），属于事件处理器（Event Handler）。

- 重写逻辑：

	- 多定时器区分：如果一个类里启动了多个定时器，需要通过 `event->timerId()` 来判断当前触发的是哪一个，避免逻辑混淆。

		C++

		```
		if (event->timerId() == myTimerId) { // 执行逻辑 }
		```

3. 停止定时器：`killTimer()`

- 函数原型：`void QObject::killTimer(int id)`。
- 必要性：停止定时器必须传入开启时返回的 ID。一旦停止，该 ID 变为失效，对应的 `timerEvent` 不再触发。

### QTimer 类核心使用笔记

1. 周期性定时器 (Object-based)

通过实例化 `QTimer` 对象，可以更方便地结合 信号与槽 机制。

- 初始化与内存管理：
	- 通常在构造函数中 `new QTimer(this)`，传入 `this` 作为父对象，这样当窗口析构时，定时器会被自动释放。
- 连接信号：
	- 使用 `connect(timer, &QTimer::timeout, this, &槽函数);`。每当时间到达，就会触发 `timeout` 信号。
- 控制方法：
	- `start(int msec)`：开启定时器，若定时器已在运行，调用此函数会重启它。
	- `stop()`：停止定时器。可以通过 `isActive()` 检查定时器当前是否正在运行。

2. 静态单次定时器 (QTimer::singleShot)

这是一种“开完即焚”的便捷方法，不需要创建 `QTimer` 对象。

- 基本语法：`QTimer::singleShot(毫秒, 接收者, 槽函数);`
- 应用场景：
	- 延迟执行：如你代码中点击按钮后 1 秒切换一次图片。
	- 非阻塞等待：相比于 `sleep()`，它不会卡死 UI 界面，任务在后台计时，主线程继续响应用户操作。
- 灵活性：它支持 Lambda 表达式，可以临时处理简单的逻辑而不需要在 `.h` 中声明新槽函数。

### 文件操作笔记 (QFile & QFileDialog)

在 Qt 中，文件操作通常遵循“**选择文件路径 -> 打开文件 -> 读写数据 -> 关闭文件**”的流程。

1. 文件对话框 (`QFileDialog`)

- `getOpenFileName`：打开现有文件对话框。返回用户选中的完整路径。如果用户取消，返回空字符串。
- `getSaveFileName`：保存文件对话框。
	- 技巧：代码中使用 `this->windowTitle()` 作为初始路径，可以实现“默认保存为当前打开文件名”的效果。

2. 读写流程 (`QFile`)

- 打开模式 (`QIODevice::OpenMode`)：
	- `ReadOnly`：只读模式。
	- `WriteOnly`：只写模式（会覆盖原内容）。
	- `Append`：追加模式。
- 数据转换：
	- 读取：`file.readAll()` 返回 `QByteArray`，使用 `QString(ba)` 将原始字节流转为字符串显示到 `QTextEdit`。
	- 写入：`toPlainText()` 获取编辑器内容，需调用 `toUtf8()` 转为字节流后才能写入文件。

------

###  事件处理笔记 (Event System)

Qt 的事件系统允许你通过重写（Override）基类的虚函数来拦截并处理特定的用户操作。

1. 键盘事件 (`keyPressEvent`)

- 修饰键判断：使用 `modifiers()` 判断是否按下了 `Ctrl`, `Shift` 或 `Alt`。
	- 例如：`Qt::ControlModifier` 代表 Ctrl 键。
- 按键判断：使用 `key()` 判断具体字母或功能键，如 `Qt::Key_S`。

`````
//  优雅地设置快捷键
// 方式 A：使用标准预设（推荐，跨平台自动适配）
ui->saveAction->setShortcut(QKeySequence::Save); 

// 方式 B：手动指定组合键
// ui->saveAction->setShortcut(QKeySequence("Ctrl+S"));
`````

2. 鼠标事件 (`mousePressEvent`)

- 坐标获取：`m->pos()` 返回鼠标点击相对于当前窗口的局部坐标 (`QPoint`)。
- 按键区分：
	- `Qt::LeftButton`：左键。
	- `Qt::RightButton`：右键。
- 注意：`QMouseEvent` 只有在鼠标按下的一瞬间触发，如果需要持续跟踪移动，则需重写 `mouseMoveEvent`。

### TCP 笔记

### 1. 通信流程与 Linux C 网络编程的映射

Qt 的网络模块本质上是对操作系统底层 Socket API（如 Linux 的 `sys/socket.h` 或 Windows 的 `winsock2.h`）的异步、面向对象封装。

| **通信步骤**   | Linux 原生 C API                                  | Qt 网络 API                                      | **差异与说明**                                               |
| -------------- | ------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| **创建服务端** | `socket()`                                        | `new QTcpServer`                                 | Qt 采用面向对象方式实例化。                                  |
| **绑定与监听** | `bind()` + `listen()`                             | `server->listen()`                               | Qt 将地址绑定和端口监听合并为一个函数。                      |
| **接受连接**   | `accept()` (默认阻塞)                             | `newConnection` 信号 + `nextPendingConnection()` | Qt 基于事件驱动。`accept()` 会卡住主线程，而 Qt 采用信号槽机制，收到连接请求才触发，完全非阻塞。 |
| **创建客户端** | `socket()`                                        | `new QTcpSocket`                                 | 实例化客户端套接字对象。                                     |
| **发起连接**   | `connect()` (默认阻塞)                            | `socket->connectToHost()`                        | 原生 `connect` 可能卡顿，Qt 立即返回，连上后触发 `connected` 信号。 |
| **发送数据**   | `send()` / `write()`                              | `socket->write()`                                | Qt 会先将数据存入发送缓冲区，由底层自动完成分片发送。        |
| **接收数据**   | `recv()` / `read()` (通常需配合 `epoll`/`select`) | `readyRead` 信号 + `socket->readAll()`           | 原生需手动轮询文件描述符可读状态，Qt 自动监控缓冲区并发出信号。 |

------

2. 核心伪代码与函数解析

服务端 (Server) 伪代码

```C++
// 1. 创建服务器对象
QTcpServer *server = new QTcpServer(this);

// 2. 启动监听
server->listen(QHostAddress::AnyIPv4, 8000);

// 3. 处理新连接请求
connect(server, &QTcpServer::newConnection, this, [=](){
    // 4. 获取负责与该客户端通信的 socket
    QTcpSocket *clientSocket = server->nextPendingConnection();

    // 5. 绑定读取数据逻辑
    connect(clientSocket, &QTcpSocket::readyRead, this, [=](){
        QByteArray data = clientSocket->readAll();
        // 处理 data...
});

        // 6. 处理断开逻辑
        connect(clientSocket, &QTcpSocket::disconnected, clientSocket, &QTcpSocket::deleteLater);
});
```

**关键函数用法：**

- `listen(const QHostAddress &address, quint16 port)`
	- **参数**：`QHostAddress::AnyIPv4` 代表监听本机所有的 IPv4 网卡地址（相当于 `0.0.0.0`）。`port` 为端口号，传 `0` 表示由系统随机分配可用端口。
- `nextPendingConnection()`
	- **用法**：只能在 `newConnection` 槽函数中调用。它返回一个指向新创建的 `QTcpSocket` 的指针，专门用于与刚刚连入的那个客户端通信。

客户端 (Client) 伪代码

```c++
// 1. 创建套接字
QTcpSocket *socket = new QTcpSocket(this);

// 2. 绑定状态与数据接收信号 (提前绑定)
connect(socket, &QTcpSocket::connected, this, [](){ /* 连上啦 */ });
connect(socket, &QTcpSocket::readyRead, this, [=](){
    QByteArray data = socket->readAll();
    // 处理 data...
});

// 3. 发起连接
socket->connectToHost(QHostAddress("192.168.1.100"), 8000);

// 4. 发送数据 (在需要的时机调用)
socket->write("Hello Server");
```

**关键函数用法：**

- `connectToHost(const QHostAddress &address, quint16 port)`
	- 参数：目标服务器的 IP 和端口。支持传入域名（如 `"www.google.com"`），内部会自动进行 DNS 解析。
- `write(const QByteArray &data)`
	- 用法：将数据写入发送缓冲区。最大支持单次写入 2GB 数据，底层会自动分包。