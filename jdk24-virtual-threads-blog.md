
# 🚀 JDK 24 Makes Virtual Threads Even Better (No More Pinning!)

Java’s new **virtual threads** made it easier to build fast, scalable apps. But in **JDK 21**, there was one big problem — if your code used `synchronized` methods or blocks, virtual threads didn’t work as well as they should.

Now with **JDK 24**, that problem is fixed. Let’s take a look at what changed and why it matters — even for older codebases!

## 🧵 What Are Virtual Threads?

In **JDK 21**, Java introduced **virtual threads** — lightweight threads that are much faster and more efficient than traditional (platform) threads. You can run **thousands of virtual threads** without using too much memory or CPU.

But there was a catch...

## ⚠️ The Problem in JDK 21: Pinning

In JDK 21, if a virtual thread entered a `synchronized` method and **got blocked** (like waiting on I/O or a database), it would get **"pinned"** to a platform thread.

That means the platform thread had to sit and wait — it couldn’t help other virtual threads — which defeated the whole point of using virtual threads.

### 🍽 A Simple Analogy

Imagine a busy restaurant:

- **JDK 21:** A waiter (platform thread) is stuck standing next to one customer (virtual thread) the whole time they’re waiting for food. No other customers can be served.

- **JDK 24:** The waiter gives the customer a pager and helps other customers. When the food is ready, any waiter can serve it.

## ✅ The Fix in JDK 24: No More Pinning

Thanks to **JEP 491**, JDK 24 allows virtual threads to **use synchronized code without getting pinned**, as long as different threads are locking on different objects.

That means:

- You can still use `synchronized` blocks or methods
- You get all the performance benefits of virtual threads
- Legacy code (older codebases) can work better without major rewrites

## 🧑‍💻 What is `synchronized` in Java?

`synchronized` is used to **protect shared data** so only one thread can use it at a time.

### 1. Synchronizing the Whole Method

```java
public class MethodSynchronizedCounter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}
```

### 2. Synchronizing Just a Block of Code

```java
public class BlockSynchronizedCounter {
    private int count = 0;
    private final Object lock = new Object();

    public void increment() {
        synchronized (lock) {
            count++;
        }
    }
}
```

## 🚀 How Much Faster Is JDK 24?

### ⚡ Example 1: Simple Java Benchmark

```java
for (int i = 0; i < 5000; i++) {
    final Object lock = new Object();
    executor.submit(() -> {
        try {
            doCpuWork(); // Math operations

            synchronized (lock) {
                Thread.sleep(5); // Blocking call
            }
        } catch (Exception e) {
            // Handle error
        }
    });
}
```

**Results:**

- 🐢 **JDK 21:** 31.791 seconds  
- 🚀 **JDK 24:** 0.454 seconds  

### 📦 Example 2: Spring Boot Inventory System

```java
@Service
public class InventoryService {

    private final Map<String, Integer> inventory = new ConcurrentHashMap<>();
    private final ConcurrentHashMap<String, Object> productLocks = new ConcurrentHashMap<>();

    public boolean updateInventory(String productId, int quantity) {
        Object lock = productLocks.computeIfAbsent(productId, k -> new Object());

        synchronized (lock) {
            int stock = inventory.getOrDefault(productId, 0);
            if (stock + quantity < 0) return false;

            dbService.persistInventoryChange(productId, quantity);
            inventory.put(productId, stock + quantity);
            return true;
        }
    }
}
```

**Results:**

- 🐌 **JDK 21:** 800 requests/sec  
- 🏎️ **JDK 24:** 4,264 requests/sec  

## ⚙️ How to Use Virtual Threads in Spring Boot

```properties
spring.threads.virtual.enabled=true
```

## ✅ Best Practices for Virtual Threads with `synchronized`

- Use **a different lock** for each resource
- Avoid using `this` or static locks
- Prefer built-in thread-safe tools
- Be consistent and test thoroughly

## 🛠 Thinking About Migration?

- Find `synchronized` code with blocking operations
- Refactor to use per-resource locks
- Test well
- Check library compatibility

## 🎯 Final Thoughts

JDK 24 brings a big win:

- Virtual threads now work great with synchronized code  
- Older apps can now scale better without big changes  
- You get better performance with just a small update  

**Have you tried virtual threads in JDK 24? Let me know in the comments!**
