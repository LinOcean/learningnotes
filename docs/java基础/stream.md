# Stream

- 流的一个核心好处是，它使得程序更加短小并且易于理解。

  ```java
  public class Randoms {
      public static void main(String[] args) {
          new Random(47).ints(5,20)
                  .distinct()
                  .limit(7)
                  .sorted()
                  .forEach(System.out::println);
      }
  }
  ```

  `ints()` 方法产生一个流并且 `ints()` 方法有多种方式的重载 — 两个参数限定了数值产生的边界。这将生成一个整数流。

- 声明式编程(*Declarative programming*):

  ```java
  public class ImperativeRandoms {
      public static void main(String[] args) {
          Random rand = new Random(47);
          SortedSet<Integer> rints = new TreeSet<>();
          while (rints.size() < 7) {
              int r = rand.nextInt(20);
              if (r < 5) continue;
              rints.add(r);
          }
          System.out.println(rints);
      }
  }
  ```

  流式编程采用内部迭代，这是流式编程的核心特性之一。这种机制使得编写的代码可读性更强，也更能利用多核处理器的优势。通过放弃对迭代过程的控制，我们把控制权交给并行化机制。

  另一个重要方面，流是懒加载的。这代表着它只在绝对必要时才计算。你可以将流看作“延迟列表”。由于计算延迟，流使我们能够表示非常大（甚至无限）的序列，而不需要考虑内存问题。

## 流支持

