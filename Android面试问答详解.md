# Android面试问答详解

## 一、Java 基础

### 1. == 和 equals 的区别？

**回答：**
- `==` 比较的是内存地址（引用是否指向同一对象）
- `equals` 比较的是内容（逻辑上是否相等）

**代码示例：**
```java
String s1 = new String("abc");
String s2 = new String("abc");
s1 == s2       // false，不同对象
s1.equals(s2)  // true，内容相同

String s3 = "abc";
String s4 = "abc";
s3 == s4       // true，常量池复用
```

**追问：基本类型呢？**
基本类型没有 equals 方法，只能用 `==` 比较值。

---

### 2. 为什么重写 equals 必须重写 hashCode？

**回答：**
因为 HashMap 等集合先用 hashCode 定位桶，再用 equals 判断是否相等。
如果两个对象 equals 相等但 hashCode 不同，会被放到不同的桶里，导致查找失败。

**反例：**
```java
class User {
    String name;
    @Override
    public boolean equals(Object o) {
        return this.name.equals(((User)o).name);
    }
    // 没重写 hashCode
}

Map<User, String> map = new HashMap<>();
User u1 = new User("张三");
map.put(u1, "value");
User u2 = new User("张三");
map.get(u2);  // null！因为 hashCode 不同，定位到不同桶
```

**正确写法：**
```java
@Override
public int hashCode() {
    return Objects.hash(name);
}
```

---

### 3. String 为什么不可变？有什么好处？

**回答：**
String 内部用 `final char[]` 存储，且没有提供修改方法。

**好处：**
1. 线程安全，无需同步
2. 可以缓存 hashCode，HashMap 性能更好
3. 字符串常量池复用，节省内存

**追问：String 为什么容易造成内存问题？**
频繁拼接会创建大量临时对象，应该用 StringBuilder。
```java
// 差
String s = "";
for (int i = 0; i < 10000; i++) {
    s += i;  // 每次创建新对象
}

// 好
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i);
}
```

---

### 4. String、StringBuilder、StringBuffer 区别？

| 类 | 可变性 | 线程安全 | 性能 |
|---|---|---|---|
| String | 不可变 | 安全 | 拼接慢 |
| StringBuilder | 可变 | 不安全 | 快 |
| StringBuffer | 可变 | 安全（synchronized） | 较慢 |

**使用场景：**
- 单线程拼接：StringBuilder
- 多线程拼接：StringBuffer
- 不拼接：String

---

### 5. ArrayList 和 LinkedList 区别？

| 特性 | ArrayList | LinkedList |
|---|---|---|
| 底层 | 数组 | 双向链表 |
| 随机访问 | O(1) | O(n) |
| 插入删除 | O(n) | O(1)（已定位） |
| 内存 | 连续，紧凑 | 分散，有指针开销 |

**ArrayList 扩容：**
1. 默认容量 10
2. 扩容为原来的 1.5 倍
3. Arrays.copyOf 复制到新数组

**使用场景：**
- 查询多：ArrayList
- 增删多（头尾操作）：LinkedList
- 实际开发中 90% 用 ArrayList

---

### 6. HashMap 原理？（必考）

**结构：**
JDK 1.8：数组 + 链表 + 红黑树

**put 流程：**
1. 计算 key 的 hash：`(h = key.hashCode()) ^ (h >>> 16)`
2. 定位下标：`(n - 1) & hash`
3. 如果为空，直接放入
4. 如果有冲突：
   - 链表长度 < 8：尾插法插入链表
   - 链表长度 >= 8 且数组长度 >= 64：转红黑树
5. 检查是否需要扩容

**扩容：**
- 默认容量 16，负载因子 0.75
- 元素数量 > 16 * 0.75 = 12 时扩容
- 扩容为原来的 2 倍
- 重新计算所有元素位置

**追问：为什么线程不安全？**
1. JDK 1.7 头插法扩容会形成环形链表，导致死循环
2. JDK 1.8 虽然改成尾插法，但 put 时仍可能数据覆盖

**追问：1.7 和 1.8 区别？**
| 版本 | 结构 | 插入方式 | 扩容 |
|---|---|---|---|
| 1.7 | 数组+链表 | 头插法 | 先扩容后插入 |
| 1.8 | 数组+链表+红黑树 | 尾插法 | 先插入后扩容 |

---

### 7. ConcurrentHashMap 原理？

