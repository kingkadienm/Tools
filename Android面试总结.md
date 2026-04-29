# Android面试总结

## 第一阶段：Java 基础（1-2周）

### 1️⃣ 面向对象（必须能说清）
重点不是定义，而是：
• 封装 / 继承 / 多态 在 Android 里的真实使用
• 抽象类 vs 接口（Java 8 以后变化）
• 重写 vs 重载
• equals / hashCode 为什么要一起重写
👉 面试常问：
"你在项目里是怎么用接口解耦的？"

**Day 1｜Java 面向对象 & 内存**
• OOP：封装 / 继承 / 多态（结合 Android 场景）
• JVM 内存初步：栈 / 堆
• String / StringBuilder / StringBuffer
✅ 输出：
• 用 2 分钟讲清「多态在 Android 中的使用」
• 解释 String 为什么不可变

**Day 2｜equals & hashCode**
• == vs equals
• 为什么重写 equals 必须重写 hashCode
• 常见错误写法
✅ 输出：
• 手写一个正确的 equals / hashCode
• 面试回答模板

---

### 2️⃣ Java 基础类型 & 内存
必须掌握：
• 基本类型 vs 包装类
• 自动装箱/拆箱 坑
• String / StringBuilder / StringBuffer
• == 和 equals 的区别（99% 会问）
👉 延伸到 Android：
• String 为什么容易造成内存问题
• 常量池 + 内存分配

---

### 3️⃣ 集合框架（高频）
你至少要能 画结构图：
• List：ArrayList / LinkedList / Vector
• Map：HashMap（重中之重） / ConcurrentHashMap
• Set：HashSet / TreeSet
重点是 HashMap：
• put 过程
• 扩容机制
• 1.7 vs 1.8 的区别（链表 → 红黑树）
• 为什么线程不安全
👉 面试官最爱追：
"HashMap 为什么在多线程下会死循环？"

**Day 3｜List & Set**
• ArrayList vs LinkedList
• 扩容机制
• Fail-Fast
✅ 输出：
• ArrayList 扩容流程
• 使用场景对比

**Day 4｜HashMap（重点）**
• put / get 流程
• 扩容
• 1.7 vs 1.8 区别
• 红黑树触发条件
✅ 输出：
• 画 HashMap 结构图
• 回答：为什么线程不安全？

**Day 5｜Map & 并发集合**
• HashMap vs ConcurrentHashMap
• Hashtable 为什么被淘汰
✅ 输出：
• ConcurrentHashMap 为什么更安全

---

### 4️⃣ 异常机制
• checked / unchecked
• try-catch-finally 执行顺序
• 自定义异常什么时候用

**Day 6｜异常 & 泛型**
• checked / unchecked
• 泛型擦除
• <? extends T> vs <? super T>

**Day 7｜复盘 + 模拟面试**
• 自问自答 20 题
• 找 5 个讲不顺的点重点补

---

## 第二阶段：Java 并发（1周）

### 1️⃣ 线程基础
• Thread vs Runnable vs Callable
• 线程生命周期
• sleep / wait / notify 区别

**Day 8｜线程基础**
• Thread / Runnable / Callable
• 线程状态
• sleep vs wait

---

### 2️⃣ 线程安全 & 锁（必考）
重点：
• synchronized 原理（对象锁 / 类锁）
• volatile 的作用（可见性 & 禁止指令重排）
• ReentrantLock vs synchronized
👉 Android 结合：
• Handler / Looper 是怎么保证线程安全的

**Day 9｜synchronized & volatile**
• 对象锁 / 类锁
• volatile 解决什么问题

**Day 10｜Lock & CAS**
• ReentrantLock
• CAS 原理 & ABA 问题

---

### 3️⃣ 线程池（非常重要）
• Executor 框架
• 4 种常见线程池
• 线程池参数含义
• 为什么 不推荐 Executors.newFixedThreadPool()
👉 面试杀招：
"如果你自己设计一个线程池，你会怎么配参数？"

**Day 11｜线程池（必考）**
• 核心参数
• 四种线程池
• 拒绝策略
⚠️ 重点：
为什么不推荐 Executors？