- 为了将流融入到现有的类库中，**java8**的解决方案是：在[接口中添加被 `default`修饰的方法。通过这种方案，设计者们可以将流式（*stream*）方法平滑地嵌入到现有类中。流方法预置的操作几乎已满足了我们平常所有的需求。流操作的类型有三种：创建流，修改流元素（中间操作， *Intermediate Operations*），消费流元素（终端操作， *Terminal Operations*）

## 流创建

- 使用**Stream.of**

  ```java
  public class StreamOf {
      public static void main(String[] args) {
          Stream.of(new Bubble(1), new Bubble(2), new Bubble(3))
                  .forEach(System.out::println);
          Stream.of("It's ", "a ", "wonderful ", "day ", "for ", "pie!")
                  .forEach(System.out::println);
          System.out.println();
          Stream.of(3.14159, 2.728, 1.618)
                  .forEach(System.out::println);
      }
  }
  ```

  ```java
  public class Bubble {
      public final int i;
      public Bubble(int i) {
          this.i = i;
      }
      @Override
      public String toString() {
          return "Bubble(" + i + ")";
      }
      private static int count = 0;
      public static Bubble bubbler() {
          return new Bubble(count++);
      }
  }
  ```

- 集合通过**Stream**来创建

  ```java
  public class CollectionToStream {
      public static void main(String[] args) {
          List<Bubble> bubbles = Arrays.asList(new Bubble(1),
                  new Bubble(2), new Bubble(3));
          System.out.println(bubbles.stream()
                  .mapToInt(b -> b.i)
                  .sum());
          Set<String> w = new HashSet<>(Arrays.asList(
                  "It's a  wonderful day for pie".split(" ")));
          w.stream().map(x -> x + " ")
                  .forEach(System.out::println);
          HashMap<String, Double> map = new HashMap<>();
          map.put("pi", 3.14159);
          map.put("e", 2.718);
          map.put("phi", 1.618);
          map.entrySet().stream()
                  .map(e -> e.getKey() + ": " + e.getValue())
                  .forEach(System.out::println);
      }
  }
  ```

  `mapToInt()` 方法将一个对象流（*object stream*）转换成为包含整型数字的 `IntStream`

### 随机数流

- 创建随机数流

  ```java
   public class RandomGenerators {
        public static <T> void show(Stream<T> stream) {
            stream.limit(4)
                    .forEach(System.out::println);
            System.out.println("----------------");
        }
        public static void main(String[] args) {
            Random rand = new Random(47);
            show(rand.ints().boxed());
            show(rand.longs().boxed());
            show(rand.doubles().boxed());
            // 控制上限和下限
            show(rand.ints(10, 20).boxed());
            show(rand.longs(10, 20).boxed());
            show(rand.doubles(10, 20).boxed());
            // 控制流大小
            show(rand.ints(2).boxed());
            show(rand.longs(2).boxed());
            show(rand.doubles(2).boxed());
            // 控制流大小和边界
            // 控制流的大小和界限
            show(rand.ints(3, 3, 9).boxed());
            show(rand.longs(3, 12, 22).boxed());
            show(rand.doubles(3, 11.5, 12.3).boxed());
        }
    }
  ```

  类型参数 `T` 可以是任何类型，所以这个方法对 **Integer**、**Long** 和 **Double** 类型都生效。但是 **Random** 类只能生成基本类型 **int**， **long**， **double** 的流。幸运的是， `boxed()` 流操作将会自动地把基本类型包装成为对应的装箱类型，从而使得 `show()` 能够接受流

- 读取配置文件创建

  ```java
  public class RandomWords implements Supplier<String> {
      List<String> words = new ArrayList<>();
      Random rand = new Random(47);
      public RandomWords(String fname) throws IOException {
          List<String> lines = Files.readAllLines(Paths.get(fname));
          for (String line : lines.subList(1, lines.size())) {
              for (String word : line.split("[ .?,]+")) {
                  words.add(word.toLowerCase(Locale.US));
              }
          }
      }
      @Override
      public String get() {
          return words.get(rand.nextInt(words.size()));
      }
      @Override
      public String toString() {
          return words.stream().collect(Collectors.joining(" "));
      }
      public static void main(String[] args) throws IOException {
          System.out.println(Stream.generate(
                  new RandomWords("D:\\Cheese.dat"))
                  .limit(10)
                  .collect(Collectors.joining(" ")));
      }
  }
  ```

  **Stream.**`generate()` 的用法，它可以把任意 `Supplier` 用于生成 `T` 类型的流。 **Collectors.**`joining()`，你将会得到一个 `String` 类型的结果，每个元素都根据 `joining()` 的参数来进行分割

### int 类型的范围

- `IntStream` 类提供了 `range()` 方法用于生成整型序列的流

  ```java
  public class Ranges {
      public static void main(String[] args) {
          //  The traditional way:
          int result = 0;
          for (int i = 0; i < 20; i++) {
              result += i;
          }
          System.out.println(result);
          // for-in with a range:
          result = 0;
          for (int i : range(10, 20).toArray()) {
              result += i;
          }
          System.out.println(result);
          //Use streams:
          System.out.println(range(10, 20).sum());
      }
  }
  ```

  循环：

  ```java
  public class Repeat {
      public static void repeat(int n, Runnable run) {
          range(0, n).forEach(i -> run.run());
      }
  }
  public class Looping {
      static void hi() {
          System.out.println("Say Hi!");
      }
      public static void main(String[] args) {
          Repeat.repeat(3, () -> System.out.println("Looping"));
          Repeat.repeat(2, Looping::hi);
      }
  }
  ```

### generate()创建流

- `generate()` 搭配 `Supplier` 使用

  ```java
  public class Duplicator {
      public static void main(String[] args) {
          Stream.generate(() -> "duplicate")
                  .limit(3)
                  .forEach(System.out::println);
      }
  }
  public class Bubbles {
      public static void main(String[] args) {
          Stream.generate(Bubble::bubbler)
                  .limit(5)
                  .forEach(System.out::println);
      }
  }
  
  ```

### iterate()

- **Stream.**`iterate()` 以种子（第一个参数）开头，并将其传给方法（第二个参数）。方法的结果将添加到流，并存储作为第一个参数用于下次调用 `iterate()`，依次类推。我们可以利用 `iterate()` 生成一个斐波那契数列

  ```java
  public class Fibonacci {
      int x = 1;
  
      Stream<Integer> numbers() {
          return Stream.iterate(0, i -> {
              int result = x + i;
              x = i;
              return result;
          });
      }
  
      public static void main(String[] args) {
          new Fibonacci().numbers()
                  .skip(20)
                  .limit(10)
                  .forEach(System.out::println);
      }
  }
  
  ```

### 流的构建者模式

- 在建造者设计模式（也称构造器模式）中，首先创建一个 `builder` 对象，传递给它多个构造器信息，最后执行“构造”。**Stream** 库提供了这样的 `Builder`

  ```java
  public class FileToWordsBuilder {
      Stream.Builder<String> builder = Stream.builder();
      public FileToWordsBuilder(String filePath) throws IOException {
          Files.lines(Paths.get(filePath)).skip(1).forEach(line -> {
              for (String w : line.split("[ .?,]+")) {
                  builder.add(w);
              }
          });
      }
      Stream<String> stream() {
          return builder.build();
      }
      public static void main(String[] args) throws IOException {
          new FileToWordsBuilder("D:\\Cheese.dat")
                  .stream()
                  .limit(7)
                  .map(w -> w + " ").forEach(System.out::print);
      }
  }
  ```

### Arrays

- 用于把数组转换成为流

  ```java
  public class Machine {
      public static void main(String[] args) {
          Arrays.stream(new Operations[]{
                  () -> Operations.show("Bing"),
                  () -> Operations.show("Crack"),
                  () -> Operations.show("Twist"),
                  () -> Operations.show("Pop")
          }).forEach(Operations::execute);
      }
  }
  public interface Operations {
      void execute();
      static void runOps(Operations ... ops){
          for (Operations op : ops) {
              op.execute();
          }
      }
      static void show(String msg){
          System.out.println(msg);
      }
  }
  ```

### 正则表达式

- `java.util.regex.Pattern` 中增加了一个新的方法 `splitAsStream()`。这个方法可以根据传入的公式将字符序列转化为流。但是有一个限制，输入只能是 **CharSequence**，因此不能将流作为 `splitAsStream()` 的参数。

  ```java
  public class FileToWordsRegexp {
      private String all;
  
      public FileToWordsRegexp(String path) throws IOException {
          all = Files.lines(Paths.get(path))
                  .skip(1)
                  .collect(Collectors.joining(" "));
      }
  
      public Stream<String> stream() {
          return Pattern.compile("[ .,?]+").splitAsStream(all);
      }
  
      public static void main(String[] args) throws IOException {
          FileToWordsRegexp fw = new FileToWordsRegexp("D:\\Cheese.dat");
          fw.stream()
                  .limit(7)
                  .map(w -> w + " ")
                  .forEach(System.out::print);
          fw.stream()
                  .skip(7)
                  .limit(2)
                  .map(w -> w + " ")
                  .forEach(System.out::print);
      }
  }
  
  ```

  这里有个限制，整个文件必须存储在内存中；在大多数情况下这并不是什么问题，但是这损失了流操作非常重要的优势：

  1. 流“不需要存储”。当然它们需要一些内部存储，但是这只是序列的一小部分，和持有整个序列并不相同。
  2. 它们是懒加载计算的。

## 中间操作

### 跟踪和调试

- `peek()` 操作的目的是帮助调试。它允许你无修改地查看流中的元素

  ```java
  public class Peeking {
      public static void main(String[] args) throws IOException {
          FileToWords.stream("D:\\Cheese.dat")
                  .skip(21)
                  .limit(4)
                  .map(w -> w + " ")
                  .peek(System.out::println)
                  .map(String::toUpperCase)
                  .peek(System.out::println)
                  .map(String::toLowerCase)
                  .forEach(System.out::println);
      }
  }
  public class FileToWords {
      public static Stream<String> stream(String filePath) throws IOException {
          return Files.lines(Paths.get(filePath))
                  .skip(1)
                  .flatMap(line -> Pattern.compile("\\W").splitAsStream(line));
      }
  }
  ```

### 流元素排序

- 传入一个 **Comparator** 参数

  ```java
  public class SortedComparator {
      public static void main(String[] args) throws IOException {
          FileToWords.stream("D:\\Cheese.dat")
                  .skip(10)
                  .limit(10)
                  .sorted(Comparator.reverseOrder())
                  .map(w -> w + " ")
                  .forEach(System.out::print);
      }
  }
  ```

### 移除元素

- `distinct()`用于消除流中的重复元素。

- `filter(Predicate)`根据传进来的条件进行过滤

  ```java
  public class Prime {
      public static Boolean isPrime(long n) {
          return LongStream.rangeClosed(2, (long) Math.sqrt(n))
                  .noneMatch(i -> n % i == 0);
      }
  
      public LongStream numbers() {
          return LongStream.iterate(2, i -> i + 1)
                  .filter(Prime::isPrime);
      }
  
      public static void main(String[] args) {
          new Prime().numbers()
                  .limit(10)
                  .forEach(n -> System.out.format("%d ", n));
          System.out.println();
          new Prime().numbers()
                  .skip(90)
                  .limit(10)
                  .forEach(n -> System.out.format("%d ", n));
      }
  }
  ```

### 应用函数到元素

- `map(Function)`：将函数操作应用在输入流的元素中，并将返回值传递到输出流中。

- `mapToInt(ToIntFunction)`：操作同上，但结果是 **IntStream**。

- `mapToLong(ToLongFunction)`：操作同上，但结果是 **LongStream**。

- `mapToDouble(ToDoubleFunction)`：操作同上，但结果是 **DoubleStream**。

  ```java
  public class FunctionMap {
      static String[] elements = {"12", "", "23", "45"};
    static Stream<String> testStream() {
          return Arrays.stream(elements);
      }
      static void test(String descr, Function<String, String> func) {
          System.out.println("--- ( " + descr + " )----");
          testStream().map(func).forEach(System.out::println);
      }
      public static void main(String[] args) {
          test("add brackets", s -> "[" + s + "]");
          test("Increment", s -> {
                      try {
                          return Integer.parseInt(s) + 1 + "";
                      }
                      catch(NumberFormatException e) {
                          return s;
                      }
                  }
          );
          test("Replace", s -> s.replace("2", "9"));
          test("Take last digit", s -> s.length() > 0 ?
                  s.charAt(s.length() - 1) + "" : s);
      }
  }
  ```

  `map()` 将一个字符串映射为另一个字符串，但是我们完全可以产生和接收类型完全不同的类型，从而改变流的数据类型。

  ```java
  public class FunctionMap2 {
      public static void main(String[] args) {
          Stream.of(1, 5, 7, 9, 11, 13).map(Numbered::new).forEach(System.out::println);
      }
  }
  class Numbered {
      final int n;
  
      public Numbered(int n) {
          this.n = n;
      }
      @Override
      public String toString() {
          return "Numbered(" + n + ")";
      }
  }
  ```

  使用合适的 `mapTo数值类型`

  ```java
  public class FunctionMap3 {
      public static void main(String[] args) {
          Stream.of("5","7","9")
                  .mapToInt(Integer::parseInt)
                  .forEach(n -> System.out.format("%d ", n));
          System.out.println();
          Stream.of("17", "19", "23")
                  .mapToLong(Long::parseLong)
                  .forEach(n -> System.out.format("%d ", n));
          System.out.println();
          Stream.of("17", "1.9", ".23")
                  .mapToDouble(Double::parseDouble)
                  .forEach(n -> System.out.format("%f ", n));
      }
  }
  ```

### 在map( )中组合流

- `flatMap()` 做了两件事：将产生流的函数应用在每个元素上（与 `map()` 所做的相同），然后将每个流都扁平化为元素，因而最终产生的仅仅是元素。

- `flatMap(Function)`：当 `Function` 产生流时使用。

- `flatMapToInt(Function)`：当 `Function` 产生 `IntStream` 时使用。

- `flatMapToLong(Function)`：当 `Function` 产生 `LongStream` 时使用。

- `flatMapToDouble(Function)`：当 `Function` 产生 `DoubleStream` 时使用。

  ```java
  public class StreamOfStreams {
      public static void main(String[] args) {
          Stream.of(1, 2, 3)
          .map(i -> Stream.of("Gonzo", "Kermit", "Beaker"))
          .map(e-> e.getClass().getName())
          .forEach(System.out::println);
      }
  }
  ```

  输出结果

  ```java
  java.util.stream.ReferencePipeline$Head
  java.util.stream.ReferencePipeline$Head
  java.util.stream.ReferencePipeline$Head
  ```

  使用`flatMap`

  ```java
  public class FlatMap {
      public static void main(String[] args) {
          Stream.of(1, 2, 3)
                  .flatMap(i -> Stream.of("Gonzo", "Kermit", "Beaker"))
                  .forEach(System.out::println);
      }
  }
  ```

  下面是另一个演示，我们从一个整数流开始，然后使用每一个整数去创建更多的随机数。

  ```java
  public class StreamOfRandoms {
      static Random rand = new Random(47);
      public static void main(String[] args) {
          Stream.of(1, 2, 3, 4, 5)
                  .flatMapToInt(i -> IntStream.concat
                          (rand.ints(0, 100)
                                  .limit(i), IntStream.of(-1)))
                  .forEach(n -> System.out.format("%d ", n));
      }
  }
  ```

## Optional 类

- **Optional** 可以处理流中返回`null`的情况。一些标准流操作返回 **Optional** 对象，因为它们并不能保证预期结果一定存在。

- `findFirst()`返回一个包含第一个元素的 **Optional** 对象，如果流为空则返回 **Optional.empty**

- `findAny()` 返回包含任意元素的 **Optional** 对象，如果流为空则返回 **Optional.empty**

- `max()` 和 `min()` 返回一个包含最大值或者最小值的 **Optional** 对象，如果流为空则返回 **Optional.empty**

  ```java
  public class OptionalsFromEmptyStreams {
      public static void main(String[] args) {
          System.out.println(Stream.<String>empty().findFirst());
          System.out.println(Stream.<String>empty().findAny());
          System.out.println(Stream.<String>empty().max(String.CASE_INSENSITIVE_ORDER));
          System.out.println(Stream.<String>empty().min(String.CASE_INSENSITIVE_ORDER));
          System.out.println(Stream.<String>empty().reduce((s1, s2) -> s1 + s2));
          System.out.println(IntStream.empty().average());
      }
  }
  ```

### 便利函数

- `ifPresent(Consumer)`：当值存在时调用 **Consumer**，否则什么也不做。
- `orElse(otherObject)`：如果值存在则直接返回，否则生成 **otherObject**。
- `orElseGet(Supplier)`：如果值存在则直接返回，否则使用 **Supplier** 函数生成一个可替代对象。
- `orElseThrow(Supplier)`：如果值存在直接返回，否则使用 **Supplier** 函数生成一个异常。

### 创建Optional

- `empty()`：生成一个空 **Optional**。
- `of(value)`：将一个非空值包装到 **Optional** 里。
- `ofNullable(value)`：针对一个可能为空的值，为空时自动生成 **Optional.empty**，否则将值包装在 **Optional** 中。

### Optional对象操作

- `filter(Predicate)`：将 **Predicate** 应用于 **Optional** 中的内容并返回结果。当 **Optional** 不满足 **Predicate** 时返回空。如果 **Optional** 为空，则直接返回。
- `map(Function)`：如果 **Optional** 不为空，应用 **Function** 于 **Optional** 中的内容，并返回结果。否则直接返回 **Optional.empty**。
- `flatMap(Function)`：同 `map()`，但是提供的映射函数将结果包装在 **Optional** 对象中，因此 `flatMap()` 不会在最后进行任何包装。

### Optional流

- 假设你的生成器可能产生 `null` 值，那么当用它来创建流时，你会自然地想到用 **Optional** 来包装元素

  ```java
  class Signal {
      private final String msg;
      public Signal(String msg) {
          this.msg = msg;
      }
      @Override
      public String toString() {
          return "Signal(" + msg + ")";
      }
      static Random rand = new Random(47);
      public static Signal morse() {
          switch (rand.nextInt(4)) {
              case 1:
                  return new Signal("dot");
              case 2:
                  return new Signal("dash");
              default:
                  return null;
          }
      }
      public static Stream<Optional<Signal>> stream() {
          return Stream.generate(Signal::morse).map(Optional::ofNullable);
      }
  }
  public class StreamOfOptionals {
      public static void main(String[] args) {
          Signal.stream().limit(10)
                  .forEach(System.out::println);
          System.out.println("--------------");
          Signal.stream()
                  .limit(10)
                  .filter(Optional::isPresent) // 对Optional.empty的情况进行了过滤
                  .map(Optional::get)
                  .forEach(System.out::println);
      }
  }
  ```

## 终端操作

### 数组

- `toArray()`：将流转换成适当类型的数组。

- `toArray(generator)`：在特殊情况下，生成自定义类型的数组。

- 下面为复用流产生的随机数

  ```java
  public class RandInts {
      private static int[] rints = new Random(47)
              .ints(0, 100)
              .limit(100)
              .toArray();
  
      public static IntStream rands() {
          return Arrays.stream(rints);
      }
  }
  ```

  上例将100个数值范围在 0 到 1000 之间的随机数流转换成为数组并将其存储在 `rints` 中。这样一来，每次调用 `rands()` 的时候可以重复获取相同的整数流。

### 循环

- `forEach(Consumer)`:无序操作，仅在引入并行流时才有意义

- `forEachOrdered(Consumer)`:保证`forEach`按照原始流顺序操作。

  ```java
  public class ForEach {
      static final int SZ = 14;
      public static void main(String[] args) {
          RandInts.rands().limit(14)
                  .forEach(n -> System.out.format("%d ", n));
          System.out.println();
          RandInts.rands().limit(14)
                  .parallel()
                  .forEach(n -> System.out.format("%d ", n));
          System.out.println();
          RandInts.rands().limit(14)
                  .parallel()
                  .forEachOrdered(n -> System.out.format("%d ", n));
      }
  }
  ```

  在第一个流中，未使用 `parallel()` ，所以 `rands()` 按照元素迭代出现的顺序显示结果；在第二个流中，引入`parallel()` ，即便流很小，输出的结果顺序也和前面不一样。这是由于多处理器并行操作的原因。多次运行测试，结果均不同。多处理器并行操作带来的非确定性因素造成了这样的结果。

  在最后一个流中，同时使用了 `parallel()` 和 `forEachOrdered()` 来强制保持原始流顺序。因此，对非并行流使用 `forEachOrdered()` 是没有任何影响的。

### 集合

- `collect(Collector)`：使用 **Collector** 收集流元素到结果集合中。

- `collect(Supplier, BiConsumer, BiConsumer)`：同上，第一个参数 **Supplier** 创建了一个新结果集合，第二个参数 **BiConsumer**将下一个元素包含到结果中，第三个参数 **BiConsumer** 用于将两个值组合起来。

- 假设需要将元素收集到`TreeSet`中，`Collectors`里面没有`toTreeSet()`

  ```java
  public class TreeSetOfWords {
      public static void main(String[] args) throws IOException {
          TreeSet<String> words = Files.lines(Paths.get("TreeSetOfWords.java"))
                  .flatMap(s -> Arrays.stream(s.split("\\W+")))
                  .filter(s -> s.matches("\\d+"))// No Number
                  .map(String::trim)
                  .filter(s -> s.length() > 2)
                  .limit(100)
                  .collect(Collectors.toCollection(TreeSet::new));
          System.out.println(words);
      }
  }
  ```

- 我们也可以在流中生成 **Map**

  ```java
  public class MapCollector {
      public static void main(String[] args) {
          Map<Integer, Character> map = new RandomPair()
                  .stream()
                  .limit(8)
                  .collect(
                          Collectors.toMap(Pair::getI, Pair::getC));
          System.out.println(map);
      }
  }
  
  class Pair {
      public final Character c;
      public final Integer i;
      public Pair(Character c, Integer i) {
          this.c = c;
          this.i = i;
      }
      public Character getC() {
          return c;
      }
      public Integer getI() {
          return i;
      }
      @Override
      public String toString() {
          return "Pair(" + c + ", " + i + ")";
      }
  }
  
  class RandomPair {
      Random rand = new Random(47);
      Iterator<Character> capChars = rand
              .ints(65, 91)
              .mapToObj(i -> (char) i)
              .iterator();
  
      public Stream<Pair> stream() {
          return rand.
                  ints(100, 1000)
                  .distinct()
                  .mapToObj(i -> new Pair(capChars.next(), i));
      }
  }
  ```

  这是组合多个流以生成新的对象流的唯一方法。

- 大多数情况下，`java.util.stream.Collectors` 中预设的 **Collector** 就能满足我们的要求。除此之外，你还可以使用第二种形式的 `collect()`

  ```java
  public class SpecialCollector {
      public static void main(String[] args) throws IOException {
          ArrayList<String> words = FileToWords.stream("D:\\Cheese.dat")
                  .collect(ArrayList::new,
                          ArrayList::add,
                          ArrayList::addAll);
          words.stream()
                  .filter(s -> s.equals("cheese"))
                  .forEach(System.out::println);
      }
  }
  public class FileToWords {
      public static Stream<String> stream(String filePath) throws IOException {
          return Files.lines(Paths.get(filePath))
                  .skip(1)
                  .flatMap(line -> Pattern.compile("\\W").splitAsStream(line));
      }
  }
  ```

### 组合

- `reduce(BinaryOperator)`：使用 **BinaryOperator** 来组合所有流中的元素。因为流可能为空，其返回值为 **Optional**。

- `reduce(identity, BinaryOperator)`：功能同上，但是使用 **identity** 作为其组合的初始值。因此如果流为空，**identity** 就是结果。

- `reduce(identity, BiFunction, BinaryOperator)`：更复杂的使用形式s，这里把它包含在内，因为它可以提高效率。通常，我们可以显式地组合 `map()` 和 `reduce()` 来更简单的表达它。

  ```java
  public class Reduce {
      public static void main(String[] args) {
          Stream.generate(Frobnitz::supply)
                  .limit(10)
                  .peek(System.out::println)
                  .reduce((fr0, fr1) -> fr0.size < 50 ? fr0 : fr1)
                  .ifPresent(System.out::println);
      }
  }
  
  class Frobnitz {
      int size;
      public Frobnitz(int size) {
          this.size = size;
      }
      @Override
      public String toString() {
          return "Frobnitz(" + size + ")";
      }
      static Random rand = new Random(47);
      static final int BOUND = 100;
      static Frobnitz supply() {
          return new Frobnitz(rand.nextInt(BOUND));
      }
  }
  
  ```

  **Lambda**表达式中的第一个参数 `fr0` 是上一次调用 `reduce()` 的结果。而第二个参数 `fr1` 是从流传递过来的值。

### 匹配

- `allMatch(Predicate)` ：如果流的每个元素根据提供的 **Predicate** 都返回 true 时，结果返回为 true。在第一个 false 时，则停止执行计算。

- `anyMatch(Predicate)`：如果流中的任意一个元素根据提供的 **Predicate** 返回 true 时，结果返回为 true。在第一个 false 是停止执行计算。

- `noneMatch(Predicate)`：如果流的每个元素根据提供的 **Predicate** 都返回 false 时，结果返回为 true。在第一个 true 时停止执行计算。

  ```java
  public class Matching {
      static void show(Matcher match, int val) {
          System.out.println(
                  match.test(
                          IntStream.rangeClosed(1, 9)
                                  .boxed()
                                  .peek(n -> System.out.format("%d ", n)),
                          n -> n < val)
          );
      }
  
      public static void main(String[] args) {
          show(Stream::allMatch, 10);
          show(Stream::allMatch, 4);
          show(Stream::anyMatch, 2);
          show(Stream::anyMatch, 0);
          show(Stream::noneMatch, 5);
          show(Stream::noneMatch, 0);
      }
  }
  
  interface Matcher extends BiPredicate<Stream<Integer>, Predicate<Integer>> {
  }
  ```

  **BiPredicate** 是一个二元谓词，它只能接受两个参数且只返回 true 或者 false。它的第一个参数是我们要测试的流，第二个参数是一个谓词 **Predicate**。

### 查找

- `findFirst()`：返回第一个流元素的 **Optional**，如果流为空返回 **Optional.empty**。

- `findAny()`：返回含有任意流元素的 **Optional**，如果流为空返回 **Optional.empty**。

  ```java
  public class SelectElement {
      public static void main(String[] args) {
          System.out.println(rands().findFirst().getAsInt());
          System.out.println(
                  rands().parallel().findFirst().getAsInt());
          System.out.println(rands().findAny().getAsInt());
          System.out.println(
                  rands().parallel().findAny().getAsInt());
      }
  }
  
  ```

  `findFirst()` 无论流是否为并行化的，总是会选择流中的第一个元素。对于非并行流，`findAny()`会选择流中的第一个元素（即使从定义上来看是选择任意元素）。在这个例子中，我们使用 `parallel()` 来并行流从而引入 `findAny()` 选择非第一个流元素的可能性。

- 如果必须选择流中最后一个元素，那就使用 `reduce()`

  ```java
  public class LastElement {
      public static void main(String[] args) {
          OptionalInt last = IntStream.range(10, 20)
                  .reduce((n1, n2) -> n2);//n1为执行reduce产生的值，n2为再次传进来的值
          System.out.println(last.orElse(-1));
          Optional<String> lastObj = Stream
                  .of("one", "two", "three")
                  .reduce((n1, n2) -> n2);
          System.out.println(lastObj.orElse("Nothing there!"));
      }
  }
  ```

### 信息

- `count()`：流中的元素个数。

- `max(Comparator)`：根据所传入的 **Comparator** 所决定的“最大”元素。

- `min(Comparator)`：根据所传入的 **Comparator** 所决定的“最小”元素。

  ```java
  public class Informational {
      public static void main(String[] args) throws IOException {
          System.out.println(FileToWords
                  .stream("Cheese.dat").count());
          System.out.println(FileToWords
                  .stream("Cheese.dat")
                  .min(String.CASE_INSENSITIVE_ORDER)
                  .orElse("NONE")
          );
          System.out.println(
                  FileToWords.stream("Cheese.dat")
                          .max(String.CASE_INSENSITIVE_ORDER)
                          .orElse("NONE")
          );
      }
  }
  ```

### 数字流信息

- `average()` ：求取流元素平均值。

- `max()` 和 `min()`：数值流操作无需 **Comparator**。

- `sum()`：对所有流元素进行求和。

  ```java
  import java.util.stream.*;
  import static streams.RandInts.*;
  public class NumericStreamInfo {
      public static void main(String[] args) {
          System.out.println(rands().average().getAsDouble());
          System.out.println(rands().max().getAsInt());
          System.out.println(rands().min().getAsInt());
          System.out.println(rands().sum());
          System.out.println(rands().summaryStatistics());
      }
  }
  ```

  