**JDK 1.7：**
分段锁（Segment），每个 Segment 是一个小 HashMap，默认 16 个。
并发度 = Segment 数量

**JDK 1.8：**
CAS + synchronized
- 写操作：synchronized 锁住链表头节点
- 读操作：volatile 保证可见性，无锁

**追问：为什么不用 Hashtable？**
Hashtable 用一把大锁锁住整个表，并发度太低。

---

### 8. 泛型擦除是什么？

**回答：**
Java 泛型只在编译期检查，运行时会擦除类型信息。
`List<String>` 和 `List<Integer>` 运行时都是 `List`。

**问题：**
```java
List<String> list = new ArrayList<>();
list.getClass() == ArrayList.class  // true，类型信息丢失
```

**追问：<? extends T> 和 <? super T> 区别？**
- `<? extends T>`：上界通配符，只能读不能写（生产者）
- `<? super T>`：下界通配符，只能写不能读（消费者）

记忆口诀：**PECS（Producer Extends, Consumer Super）**

---

## 二、Java 并发

### 9. 线程的生命周期？

```
NEW → RUNNABLE → (BLOCKED/WAITING/TIMED_WAITING) → TERMINATED
```

| 状态 | 说明 |
|---|---|
| NEW | 创建未启动 |
| RUNNABLE | 可运行（包含运行中和就绪） |
| BLOCKED | 等待锁 |
| WAITING | 无限等待（wait/join） |
| TIMED_WAITING | 限时等待（sleep/wait(timeout)） |
| TERMINATED | 终止 |

---

### 10. sleep 和 wait 区别？

| 特性 | sleep | wait |
|---|---|---|
| 所属类 | Thread | Object |
| 释放锁 | 不释放 | 释放 |
| 使用位置 | 任意 | synchronized 块内 |
| 唤醒方式 | 时间到自动 | notify/notifyAll |
| 用途 | 暂停执行 | 线程间通信 |

---

### 11. synchronized 原理？

**回答：**
synchronized 通过 Monitor（监视器锁）实现。
每个对象都有一个 Monitor，线程进入 synchronized 块时获取 Monitor，退出时释放。

**对象锁 vs 类锁：**
```java
// 对象锁：锁的是 this
synchronized void method() {}
synchronized(this) {}

// 类锁：锁的是 Class 对象
static synchronized void method() {}
synchronized(MyClass.class) {}
```

**锁升级（JDK 1.6+）：**
无锁 → 偏向锁 → 轻量级锁 → 重量级锁

---

### 12. volatile 作用？

**两个作用：**
1. **可见性**：一个线程修改后，其他线程立即可见
2. **禁止指令重排**：防止编译器和 CPU 重排序

**不保证原子性：**
```java
volatile int count = 0;
count++;  // 不是原子操作！读-改-写三步
```

**使用场景：**
- 状态标记：`volatile boolean running = true;`
- 双重检查锁定（DCL）单例

---

### 13. synchronized 和 volatile 区别？

| 特性 | synchronized | volatile |
|---|---|---|
| 原子性 | 保证 | 不保证 |
| 可见性 | 保证 | 保证 |
| 有序性 | 保证 | 保证 |
| 阻塞 | 会 | 不会 |
| 使用范围 | 方法/代码块 | 变量 |

---

### 14. ReentrantLock 和 synchronized 区别？

| 特性 | synchronized | ReentrantLock |
|---|---|---|
| 实现 | JVM 关键字 | API |
| 释放锁 | 自动 | 手动 unlock |
| 可中断 | 不可 | lockInterruptibly |
| 超时 | 不支持 | tryLock(timeout) |
| 公平锁 | 非公平 | 可选公平/非公平 |
| 条件变量 | 单个 | 多个 Condition |

**使用建议：**
- 简单场景：synchronized
- 需要高级功能：ReentrantLock

---

### 15. 线程池参数含义？

```java
ThreadPoolExecutor(
    int corePoolSize,      // 核心线程数（常驻）
    int maximumPoolSize,   // 最大线程数
    long keepAliveTime,    // 非核心线程空闲存活时间
    TimeUnit unit,         // 时间单位
    BlockingQueue<Runnable> workQueue,  // 任务队列
    ThreadFactory threadFactory,         // 线程工厂
    RejectedExecutionHandler handler     // 拒绝策略
)
```