**Day 12｜并发工具类**
• CountDownLatch
• Semaphore
• BlockingQueue

**Day 13｜并发综合题**
• 多线程下如何保证安全
• 实际项目中的线程问题

**Day 14｜并发专项面试**
• 连续回答 30 分钟并发问题

---

## 第三阶段：JVM（3-4天）

### 1️⃣ 内存结构（必会）
• 堆（新生代 / 老年代）
• 栈
• 方法区（元空间）
• 程序计数器

**Day 15｜JVM 内存模型**
• 新生代 / 老年代
• 方法区

---

### 2️⃣ GC（高频）
• GC Roots
• 常见 GC 算法
• Minor GC / Full GC
• CMS / G1 基本思路
👉 Android 延伸：
• 内存抖动
• OOM 常见场景

**Day 16｜GC**
• GC Roots
• GC 算法
• 常见 GC

---

### 3️⃣ 类加载机制
• 双亲委派
• 类加载流程

**Day 17｜类加载机制**
• 双亲委派
• 加载流程

---

## 第四阶段：Android 基础原理（1周）

### 1️⃣ 四大组件（面试必问）
每一个都要会：
• 生命周期（为什么这样设计）
• 使用场景
• 常见坑

**Day 18｜四大组件**
• 生命周期
• 使用场景
• 常见坑

---

### 2️⃣ Handler 机制（必会）
• Handler / Looper / MessageQueue
• 子线程能不能更新 UI？为什么？
• ANR 原因

**Day 19｜Handler 机制（必考）**
• Looper / MessageQueue
• ANR 原因

---

### 3️⃣ View 绘制流程（高频）
三大流程必须能背 + 能画图
• measure
• layout
• draw
👉 延伸：
• 自定义 View
• wrap_content 为什么不生效

**Day 20｜View 绘制流程**
• measure / layout / draw
• 自定义 View

**Day 21｜Android 模拟面试**
• 不查资料连续答 1 小时

---

## 第五阶段：Android 进阶（1周）

### 1️⃣ 性能优化
• 内存优化
• 布局优化
• 启动优化
• 卡顿优化（Choreographer）

**Day 22｜性能优化**
• 内存泄漏
• 布局优化
• 启动优化

---

### 2️⃣ 架构
• MVC / MVP / MVVM
• Jetpack（ViewModel / LiveData）
• 模块化 / 组件化

**Day 23｜架构**
• MVP / MVVM
• Jetpack

---

### 3️⃣ 网络 & 存储
• OkHttp 原理
• Retrofit
• SQLite / Room
• 缓存策略

**Day 24｜网络**
• OkHttp 原理
• Retrofit

**Day 25｜存储**
• SharedPreferences
• SQLite / Room

---

## 第六阶段：项目包装 & 冲刺（3天）

**Day 26｜项目复盘**
• 一个核心项目讲 10 分钟
• 技术难点 3 个

**Day 27｜高频题冲刺**
• 背答案 ❌
• 讲逻辑 ✅

**Day 28｜终极模拟面试**
• 自我介绍
• 技术深挖
• 反问准备

---

## 面试回答模板

### 模板结构（记住这个顺序）
一句话定义 → 核心原理 → 场景 / 优缺点 → 项目中的使用

---

### 1️⃣ HashMap 原理（必考）
**标准回答：**
HashMap 底层是 数组 + 链表 + 红黑树 的结构。
put 的时候先根据 key 的 hash 定位到数组下标，如果发生哈希冲突，会以链表或红黑树的形式存储。
当链表长度超过阈值（8）并且数组容量大于 64 时，会转成红黑树来提高查询效率。
在项目中我主要用 HashMap 做缓存或数据快速查找，但在多线程场景下会用 ConcurrentHashMap。
**追问准备：**
• 为什么线程不安全？
• 1.7 和 1.8 的区别？
• 扩容过程？

---

### 2️⃣ equals 和 hashCode
**标准回答：**
equals 用来判断两个对象逻辑上是否相等，hashCode 用于在哈希结构中快速定位。
如果两个对象 equals 相等，那它们的 hashCode 必须相等，否则在 HashMap 中会出现无法取值的问题。
所以在重写 equals 的时候必须同时重写 hashCode。

