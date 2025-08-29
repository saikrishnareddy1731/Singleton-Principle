# Singleton Design Pattern (Java) - Notes

This repository demonstrates different ways to implement the Singleton pattern in Java,
how serialization and reflection can violate the pattern, and how to prevent those violations.

## 1) What is a Singleton?
A **Singleton** ensures a class has **exactly one instance** for the entire application
and provides a single, global access point to that instance.

---

## 2) Lazy Loading vs. Eager Loading

### Eager Loading
- The instance is created when the class is loaded by the JVM.
- Use when the instance is lightweight and always needed (for example, a small cache).
- Downside: may allocate resources even if the instance is never used.

### Lazy Loading
- The instance is created only on the first call to `getInstance()`.
- Saves resources if the instance is not always required.
- Not thread-safe by default (requires synchronization or other thread-safety measures).

---

## 3) Common Singleton Implementations

### A) Eager Initialization
- Create the instance at class load time.
- Simple and inherently thread-safe.
- Can waste memory if unused.

### B) Lazy Initialization (Non-Thread-Safe)
- Create the instance on first access.
- Not safe in multi-threaded contexts.

### C) Thread-Safe (Double-Checked Locking)
- Use `synchronized` with a double null-check to reduce lock overhead.
- Appropriate for multi-threaded applications requiring lazy initialization.

### D) Serializable Singleton
- Problem: deserialization can create a new instance.
- Fix: implement `readResolve()` to return the existing instance.
- * **Problem:**
   When a singleton is serialized and then deserialized, Java creates a new object instead of using the existing singleton instance. This happens because deserialization bypasses the private constructor, so a fresh instance is allocated in memory, breaking the singleton guarantee.

  1. It allocates memory.
  2. It calls the constructor **without using your existing instance**.
  3. This results in a **new object** being created, breaking the singleton guarantee.

* **Fix – `readResolve()`:**
  Java provides a special hook method called `readResolve()`.
  If this method is defined, it is invoked during deserialization instead of returning the newly created object.

  ```java
  protected Object readResolve() {
      return getInstance(); // or return instance;
  }
  ```

* **How it works internally:**

  * After reading the object from the stream, the JVM checks if the class defines `readResolve()`.
  * If yes, the **return value of this method replaces the deserialized object**.
  * This ensures that deserialization does not create a new instance, but instead reuses the original singleton instance.

* **Important:**

  * Without `readResolve()`, every deserialization produces a new instance.
  * With `readResolve()`, all deserialization calls return the **same singleton instance**, preserving the pattern.

### E) Reflection Issue
- Reflection can access private constructors and create a new instance.
- Mitigation: checks in constructor or prefer enum-based singletons.

- * **Problem:**
  Java Reflection can access **private constructors**. This allows code to bypass the singleton’s control and **create a new instance**, breaking the singleton guarantee.
Normally, a singleton restricts instance creation via a private constructor. But with reflection, code can bypass this restriction:
Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
constructor.setAccessible(true);          // Makes private constructor accessible
Singleton newInstance = constructor.newInstance(); // Creates a new object
This creates a new inst

  ```java
  Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
  constructor.setAccessible(true);
  Singleton instance2 = constructor.newInstance(); // new instance created!
  ```

* **Mitigation:**

  1. **Constructor check:**
     You can throw an exception if an instance already exists.

     ```java
     private Singleton() {
         if (instance != null) {
             throw new RuntimeException("Use getInstance() method to create");
         }
     }
     ```

  2. **Prefer enum-based singletons:**
     Enums are inherently singleton and immune to reflection attacks.

     ```java
     public enum SingletonEnum {
         INSTANCE;
     }
     ```

* **Important:**

  * Reflection can break the singleton pattern if no precautions are taken.
  * Enum singletons are the safest and simplest solution against reflection attacks.

### F) Enum Singleton (Recommended)
- JVM guarantees a single instance for each enum constant.
- Safe against serialization and reflection attacks.
- Thread-safe by default and the simplest robust solution.

---

## 4) How Singletons Get Broken

- **Serialization**  
  Deserializing a serialized singleton can create a second instance unless `readResolve()` is implemented.

- **Reflection**  
  Private constructors can be accessed reflectively to instantiate a new object.

---

## 5) Best Practices

- Prefer **Enum Singleton** for a robust and simple solution.
- If you need lazy creation and thread-safety, use **double-checked locking**.
- Use **eager initialization** when the instance is small and always required.
- If the singleton is serializable, implement **`readResolve()`** to return the singleton instance.

---

## 6) Visual: Violations & Fixes

```text
+-------------------+
|  Singleton Class  |
+-------------------+
         |
         v
    [One Instance]
         |
    --------------
    |            |
    v            v
Serialization  Reflection
(Object↔File)  (Private ctor via reflection)
    |            |
   New          New
  Object       Object
    |            |
    v            v
 Breaks Singleton Pattern

Fixes:
- Serializable singleton: implement readResolve() → return existing instance
- Thread-safe lazy: use double-checked locking
- Enum singleton: JVM-enforced single instance; safe vs. serialization & reflection
```

---

## 7) Comparison Table

| Approach                  | Thread-Safe | Serialization-Safe | Reflection-Safe | When to Use                                        |
|--------------------------|-------------|--------------------|-----------------|----------------------------------------------------|
| Eager Initialization     | Yes         | Yes                | No              | Instance is small and always needed                |
| Lazy Initialization      | No          | No                 | No              | Simple, single-threaded contexts only              |
| Thread-Safe (DCL)        | Yes         | No                 | No              | Multi-threaded lazy creation                       |
| Serializable Singleton   | Yes         | Yes (with `readResolve`) | No       | Needs Java serialization                           |
| Enum Singleton           | Yes         | Yes                | Yes             | Recommended default: simplest and most robust      |

---