**执行流程：**
1. 线程数 < corePoolSize：创建核心线程
2. 核心线程满：放入队列
3. 队列满：创建非核心线程（不超过 maximumPoolSize）
4. 都满：执行拒绝策略

**拒绝策略：**
| 策略 | 行为 |
|---|---|
| AbortPolicy | 抛异常（默认） |
| CallerRunsPolicy | 调用者线程执行 |
| DiscardPolicy | 静默丢弃 |
| DiscardOldestPolicy | 丢弃队列最老任务 |

---

### 16. 为什么不推荐 Executors？

```java
// 1. newFixedThreadPool / newSingleThreadExecutor
// 队列是 LinkedBlockingQueue，无界！可能 OOM
Executors.newFixedThreadPool(10);

// 2. newCachedThreadPool
// maximumPoolSize = Integer.MAX_VALUE，可能创建大量线程 OOM
Executors.newCachedThreadPool();
```

**正确做法：**
```java
new ThreadPoolExecutor(
    5, 10, 60, TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(100),  // 有界队列
    new ThreadPoolExecutor.CallerRunsPolicy()
);
```

---

### 17. CAS 原理？什么是 ABA 问题？

**CAS（Compare And Swap）：**
比较并交换，原子操作。
```
期望值 == 内存值 → 更新为新值，返回 true
期望值 != 内存值 → 不更新，返回 false
```

**ABA 问题：**
线程 1 读取 A，线程 2 把 A 改成 B 再改回 A，线程 1 CAS 成功但数据其实变过。

**解决：**
AtomicStampedReference，加版本号。

---

## 三、JVM

### 18. JVM 内存结构？

| 区域 | 作用 | 线程 | GC |
|---|---|---|---|
| 堆 | 对象实例 | 共享 | 主要区域 |
| 栈 | 方法调用、局部变量 | 私有 | 不 GC |
| 方法区 | 类信息、常量 | 共享 | Full GC |
| 程序计数器 | 当前指令地址 | 私有 | 不 GC |
| 本地方法栈 | Native 方法 | 私有 | 不 GC |

**堆内存划分：**
- 新生代（Young）：Eden + Survivor0 + Survivor1（8:1:1）
- 老年代（Old）

---

### 19. GC Roots 有哪些？

1. 虚拟机栈中的局部变量
2. 方法区的静态变量
3. 方法区的常量引用
4. 本地方法栈中的 JNI 引用
5. synchronized 锁定的对象

**可达性分析：**
从 GC Roots 出发，不可达的对象就是垃圾。

---

### 20. GC 算法？

| 算法 | 原理 | 优点 | 缺点 |
|---|---|---|---|
| 标记-清除 | 标记垃圾后清除 | 简单 | 碎片 |
| 复制 | 存活对象复制到另一块 | 无碎片 | 空间浪费 |
| 标记-整理 | 标记后整理到一端 | 无碎片 | 效率低 |

**分代收集：**
- 新生代：复制算法（对象存活率低）
- 老年代：标记-清除/标记-整理

---

### 21. Minor GC 和 Full GC？

| 类型 | 触发条件 | 范围 | 速度 |
|---|---|---|---|
| Minor GC | Eden 满 | 新生代 | 快 |
| Full GC | 老年代满/方法区满/System.gc() | 全堆 | 慢 |

**对象进入老年代：**
1. 年龄到达阈值（默认 15）
2. 大对象直接进老年代
3. Survivor 空间不足

---

### 22. 双亲委派机制？

**加载顺序：**
```
Bootstrap ClassLoader（启动类加载器）
    ↑
Extension ClassLoader（扩展类加载器）
    ↑
Application ClassLoader（应用类加载器）
    ↑
自定义 ClassLoader
```

**流程：**
先委托父加载器加载，父加载器无法加载时才自己加载。

**好处：**
1. 避免类重复加载
2. 保证核心类安全（如 java.lang.String 不会被篡改）

---

## 四、Android 基础

### 23. Activity 生命周期？

```
onCreate → onStart → onResume → [运行中]
    → onPause → onStop → onDestroy
```

| 方法 | 时机 | 常见操作 |
|---|---|---|
| onCreate | 创建 | 初始化、setContentView |
| onStart | 可见 | - |
| onResume | 可交互 | 恢复资源、注册监听 |
| onPause | 部分可见 | 释放资源、停止动画 |
| onStop | 不可见 | 保存数据 |
| onDestroy | 销毁 | 释放所有资源 |

