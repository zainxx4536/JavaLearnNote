# Java 函数式接口与 Lambda

很多公共方法的处理步骤相同，只是其中某一步不同：缓存查询中的“真正加载数据”、集合处理中的“筛选条件”、批量操作中的“每个元素如何处理”。把这一步写死在公共方法里，会限制复用；为每种情况复制整个方法，又容易产生重复。

Java 可以通过接口把这段行为作为参数传入。函数式接口规定输入与输出，Lambda 或方法引用提供实现，接收方再按自己的流程调用。本文使用 Java 8 兼容语法，重点说明这些动作何时执行、结果如何流转，以及它们怎样用于缓存和集合处理。

## 一、从传递数据到传递行为

### 1. 查询结果与查询动作

普通参数通常提供已经算好的数据，例如把一个字符串交给展示方法。函数式接口参数则可以提供“需要时怎样取得数据”：

```java
Supplier<String> loadName = () -> {
    System.out.println("执行加载动作");
    return "张三";
};

System.out.println("加载动作已定义");
String name = loadName.get();
System.out.println(name);
```

输出顺序是：

```text
加载动作已定义
执行加载动作
张三
```

==定义 Lambda 时提供的是行为，调用 `get()` 时才执行这里的加载动作。== 接收方因此能够先做判断：如果已有数据就直接返回，确实需要加载时才调用它。把这种可变化的行为作为参数，通常称为**行为参数化**。

Java 中传入的仍是具有接口类型的引用，不是把一段源码文本交出去再解释执行。`loadName` 的类型是 `Supplier<String>`，它约定了调用入口与结果类型。

### 2. 函数式接口与匿名内部类

==函数式接口的核心条件是只有一个需要实现的抽象方法；可以同时拥有 `default` 方法和 `static` 方法。== 与 `Object` 公共方法等价的声明不计入这个抽象方法数量。`Supplier` 的核心定义可以简化为：

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

