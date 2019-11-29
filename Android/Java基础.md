# 一、Java基础

## 1、引用类型

对象的可及性

强可及对象，永远不会都不会被GC回收

软可及对象：当系统内存不足的时候，被GC回收

弱可及对象：当系统GC发现这个对象，就被回收

虚引用

```java
public class ReferenceTest {
    public static void main(String[] args) {
        String str = "abc"; // 常量池中
        // 1、再堆内存中创建了String对象，2、在常量池中创建了abc对象
//        String str = new String("abc");
        // 创建一个软引用，引用到str
        SoftReference<String> sfr = new SoftReference<>(str);

        // 创建一个弱引用，引用到str
        WeakReference<String> wrf = new WeakReference<>(str);

        str = null; // 相当于去掉了强引用链
        // 清除软引用的引用链
        sfr.clear();

        System.gc(); // 堆内存

        String srfString = sfr.get();
        String wrfString = wrf.get();
        System.out.println("软引用: "+srfString);
        System.out.println("弱引用: "+wrfString);
    }
}
```

## 2、线程池

控制执行的线程数

1）信号量实现

```java
public class ThreadPoolTest {
    // 信号量，允许个数，相当于放了几个锁
    private static Semaphore semaphore = new Semaphore(3);
    public static void main(String[] args) {
        for (int i=0;i<100;i++){
            new Thread(()->{
                try {
                    method();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }

    public static void method() throws InterruptedException {
        semaphore.acquire();//获取一把锁

        System.out.println("ThreadName="+Thread.currentThread().getName()+" 过来了 ");

        Thread.sleep(1000);

        System.out.println("ThreadName="+Thread.currentThread().getName()+" 出去了 ");

        semaphore.release();
    }
}
```

2）自定义线程池实现

```java
public class ThreadPoolTest2 {

    public static void main(String[] args) {

        LinkedBlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>(100);
        ThreadFactory threadFactory = new ThreadFactory() {
            AtomicInteger atomicInteger = new AtomicInteger(0);

            @Override
            public Thread newThread(Runnable r) {
                Thread thread = new Thread(r);
                thread.setName("MyThread: " + atomicInteger.getAndIncrement());
                return thread;
            }
        };
        
        /**
        执行流程：
        核心线程执行任务--》多余的放到任务队列---》队列满了---》创建新的线程（不能超过最大线程数）
        */
        ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor
            (4, 10, 1, TimeUnit.SECONDS, workQueue, threadFactory);

        for (int i = 0; i < 110; i++) {  // 最多只能放100+10个任务，超过就会抛出异常
            System.out.println("i="+i);
            poolExecutor.execute(() -> {
                try {
                    method();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
    }

    public static void method() throws InterruptedException {
        semaphore.acquire();//获取一把锁

        System.out.println("ThreadName=" + Thread.currentThread().getName() + " 过来了 ");

        Thread.sleep(1000);

        System.out.println("ThreadName=" + Thread.currentThread().getName()+" 出去了 ");

        semaphore.release();
    }
}
```



## 3、实现简单的 ButterKnife框架

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ViewInject {

    String name();

    int age();
}

public class User {

    @ViewInject(name = "xiaolin", age = 10)
    private String name;
    private int age;

    private void eat(String food) {
        System.out.println("eat: " + food);
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```

测试依赖注入：

```java
public class AnnotationTest {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException, NoSuchMethodException, InvocationTargetException {

        User user = new User();
        // 1、获取到User的字节码
        Class clazz = User.class;
//        clazz.getField() // 只能获取 public 字段
        // 2、获取字节码中的 name 字段
        Field declaredField = clazz.getDeclaredField("name");
        Field ageField = clazz.getDeclaredField("age");

        // 3、获取字段上含有指定的注解
        ViewInject viewInject = declaredField.getAnnotation(ViewInject.class);
        if (viewInject != null) {
            // 4、获取自定义注解的参数
            String name = viewInject.name();
            int age = viewInject.age();
            System.out.println("name=" + name + " age=" + age);

            // 5、通过反射设置值给user对象
            declaredField.setAccessible(true); // 暴力反射
            declaredField.set(user, name); // 将name设置给user对象的declaredField属性
            ageField.setAccessible(true);
            ageField.set(user, age);   // 可以实现类似ButterKnife，通过activity.findById（id，注解传入）赋值控件

            System.out.println("user:" + user);

            Method method = clazz.getDeclaredMethod("eat", String.class);
            method.setAccessible(true);
            /**
             * 扩展
             * 1、根据注解的 id 获取控件，然后设置 click 监听；
             * 2、在监听器里反射调用用户的方法
             */
            method.invoke(user, "苹果");
        } else {
            System.out.println("没有注解");
        }

    }
}
```