**追问：横竖屏切换？**
默认会销毁重建：onPause → onStop → onDestroy → onCreate → onStart → onResume

配置 `android:configChanges="orientation|screenSize"` 后只调用 onConfigurationChanged。

**追问：A 启动 B，生命周期顺序？**
A.onPause → B.onCreate → B.onStart → B.onResume → A.onStop

**追问：B 返回 A？**
B.onPause → A.onRestart → A.onStart → A.onResume → B.onStop → B.onDestroy

**追问：A 启动透明/Dialog 主题的 B？**
A.onPause → B.onCreate → B.onStart → B.onResume
（A 不会调 onStop，因为仍部分可见）

**追问：Activity 启动模式？**

| 模式 | 特点 | 场景 |
|---|---|---|
| standard | 每次创建新实例 | 默认 |
| singleTop | 栈顶复用，调 onNewIntent | 通知跳转、防重复点击 |
| singleTask | 栈内复用，清除其上所有 | 首页、登录页 |
| singleInstance | 独占任务栈 | 来电界面、系统应用 |

**追问：singleTop 生命周期？**

场景 1：A 在栈顶，再启动 A（singleTop）
```
A.onPause → A.onNewIntent → A.onResume
```
不会创建新实例，直接复用。

场景 2：栈是 A → B，启动 A（singleTop）
```
B.onPause → A.onCreate → A.onStart → A.onResume → B.onStop
```
A 不在栈顶，创建新实例，栈变成 A → B → A。

**追问：singleTask 生命周期？**

场景 1：栈是 A → B → C，启动 A（singleTask）
```
C.onPause → B.onDestroy → C.onDestroy → A.onNewIntent → A.onRestart → A.onStart → A.onResume
```
A 在栈内，清除 B、C，复用 A。

场景 2：栈是 A → B，启动 C（singleTask，不在栈内）
```
B.onPause → C.onCreate → C.onStart → C.onResume → B.onStop
```
正常创建 C。

**追问：onNewIntent 注意事项？**
- getIntent() 返回的还是旧 Intent
- 需要调用 setIntent(intent) 更新
```java
@Override
protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);
    setIntent(intent);  // 更新 Intent
    // 处理新数据
}
```

**追问：onSaveInstanceState 什么时候调用？**
- 系统可能销毁 Activity 前调用（旋转、内存不足、跳转）
- onPause 之后，onStop 之前或之后（版本差异）
- 用户主动销毁（返回键）不调用

```java
@Override
protected void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    outState.putString("key", value);
}

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    if (savedInstanceState != null) {
        value = savedInstanceState.getString("key");
    }
}
```

**追问：任务栈相关 Flag？**

| Flag | 效果 |
|---|---|
| FLAG_ACTIVITY_NEW_TASK | 新任务栈启动（singleTask 效果） |
| FLAG_ACTIVITY_SINGLE_TOP | 栈顶复用（singleTop 效果） |
| FLAG_ACTIVITY_CLEAR_TOP | 清除目标之上的 Activity |
| FLAG_ACTIVITY_CLEAR_TASK | 清空任务栈（需配合 NEW_TASK） |

常用组合：
```java
// 回到首页并清空栈
intent.setFlags(FLAG_ACTIVITY_CLEAR_TOP | FLAG_ACTIVITY_SINGLE_TOP);

// 登录成功后清空栈启动主页
intent.setFlags(FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK);
```

---

### 24. Fragment 生命周期？

```
onAttach → onCreate → onCreateView → onViewCreated → onActivityCreated
→ onStart → onResume → [运行中]
→ onPause → onStop → onDestroyView → onDestroy → onDetach
```

**注意点：**
- getActivity() 可能为 null（已 detach）
- Fragment 重叠问题：用 savedInstanceState 判断

---

### 25. Service 生命周期？

**startService：**
onCreate → onStartCommand → [运行中] → onDestroy

**bindService：**
onCreate → onBind → [运行中] → onUnbind → onDestroy

**混合使用：**
先 start 再 bind，必须同时 stopService 和 unbindService 才会销毁。

---

### 26. Handler 原理？（必考）

**核心组件：**
- Handler：发送和处理消息
- Message：消息载体
- MessageQueue：消息队列（优先队列，按 when 排序）
- Looper：循环取消息