`@FunctionalInterface` 表达设计意图并启用编译检查。如果增加第二个独立的抽象方法，编译器会报告错误；不写这个注解，只要满足接口条件，仍可使用 Lambda。它是 JDK 的能力，不需要 Spring 或 Guava。规则可在 [FunctionalInterface 官方说明](https://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html)中核对。

下面两种写法在这个示例中都提供“返回默认名称”的行为，可以选择其中一种：

```java
Supplier<String> byAnonymousClass = new Supplier<String>() {
    @Override
    public String get() {
        return "默认名称";
    }
};

Supplier<String> byLambda = () -> "默认名称";
```

Lambda 省去了显式声明匿名实现类和方法的样板代码。它与匿名内部类并非所有语义都相同，例如 Lambda 中的 `this` 指向外围实例，不会引入匿名类自己的 `this`。这里的对照用于理解接口方法如何得到实现，不应扩展成两种写法在任何场景都完全等价。

## 二、参数、返回值与执行时机

### 1. Lambda 怎样匹配接口方法

Lambda 的基本形式为“参数列表 `->` 方法体”。编译器根据赋值目标或方法参数的接口类型，确定参数与返回值的约定：

| 形式 | 示例 | 对应的含义 |
| --- | --- | --- |
| 无入参，返回一个值 | `() -> "默认值"` | 实现一个无参、有结果的方法 |
| 一个入参，返回一个值 | `text -> text.length()` | 接收文本并返回长度 |
| 一个入参，没有返回值 | `text -> System.out.println(text)` | 接收文本并执行输出 |
| 多个入参 | `(left, right) -> left + right` | 由目标接口规定两个参数的类型与结果类型 |
| 多行逻辑 | `value -> { int result = value * 2; return result; }` | 用代码块执行多步处理并返回结果 |

==Lambda 不能只根据箭头右边的代码判断类型，必须结合目标接口的参数与返回值。== 例如 `text -> text.length()` 可以用于 `Function<String, Integer>`，也可以用于返回基本类型 int 的 `ToIntFunction<String>`，具体类型由使用位置决定。

对有返回值的方法，只有一个表达式时通常可以省略 `return` 和花括号；使用代码块时，需要在相应路径显式返回结果。对返回 void 的接口，只需执行动作，不必构造一个无用结果。

### 2. 捕获外围变量

接口方法没有入参，不代表 Lambda 不能使用查询条件。条件可以来自外围作用域：

```java
String userId = "U1001";
Supplier<String> cacheKey = () -> "user:" + userId;
System.out.println(cacheKey.get()); // user:U1001
```

这里 `get()` 没有参数，但 Lambda 捕获了外围的 `userId`。==被捕获的局部变量必须是 final 或事实上的 final，即初始化后没有再次赋值；方法参数也遵守这一规则。== 如果在同一作用域中重新给 `userId` 赋值，就不能再按上面的方式捕获它。

final 限制的是变量绑定，不会自动使引用的对象变成不可变对象：

```java
List<String> names = new ArrayList<>();
Consumer<String> addName = name -> names.add(name);
addName.accept("张三");
System.out.println(names); // [张三]
```

`names` 没有被重新赋值，但列表内容发生了变化。这也不使 `ArrayList` 自动具备线程安全性。变量捕获与 Lambda 方法体规则见 [Java 语言规范 15.27.2](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.27.2)。

### 3. 延迟执行与重复执行

==Lambda 不会自动创建线程，也不会自动记忆结果；执行线程、执行次数和缓存策略由调用方或具体实现决定。== 普通同步方法中调用 `get()`、`apply()` 等入口，行为就在调用线程执行；交给线程池后，才需要按照线程池的调度规则理解。

延迟也不是所有接收方都承诺的能力。有的方法接收接口后立即调用，有的方法保存到以后调用，有的方法按条件决定是否调用。阅读接口使用代码时，应继续追踪：谁持有它，谁调用抽象方法，在哪个分支调用。

## 三、四种常用函数式接口

下面四种接口都位于 `java.util.function` 包。T、R 是类型参数，表示输入或结果类型；它们不是固定的业务类名。

| 接口 | 抽象方法 | 输入 | 输出 | 常见用途 |
| --- | --- | --- | --- | --- |
| `Supplier<T>` | `T get()` | 无 | T 类型结果 | 提供数据、按需加载、创建对象 |
| `Function<T, R>` | `R apply(T value)` | 一个 T | 一个 R | 类型转换、对象映射、生成展示内容 |
| `Consumer<T>` | `void accept(T value)` | 一个 T | 无返回值 | 输出、记录日志、发送通知 |
| `Predicate<T>` | `boolean test(T value)` | 一个 T | 布尔判断结果 | 过滤、校验条件、组合规则 |

==可以按“提供、转换、处理、判断”选择接口，再用 `get、apply、accept、test` 触发对应行为。== 使用时导入相应 JDK 类型即可；下面的片段省略外围类和方法，可分别放入 `main` 或测试方法中运行。

### 1. Supplier：提供结果

```java
Supplier<String> defaultName = () -> "游客";
System.out.println(defaultName.get()); // 游客

Supplier<Long> currentTime = System::currentTimeMillis;
long now = currentTime.get();
System.out.println(now); // 调用时的毫秒时间戳
```

`Supplier<String>` 表示 `get()` 返回 String，`Supplier<Long>` 表示返回 Long。`System::currentTimeMillis` 为后者提供读取当前时间的行为，调用 `get()` 才读取。

无入参适合查询条件已经由外围变量确定，或数据本来就不需要额外输入的情况。如果公共流程需要把一个条件交给行为，再取得结果，则可考虑 `Function<条件类型, 结果类型>`。

多次 `get()` 会再次执行所提供的行为，但不保证结果不同。例如同一毫秒内两次读取时钟可以得到相同数值，查询也可以读到同一条数据。需要“只加载一次”或“保存旧结果”时，应显式设计缓存。接口契约见 [Supplier 官方 API](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html)。

### 2. Function：转换结果

```java
Function<String, Integer> toLength = text -> text.length();
System.out.println(toLength.apply("Java")); // 4

Function<Integer, String> toLabel = size -> "长度：" + size;
Function<String, String> describeText = toLength.andThen(toLabel);
System.out.println(describeText.apply("Java")); // 长度：4

System.out.println(toLabel.compose(toLength).apply("Java")); // 长度：4
```

`toLength` 把字符串映射为 Integer；`toLabel` 把 Integer 转换为 String。组合后的 `describeText` 接收字符串，依次执行这两个步骤，返回最终展示内容。常见的实体转 DTO、提取属性、格式化结果，都可以按输入与输出设计映射。

==`a.andThen(b)` 先执行 a，再把结果交给 b；`a.compose(b)` 先执行 b，再把结果交给 a。== 两个函数的类型要能够衔接，组合时并没有处理数据，调用组合对象的 `apply()` 才执行。`Function.identity()` 则提供“返回输入本身”的转换，常用于保留原对象。见 [Function 官方 API](https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html)。

### 3. Consumer：执行处理

```java
Consumer<String> recordOrder = orderId ->
        System.out.println("记录订单：" + orderId);
recordOrder.accept("O1001");

Consumer<String> notifyUser = orderId ->
        System.out.println("模拟通知：订单 " + orderId + " 已处理");

Consumer<String> handleOrder = recordOrder.andThen(notifyUser);
handleOrder.accept("O1002");
```

输出为：

```text
记录订单：O1001
记录订单：O1002
模拟通知：订单 O1002 已处理
```

==Consumer 的“没有返回值”指 `accept()` 返回 void，不代表它没有产生效果。== 它可能修改对象、输出日志或访问外部服务，因此不能给 `accept()` 的结果赋值，也不能据此认为它不会失败。这里用输出语句模拟通知，没有实际发送请求。

Consumer 的 `andThen` 把同一个输入依次交给两个处理动作，没有把前一步的返回结果传给下一步。如果前一个动作抛异常，后一个不会执行；组合也不会自动建立事务。见 [Consumer 官方 API](https://docs.oracle.com/javase/8/docs/api/java/util/function/Consumer.html)。

### 4. Predicate：判断条件

下面把“年龄不小于 18 且小于 60”作为教学筛选条件：

```java
Predicate<Integer> isAdult = age -> age >= 18;
Predicate<Integer> isUnder60 = age -> age < 60;
Predicate<Integer> isEligible = isAdult.and(isUnder60);

System.out.println(isEligible.test(24)); // true
System.out.println(isEligible.test(60)); // false
System.out.println(isAdult.negate().test(16)); // true

Predicate<Integer> isChildOrSenior =
        isAdult.negate().or(isUnder60.negate());
System.out.println(isChildOrSenior.test(60)); // true
```

`and` 要求同时满足，`or` 要求至少满足一个，`negate` 对结果取反。`isChildOrSenior` 因此表示“不满 18 或不小于 60”，并不是对输入年龄做修改。

==Predicate 的 and、or 按短路规则执行：and 的前项为 false 时不再执行后项，or 的前项为 true 时不再执行后项。== 规则组合适合表达筛选条件，但用于业务校验时，还要决定是否需要返回具体失败原因；仅有 boolean 不能自动携带“年龄不符”等解释。见 [Predicate 官方 API](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html)。

## 四、方法引用与常用接口变体

### 1. 用方法引用复用现有方法

当 Lambda 只是把参数交给一个已有方法时，方法引用通常更简洁。它也需要目标函数式接口，并不是脱离类型系统的独立函数：

| 方法引用 | 一种对应的目标类型 | 可对照理解的 Lambda |
| --- | --- | --- |
| `Integer::parseInt` | `Function<String, Integer>` | `text -> Integer.parseInt(text)` |
| `System.out::println` | `Consumer<String>` | `text -> System.out.println(text)` |
| `String::trim` | `Function<String, String>` | `text -> text.trim()` |
| `ArrayList::new` | `Supplier<List<String>>` | `() -> new ArrayList<String>()` |

`String::trim` 的输入对象承担了调用者的角色；`System.out::println` 已经指定调用对象，由传入参数作为输出内容。构造器引用则在接口方法被调用时创建对象。遇到重载方法，应结合目标类型判断匹配的是哪一个签名。方法引用规则可参考 [Oracle 方法引用教程](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)。

### 2. 双参数、同类型运算与基本类型

四种核心接口并不覆盖所有输入形状。常用扩展包括：

| 接口 | 输入与输出 | 示例用途 |
| --- | --- | --- |
| `BiFunction<T, U, R>` | 两个输入，返回 R | 两项数据组合成一个结果 |
| `BiConsumer<T, U>` | 两个输入，无返回值 | 遍历 Map 的键和值 |
| `BiPredicate<T, U>` | 两个输入，返回 boolean | 比较两个值是否符合条件 |
| `UnaryOperator<T>` | 一个 T，返回 T | 对同一种类型进行变换 |
| `BinaryOperator<T>` | 两个 T，返回 T | 求和、选最大值、归并结果 |
| `IntSupplier`、`IntConsumer`、`IntPredicate` | 对应 int 结果或输入 | 减少 Integer 装箱与拆箱 |
| `ToIntFunction<T>` | 输入 T，返回 int | 把对象映射成基本类型整数 |

```java
BiFunction<Integer, Integer, Integer> add = (left, right) -> left + right;
System.out.println(add.apply(2, 3)); // 5

UnaryOperator<String> trim = text -> text.trim();
System.out.println(trim.apply(" Java ")); // Java

Map<String, Integer> scores = new LinkedHashMap<>();
scores.put("张三", 90);
scores.put("李四", 85);
scores.forEach((name, score) -> System.out.println(name + "：" + score));
```

`Map.forEach` 接收的是 BiConsumer，因为每次提供键和值两个输入。`UnaryOperator<T>` 是 `Function<T, T>` 的特化，`BinaryOperator<T>` 是同类型双输入与结果的特化。泛型参数不能直接写基本类型 `int`；需要基本类型接口时，应使用相应特化。完整列表见 [java.util.function 官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)。

## 五、集合处理、默认值与缓存查询

### 1. 用四种接口组成一段数据处理

下面的完整示例可以直接保存为 `FunctionalInterfaceDemo.java` 运行。数据源、筛选、转换和输出分别由四种接口承担：

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

public class FunctionalInterfaceDemo {
    public static void main(String[] args) {
        Supplier<List<Integer>> ageSource = () -> Arrays.asList(16, 24, 60);
        Predicate<Integer> isEligible = age -> age >= 18 && age < 60;
        Function<Integer, String> toText = age -> "符合条件的年龄：" + age;
        Consumer<String> output = System.out::println;

        ageSource.get().stream()
                .filter(isEligible)
                .map(toText)
                .forEach(output);
    }
}
```

输出为 `符合条件的年龄：24`。执行 `get()` 先得到列表；`filter` 根据 Predicate 保留符合条件的元素；`map` 用 Function 转换元素；`forEach` 让 Consumer 处理最后的结果。

==Stream 的 filter、map 等中间操作是延迟的；本例执行到 forEach 这个终端操作，才触发筛选、转换与输出。== 这里使用顺序流，没有自动把每个步骤提交到不同线程。集合本身也没有因为被筛选而删除元素。规则见 [JDK 8 Stream 说明](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)。

### 2. Optional 中的按需默认值

如果默认值需要查询或较多计算，应区分“传入一个值”与“传入取得值的行为”：

```java
Optional<String> cachedName = Optional.of("已有名称");
Supplier<String> loadDefault = () -> {
    System.out.println("加载默认名称");
    return "默认名称";
};

String first = cachedName.orElse(loadDefault.get());
String second = cachedName.orElseGet(loadDefault);
System.out.println(first + "/" + second);
```

输出为：

```text
加载默认名称
已有名称/已有名称
```

第一行调用 `orElse` 前，需要先求出参数 `loadDefault.get()`，所以已经执行了加载，尽管最后返回的仍是原有名称。第二行把 Supplier 本身交给 `orElseGet`；原值存在，就不执行默认值加载。

==`orElse` 接收默认值，`orElseGet` 接收按需提供默认值的 Supplier；昂贵操作是否提前发生，取决于传入的是计算结果还是行为。== `Optional.empty()` 情况下，后者才会调用 Supplier。相关方法约定见 [Optional 官方 API](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)。

### 3. 用泛型和 Supplier 抽取缓存流程

缓存旁路通常有三个读取分支：允许缓存且命中，允许缓存但未命中，禁用缓存直接加载。流程可以固定，真正的数据加载行为由调用方提供。

下面用单线程内的 Map 模拟缓存，便于直接观察执行顺序；它没有连接 Redis，也没有提供分布式并发、过期或异常容错能力：

```java
import java.util.HashMap;
import java.util.Map;
import java.util.function.Supplier;

public class CacheLoadingDemo {
    static <T> T getFromCacheOrLoad(
            Map<String, T> cache,
            String key,
            boolean useCache,
            Supplier<T> loader) {

        if (!useCache) {
            return loader.get();
        }

        T cached = cache.get(key);
        if (cached != null) {
            return cached;
        }

        T loaded = loader.get();
        if (loaded != null) {
            cache.put(key, loaded);
        }
        return loaded;
    }

    public static void main(String[] args) {
        Map<String, String> cache = new HashMap<>();
        Supplier<String> loadProductName = () -> {
            System.out.println("执行数据库查询（模拟）");
            return "机械键盘";
        };

        System.out.println("首次读取：" + getFromCacheOrLoad(
                cache, "product:42", true, loadProductName));
        System.out.println("再次读取：" + getFromCacheOrLoad(
                cache, "product:42", true, loadProductName));
        System.out.println("关闭缓存：" + getFromCacheOrLoad(
                cache, "product:42", false, loadProductName));
    }
}
```

输出为：

```text
执行数据库查询（模拟）
首次读取：机械键盘
再次读取：机械键盘
执行数据库查询（模拟）
关闭缓存：机械键盘
```

==方法声明中的 `<T>` 统一本次调用的缓存值、加载结果与返回类型；`Supplier<T>` 让不同查询动作复用相同流程。== 泛型负责类型复用，函数式接口负责行为复用，两者承担不同任务。

| 分支 | 是否读取缓存 | 是否执行 loader | 是否回填 |
| --- | --- | --- | --- |
| 缓存开启且命中 | 是 | 否 | 否 |
| 缓存开启但未命中 | 是 | 是 | 非空结果才回填 |
| 缓存关闭 | 否 | 是 | 否 |

调用方应把实际查询放进 Lambda 方法体。例如 `() -> dao.findName(id)` 描述查询动作；先执行 `String loaded = dao.findName(id)`，再传入 `() -> loaded`，已经提前访问数据库，缓存命中也无法省掉那次查询。

这个示例不缓存 null，空结果会在下次读取时重新加载。替换为 Redis 后，还需要明确序列化、TTL、缓存异常与并发回源策略；它们不是 Supplier 自带的能力。相关机制可结合[《Redis 与 Redisson 开发实践》](<Redis 与 Redisson 开发实践.md>)的缓存章节阅读。

## 六、异常与线程边界

### 1. 接口签名决定能抛出哪些受检异常

JDK 的 Supplier、Function 等核心接口没有声明 `throws Exception`。如果 Lambda 内的方法会抛出受检异常，就需要在适当位置处理，或者使用明确声明异常的自定义接口。例如：

```java
@FunctionalInterface
interface ThrowingSupplier<T> {
    T get() throws Exception;
}

static <T> T load(ThrowingSupplier<T> action) throws Exception {
    return action.get();
}
```

这里的接口把异常传递写入契约，接收方也必须处理或继续声明。`RuntimeException` 不受相同的编译期强制捕获规则约束；调用行为时仍可能向上传播。不能只在外围方法加 `throws Exception`，就让一个原本不允许受检异常的 Supplier 自动接受任意抛异常的 Lambda。

### 2. Runnable、Callable 与 Supplier

这些类型也经常以 Lambda 形式出现，但契约不同：

| 接口 | 入口 | 结果与异常约定 | 常见使用场景 |
| --- | --- | --- | --- |
| `Runnable` | `void run()` | 无返回值，未声明受检异常 | 描述一个待执行任务 |
| `Callable<V>` | `V call()` | 返回 V，允许抛 Exception | 描述一个有结果的任务 |
| `Supplier<T>` | `T get()` | 返回 T，未声明受检异常 | 按需提供数据或结果 |

任务接口的签名可以分别在 [Runnable](https://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html) 与 [Callable](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Callable.html) 官方 API 中核对。

==函数式接口只规定怎样调用行为；把行为提交到执行器，才引入调度、等待结果、取消和线程切换等任务语义。== 直接调用 Runnable 的 `run()` 同样是在当前线程执行。业务方法读取数据库、修改缓存或发送通知时，也仍然存在状态与副作用，采用 Lambda 不会把它们自动变成无副作用或线程安全的操作。

选择这种写法的重点，是把稳定流程与确实需要变化的行为分开。只有一个固定动作、没有复用需求的简单方法，不必为了使用 Lambda 再增加一层抽象；有明确名字的领域接口，也不必全部替换成通用 Function，以免业务含义被抹掉。

