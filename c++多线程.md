## std::mutex (互斥锁)

`std::mutex` 用于保护共享资源，确保同一时刻只有一个线程可以访问该资源。

头文件：`<mutex>`

**常用函数**：

- `lock()`：锁定互斥量。如果锁已被占用，线程将阻塞直至获取锁。
- `unlock()`：释放互斥量。
- `try_lock()`：尝试锁定。若获取成功返回 `true`，失败（锁已被占用）返回 `false`，不阻塞。

**推荐用法 (RAII 风格)**： 避免直接调用 `lock()` 和 `unlock()`，推荐使用 `std::lock_guard` 或 `std::unique_lock` 来管理锁的生命周期，防止异常发生导致死锁。

```c++
std::mutex mtx;
void worker() {
    // 自动加锁，超出作用域自动释放
    std::lock_guard(std::mutex) mtx;
}
```

### std::condition_variable (条件变量)

`std::condition_variable` 允许线程挂起，直到另一个线程修改了共享变量并通知条件变量

**头文件**：`<condition_variable>`

**常用函数**：

- `wait(unique_lock<mutex>& lock, Predicate pred)`：
	- `lock`：必须是 `unique_lock`。
	- `pred`（可选）：谓词（通常是 Lambda），返回 `false` 时继续阻塞，返回 `true` 时停止阻塞。
- `notify_one()`：唤醒一个正在等待的线程。
- `notify_all()`：唤醒所有正在等待的线程。

```c++
std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void wait_thread() {
    std::unique_lock<std::mutex> lck(mtx);
    // 如果 ready 为 false 则释放锁并阻塞；被唤醒且 ready 为 true 时获取锁并继续
    cv.wait(lck, []{ return false; });
}

void signal_thread() {
    {
        std::lock_guard<std::mutex> lck(mtx);
        ready = true;
    }
    cv.notify_one();
}
```

**注意事项**：

1. **虚假唤醒 (Spurious Wakeup)**：线程可能在没有被通知的情况下唤醒。因此，**必须**在 `wait` 中使用循环或传入谓词（Predicate）来反复检查条件。
2. **必须配合 unique_lock**：条件变量不能配合 `lock_guard` 使用，因为它在等待时需要能够灵活地进行解锁和加锁操作。
3. **先加锁后修改**：在调用 `notify` 之前，通常需要修改条件变量关联的状态，且该修改应在互斥锁的保护下进行。

### std::lock_guard 和 std::lock_unique 差别

`lock_guard` 是最简单的锁管理器。它在构造时加锁，在析构（作用域结束）时自动解锁。

- **特点**：轻量、简单、性能高。
- **局限**：一旦锁定，在生命周期内无法手动解锁，也无法中途重新上锁。

`unique_lock` 是 `lock_guard` 的增强版，提供了更丰富的功能。

- **特点**：
	- **手动控制**：支持手动调用 `lock()` 和 `unlock()`。
	- **延迟锁定**：可以创建时不立刻加锁（使用 `std::defer_lock`）。
	- **配合条件变量**：`std::condition_variable` **必须**配合 `unique_lock` 使用。
	- **转移所有权**：支持 `std::move` 转移锁的所有权。

为什么必须用 mutex 初始化？

RAII（资源获取即初始化）的核心思想就是：在对象构造时获取资源，在析构时释放资源。

- 如果你不传入 `mutex`，这些工具就不知道该去锁哪段内存，也不知道该在退出作用域时解锁谁。

### std::thread

在 C++11 之后，标准库提供了 `std::thread` 来实现跨平台的线程管理。

- **头文件**：`#include <thread>`

常用函数及成员方法

| **函数 / 方法**                   | **功能描述**                                | **参数与返回值**                                             |
| --------------------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| **构造函数** `thread(f, args...)` | 创建一个线程并立即开始执行任务 `f`          | `f`: 可调用对象（函数、Lambda、成员函数）；`args`: 传递给函数的参数。 |
| `join()`                          | **阻塞**当前线程，直到子线程执行完毕        | 无返回值。调用后，子线程资源被回收，`thread` 对象变为不可汇合状态。 |
| `detach()`                        | **分离**子线程，让其在后台独立运行          | 子线程生命周期由系统管理。调用后，`thread` 对象不再关联该线程。 |
| `joinable()`                      | 检查线程是否可以执行 `join()` 或 `detach()` | 返回 `bool`。已执行过 join/detach 或默认构造的线程返回 `false`。 |
| `get_id()`                        | 获取当前线程的唯一标识符                    | 返回 `std::thread::id` 对象。                                |
| `native_handle()`                 | 获取底层操作系统的线程句柄                  | 返回与平台相关的句柄（如 Linux 下的 `pthread_t`）。          |

 核心代码示例

```c++
#include <iostream>
#include <thread> // 必须包含头文件

void task(int n, std::string msg) {
    std::cout << "线程 ID: " << std::this_thread::get_id() << " 接收到: " << n << ", " << msg << std::endl;
}

int main() {
    int num = 100;
    std::string text = "Hello Thread";

    // 1. 创建并启动线程
    // 注意：实参会以副本形式拷贝到线程空间中
    std::thread t1(task, num, text);

    // 2. 检查并回收线程
    if (t1.joinable()) {
        t1.join(); // 主线程在此等待 t1 结束
    }

    // 3. 使用 Lambda 表达式创建线程
    std::thread t2([]() {
        std::cout << "匿名线程正在运行..." << std::endl;
    });
    t2.detach(); // 分离线程，主线程不再负责等待它

    return 0;
}
```

------

命名空间 `std::this_thread`

标准库提供了一组函数，用于在**当前线程内部**管理自身状态：

- `std::this_thread::get_id()`: 获取当前线程 ID。
- `std::this_thread::sleep_for(duration)`: 让当前线程休眠一段时间（如 `std::chrono::seconds(1)`）。
- `std::this_thread::sleep_until(time_point)`: 休眠直到某个特定的时间点。
- `std::this_thread::yield()`: 暂时放弃当前线程的 CPU 执行权，允许其他线程运行。

------

重要注意事项（避坑指南）

1. **资源回收（必须二选一）**： 一个 `std::thread` 对象在析构前，必须确保调用了 `join()` 或 `detach()`。否则，程序会直接调用 `std::terminate` 导致崩溃。
2. **引用传递参数**： `std::thread` 的构造函数默认是**拷贝参数**。如果需要传递引用，必须使用 `std::ref` 包裹： `std::thread t(task, std::ref(my_obj));`
3. **局部变量失效**： 如果使用 `detach()`，必须确保子线程访问的局部变量（或引用）在线程运行期间一直有效，否则会导致访问非法内存。
4. **异常安全**： 如果主流程抛出异常导致 `join()` 没被执行，程序仍会崩溃。建议使用 `lock_guard` 风格的 RAII 工具（如 C++20 的 `std::jthread`）来管理线程。