**流程：**
1. Handler.sendMessage() 把 Message 放入 MessageQueue
2. Looper.loop() 死循环从 MessageQueue 取消息
3. 调用 msg.target.dispatchMessage() 处理消息
4. 最终回调到 Handler.handleMessage()

**追问：子线程能更新 UI 吗？**
不能。ViewRootImpl.checkThread() 会检查当前线程是否是创建 ViewRootImpl 的线程。
但在 onCreate 中可以，因为 ViewRootImpl 还没创建。

**追问：Handler 内存泄漏？**
非静态内部类持有外部 Activity 引用，Message 持有 Handler 引用。
如果 Message 还在队列中，Activity 无法回收。

**解决：**
1. 静态内部类 + 弱引用
2. onDestroy 中 removeCallbacksAndMessages(null)

---

### 27. ANR 原因和排查？

**超时时间：**
| 类型 | 超时 |
|---|---|
| 按键/触摸 | 5s |
| BroadcastReceiver | 前台 10s / 后台 60s |
| Service | 前台 20s / 后台 200s |

**常见原因：**
1. 主线程 IO 操作
2. 主线程做耗时计算
3. 死锁
4. Binder 对端阻塞

**排查：**
1. 查看 /data/anr/traces.txt
2. adb bugreport
3. StrictMode 检测

---

### 28. View 绘制流程？

**三大流程：**
```
measure（测量大小）→ layout（确定位置）→ draw（绘制内容）
```

**MeasureSpec：**
- EXACTLY：精确值（match_parent / 具体 dp）
- AT_MOST：最大不超过（wrap_content）
- UNSPECIFIED：不限制

**追问：wrap_content 为什么不生效？**
自定义 View 默认 onMeasure 把 AT_MOST 当 EXACTLY 处理。
需要重写 onMeasure 处理 wrap_content 情况。

**draw 流程：**
1. drawBackground（背景）
2. onDraw（自身内容）
3. dispatchDraw（子 View）
4. onDrawForeground（前景、滚动条）

---

### 29. 事件分发机制？

**三个方法：**
- dispatchTouchEvent：分发
- onInterceptTouchEvent：拦截（ViewGroup 独有）
- onTouchEvent：处理

**流程（U 型）：**
```
Activity.dispatchTouchEvent
    ↓
ViewGroup.dispatchTouchEvent
    ↓
ViewGroup.onInterceptTouchEvent
    ↓（不拦截）
View.dispatchTouchEvent
    ↓
View.onTouchEvent
    ↓（不消费，向上传）
ViewGroup.onTouchEvent
    ↓
Activity.onTouchEvent
```

**核心结论：**
- 返回 true：消费事件，不再传递
- 返回 false：不消费，传给上级
- onInterceptTouchEvent 返回 true：拦截，自己处理

---

### 30. RecyclerView 缓存机制？

**四级缓存：**
| 级别 | 名称 | 说明 |
|---|---|---|
| 1 | mAttachedScrap | 屏幕内，直接复用 |
| 2 | mCachedViews | 刚移出屏幕，默认 2 个，直接复用 |
| 3 | ViewCacheExtension | 自定义缓存 |
| 4 | RecycledViewPool | 按 ViewType 缓存，需重新绑定 |

**优化：**
- setHasFixedSize(true)
- setItemViewCacheSize()
- 共享 RecycledViewPool
- DiffUtil

---

## 五、性能优化

### 31. 内存泄漏常见场景？

| 场景 | 原因 | 解决 |
|---|---|---|
| Handler | 持有 Activity 引用 | 静态 + 弱引用 |
| 单例 | 持有 Context | 用 ApplicationContext |
| 静态变量 | 持有 View/Activity | 及时置 null |
| 匿名内部类 | 持有外部类引用 | 改静态内部类 |
| 资源未关闭 | Cursor/Stream/Bitmap | finally 关闭 |
| 注册未反注册 | 广播/监听器 | onDestroy 反注册 |
| WebView | 内核问题 | 独立进程 |

**检测工具：**
- LeakCanary
- Android Profiler
- MAT

---

### 32. 启动优化？

**启动类型：**
- 冷启动：进程不存在，最慢
- 热启动：进程存在，Activity 在后台
- 温启动：进程存在，Activity 被销毁