---

### 3️⃣ synchronized 和 volatile 区别
**标准回答：**
synchronized 主要解决的是 原子性和可见性，它通过加锁保证同一时间只有一个线程执行临界区代码。
volatile 主要解决的是 可见性和指令重排，但不保证原子性。
在项目中如果是简单的状态标记会用 volatile，如果涉及到复合操作就必须用 synchronized 或 Lock。
**加分点：**
• 提一句 JMM（Java 内存模型）

---

### 4️⃣ 线程池为什么要用？参数含义？
**标准回答：**
线程池主要是为了 复用线程、控制并发数量、避免频繁创建销毁线程带来的性能问题。
核心参数包括核心线程数、最大线程数、任务队列、线程工厂和拒绝策略。
在项目中一般不会直接用 Executors 创建线程池，而是通过 ThreadPoolExecutor 明确参数，防止 OOM。
**追问准备：**
• 拒绝策略有哪些？
• 如何设置线程数？

---

### 5️⃣ JVM 内存结构
**标准回答：**
JVM 内存主要分为 栈、堆、方法区、程序计数器。
栈主要存储方法调用和局部变量，是线程私有的；
堆用于存放对象实例，是 GC 的主要区域；
方法区存放类信息、常量等。
在 Android 中，大部分 OOM 问题都和堆内存有关。

---

### 6️⃣ GC 什么时候发生？
**标准回答：**
GC 主要发生在堆内存不足的时候。
新生代空间不足会触发 Minor GC，老年代空间不足会触发 Full GC。
如果 Full GC 后仍然无法分配对象，就会抛出 OOM。
在项目中会通过减少对象创建、避免内存泄漏来降低 GC 压力。

---

### 7️⃣ Activity 生命周期
**标准回答：**
Activity 的生命周期主要包括 onCreate、onStart、onResume、onPause、onStop、onDestroy。
前台可交互状态是 onResume，onPause 之后就不应该再执行耗时操作。
在项目中我通常在 onCreate 做初始化，在 onResume 恢复资源，在 onPause 或 onStop 释放资源，防止内存泄漏。
**追问准备：**
• 横竖屏切换
• onSaveInstanceState

---

### 8️⃣ Handler 原理（必考）
**标准回答：**
Handler 主要用于线程间通信。
每个线程可以有一个 Looper，Looper 内部维护一个 MessageQueue。
子线程通过 Handler 发送 Message，Message 进入队列后由 Looper 循环取出，最终在创建 Handler 的线程中处理。
主线程正是通过这套机制实现 UI 更新的。
**必会追问：**
• 子线程能不能更新 UI？
• ANR 为什么发生？

---

### 9️⃣ View 绘制流程
**标准回答：**
View 的绘制流程主要分为三步：measure、layout、draw。
measure 负责测量大小，layout 负责确定位置，draw 负责绘制内容。
在自定义 View 时，通常需要重写 onMeasure 或 onDraw。
如果 wrap_content 不生效，通常是 onMeasure 处理不当。

---

### 🔟 内存泄漏怎么处理？
**标准回答：**
内存泄漏通常是对象生命周期不一致导致的，比如 Activity 被静态引用。
常见场景包括 Handler、单例、匿名内部类等。
在项目中我会结合 LeakCanary 定位问题，并通过弱引用或及时释放资源来解决。

---

### 1️⃣1️⃣ 项目难点怎么讲？
**标准模板（非常重要）**
背景是什么 → 问题是什么 → 你做了什么 → 结果如何
**示例：**
项目中曾遇到启动慢的问题，通过分析发现主要是主线程初始化过多。
后来我把非必要初始化放到子线程，并做了懒加载，启动时间降低了约 30%。

---

## 学习方式

**错误方式 ❌**
• 看视频 → 觉得懂了
• 看博客 → 不总结

**正确方式 ✅**
1. 每个知识点 用一句话总结
2. 能不能写伪代码
3. 能不能举 Android 项目里的例子
4. 模拟面试回答（说出来）