**优化手段：**
| 方法 | 说明 |
|---|---|
| 懒加载 | 非必要不在 Application 初始化 |
| 异步初始化 | 子线程初始化不依赖主线程的 SDK |
| 延迟初始化 | IdleHandler 空闲时初始化 |
| 启动器 | 拓扑排序处理依赖关系 |
| 预加载 | 闪屏页预加载首页数据 |
| 减少布局层级 | 加快 View 创建 |

---

### 33. 布局优化？

| 方法 | 说明 |
|---|---|
| 减少层级 | ConstraintLayout 替代嵌套 |
| merge | 消除冗余 ViewGroup |
| ViewStub | 延迟加载 |
| include | 复用布局 |
| 避免过度绘制 | 移除不必要背景 |

**工具：**
- Layout Inspector
- Hierarchy Viewer
- GPU 过度绘制

---

### 34. 卡顿优化？

**原因：**
主线程 16ms 内未完成绘制（60fps）

**检测：**
- Choreographer.FrameCallback
- BlockCanary
- Systrace
- TraceView

**优化：**
1. 耗时操作移到子线程
2. 减少布局层级
3. 避免频繁 GC（内存抖动）
4. 减少主线程 IO

---

## 六、架构

### 35. MVC、MVP、MVVM 区别？

| 架构 | 特点 | 缺点 |
|---|---|---|
| MVC | View 和 Model 耦合 | Activity 臃肿 |
| MVP | Presenter 中转，View 接口化 | 接口多，Presenter 臃肿 |
| MVVM | ViewModel + 数据绑定 | 学习成本，调试难 |

**MVVM 核心：**
- ViewModel：持有数据，生命周期感知
- LiveData：可观察数据，生命周期感知
- DataBinding/ViewBinding：UI 绑定

---

### 36. Jetpack ViewModel 原理？

**生命周期：**
ViewModel 在 Activity 配置变化（旋转）时不销毁，直到 Activity 真正 finish。

**原理：**
1. ViewModelStore 存储 ViewModel
2. 配置变化时，Activity 通过 NonConfigurationInstances 保存 ViewModelStore
3. 新 Activity 从 NonConfigurationInstances 恢复

---

### 37. LiveData 原理？

**特点：**
- 生命周期感知，自动取消订阅
- 只在 STARTED/RESUMED 时通知
- 粘性事件（新订阅者会收到最新值）

**原理：**
1. observe() 时包装成 LifecycleBoundObserver
2. setValue() 时遍历观察者，检查生命周期状态
3. 只通知 STARTED 以上的观察者

---

## 七、网络

### 38. OkHttp 原理？

**核心：拦截器链**
```
应用拦截器
    ↓
RetryAndFollowUpInterceptor（重试/重定向）
    ↓
BridgeInterceptor（添加请求头）
    ↓
CacheInterceptor（缓存）
    ↓
ConnectInterceptor（建立连接）
    ↓
网络拦截器
    ↓
CallServerInterceptor（发送请求）
```

**连接池：**
- 默认 5 个空闲连接，5 分钟超时
- 复用 TCP 连接，减少握手

---

### 39. Retrofit 原理？

**核心：动态代理 + 注解解析**

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .build();

ApiService service = retrofit.create(ApiService.class);
```

**流程：**
1. create() 返回动态代理对象
2. 调用方法时，解析注解生成 ServiceMethod
3. ServiceMethod 创建 OkHttp Call
4. Converter 处理请求/响应

---

## 八、项目难点模板

**结构：背景 → 问题 → 方案 → 结果**

**示例 1：启动优化**
> 项目启动时间较长，用户反馈体验差。
> 分析发现 Application 中初始化了 10+ SDK，全在主线程。
> 方案：按依赖关系分类，必须主线程的保留，其他用线程池异步初始化，还有一些用 IdleHandler 延迟。
> 结果：冷启动时间从 3s 降到 1.2s。

**示例 2：内存泄漏**
> LeakCanary 报 Activity 泄漏。
> 排查发现是单例持有了 Activity Context。
> 方案：改为 ApplicationContext，同时梳理了项目中所有单例的 Context 使用。
> 结果：泄漏消除，内存占用下降 15%。

**示例 3：列表卡顿**
> RecyclerView 滑动掉帧。
> Systrace 发现 onBindViewHolder 耗时 30ms+，主要是图片处理和布局复杂。
> 方案：图片改异步加载 + 缓存，布局用 ConstraintLayout 减少层级，共享 RecycledViewPool。
> 结果：帧率从 45fps 提升到 58fps。

