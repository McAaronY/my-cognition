# 从 Java 8 到 Java 23 的变化说明书

## 一、引言

Java 自 1995 年问世以来，始终在持续演进，为开发者提供更为强大、高效且便捷的编程体验。自 Java 8 发布之后，Java 的版本迭代节奏加快，每半年便会推出新的版本。本说明书将详细阐述从 Java 8 至 Java 23 这期间各个版本的关键变化，并通过示例辅助理解。

## 二、Java 8 的重大变革

### （一）Lambda 表达式

Lambda 表达式极大地简化了匿名内部类的写法，使代码更为简洁、易读。

**示例**：对整数列表进行排序

```
import java.util.Arrays;

import java.util.List;

public class LambdaExample {

   public static void main(String[] args) {

       List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5);

       numbers.sort((a, b) -> a - b);

       System.out.println(numbers);

   }

}
```

### （二）Stream API

Stream API 提供了一种高效处理集合数据的方式，支持链式调用，可实现诸如过滤、映射、归约等操作。

**示例**：计算整数列表中偶数的平方和

```
import java.util.Arrays;

import java.util.List;

import java.util.stream.Collectors;

public class StreamExample {

   public static void main(String[] args) {

       List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

       int sumOfSquares = numbers.stream()

              .filter(n -> n % 2 == 0)

              .mapToInt(n -> n * n)

              .sum();

       System.out.println(sumOfSquares);

   }

}
```

### （三）默认方法和静态方法

接口中能够定义默认方法和静态方法，为接口的演进提供了更多灵活性，避免了在接口添加新方法时导致所有实现类都需修改的问题。

**示例**：在接口中定义默认方法

```
interface MyInterface {

   default void sayHello() {

       System.out.println("Hello from MyInterface");

   }

}

class MyClass implements MyInterface {

   // 无需实现sayHello方法，可直接使用接口的默认实现

}
```

## 三、Java 9 至 Java 17 的主要变化

### （一）Java 9

1.  **模块系统（Jigsaw）**：Java 9 引入了模块系统，用于更好地管理大型项目的依赖关系和封装。模块信息通过`module - info.java`文件进行描述。

    **示例**：定义一个简单的模块

```
module mymodule {

   exports com.example.mypackage;

}
```

1.  **集合工厂方法**：为`List`、`Set`、`Map`等集合接口新增了`of`方法，用于创建不可变集合。

    **示例**：创建不可变列表

```
import java.util.List;

public class ImmutableListExample {

   public static void main(String[] args) {

       List<String> names = List.of("Alice", "Bob", "Charlie");

       // names.add("David"); // 这行代码会抛出UnsupportedOperationException异常

       System.out.println(names);

   }

}
```

### （二）Java 10

1.  **局部变量类型推断（var）**：允许在声明局部变量时使用`var`关键字，由编译器自动推断变量类型。

    **示例**：使用`var`声明局部变量

```
public class VarExample {

   public static void main(String[] args) {

       var message = "Hello, Java 10";

       System.out.println(message);

   }

}
```

1.  **应用程序类数据共享（CDS）**：通过共享类数据，减少了 JVM 启动时加载类的时间，提升了程序的启动速度。

### （三）Java 11

1.  **性能提升**：通过优化垃圾回收等机制，Java 11 的性能得到显著提升。例如，G1 垃圾收集器在 Java 11 中得到进一步优化。

2.  **字符串增强**：`String`类新增了诸如`isBlank`、`strip`、`stripLeading`、`stripTrailing`等方法。

    **示例**：使用`isBlank`方法判断字符串是否为空或仅包含空白字符

```
public class StringEnhancementExample {

   public static void main(String[] args) {

       String str1 = "   ";

       String str2 = "Hello";

       System.out.println(str1.isBlank()); // true

       System.out.println(str2.isBlank()); // false

   }

}
```

1.  **HTTP Client API**：提供了全新的 HTTP 客户端 API，支持同步和异步请求。

    **示例**：发送同步 HTTP GET 请求

```
import java.io.IOException;

import java.net.URI;

import java.net.http.HttpClient;

import java.net.http.HttpRequest;

import java.net.http.HttpResponse;

public class HttpClientExample {

   public static void main(String[] args) throws IOException, InterruptedException {

       HttpClient client = HttpClient.newHttpClient();

       HttpRequest request = HttpRequest.newBuilder()

              .uri(URI.create("https://example.com"))

              .build();

       HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

       System.out.println(response.body());

   }

}
```

### （四）Java 12

1.  **Switch 表达式的改进**：Java 12 中的 Switch 表达式支持了更简洁的语法，并且可以返回值。

    **示例**：使用改进后的 Switch 表达式

```
public class SwitchExpressionExample {

   public static void main(String[] args) {

       int dayOfWeek = 3;

       String dayName = switch (dayOfWeek) {

           case 1 -> "Monday";

           case 2 -> "Tuesday";

           case 3 -> "Wednesday";

           default -> "Unknown";

       };

       System.out.println(dayName);

   }

}
```

### （五）Java 13

1.  **文本块（Text Blocks）**：文本块使得处理多行字符串更加容易，无需再使用繁琐的转义字符。

    **示例**：使用文本块表示 HTML 内容

```
public class TextBlockExample {

   public static void main(String[] args) {

       String html = """

                     <html>

                         <body>

                            <p>Hello, Java 13</p>

                        </body>

                    </html>

                     """;

       System.out.println(html);

   }

}
```

### （六）Java 14

1.  **Pattern Matching for instanceof（预览）**：该特性使得在使用`instanceof`进行类型判断后，可以直接进行类型转换，代码更为简洁。

    **示例**：使用`instanceof`模式匹配

```
public class InstanceOfPatternMatchingExample {

   public static void main(String[] args) {

       Object obj = "Hello";

       if (obj instanceof String s) {

           System.out.println(s.length());

       }

   }

}
```

### （七）Java 15

1.  **Sealed Classes（密封类）**：密封类限制了其他类对其的继承，增强了代码的安全性和可维护性。

    **示例**：定义密封类及其子类

```
public sealed class Shape permits Circle, Rectangle {

}

final class Circle extends Shape {

}

final class Rectangle extends Shape {

}
```

### （八）Java 16

1.  **Records（记录）**：记录是一种不可变的数据载体，简化了 POJO（Plain Old Java Object）类的定义。

    **示例**：定义记录

```
public record Person(String name, int age) {

}
```

### （九）Java 17

1.  **Switch 表达式的模式匹配（最终版）**：在 Java 17 中，Switch 表达式的模式匹配特性正式成为标准特性，功能更为强大。

    **示例**：使用 Switch 表达式的模式匹配处理不同类型的图形

```
public sealed class Shape permits Circle, Rectangle {

}

final class Circle extends Shape {

   private final double radius;

   public Circle(double radius) {

       this.radius = radius;

   }

   public double getRadius() {

       return radius;

   }

}

final class Rectangle extends Shape {

   private final double width;

   private final double height;

   public Rectangle(double width, double height) {

       this.width = width;

       this.height = height;

   }

   public double getWidth() {

       return width;

   }

   public double getHeight() {

       return height;

   }

}

public class SwitchPatternMatchingShapeExample {

   public static void main(String[] args) {

       Shape shape = new Circle(5.0);

       double area = switch (shape) {

           case Circle c -> Math.PI * c.getRadius() * c.getRadius();

           case Rectangle r -> r.getWidth() * r.getHeight();

       };

       System.out.println(area);

   }

}
```

1.  **弃用 Finalization**：Java 17 弃用了对象终结方法（`finalize`方法），鼓励使用其他资源管理方式。

## 四、Java 18 至 Java 23 的新特性

### （一）Java 18

1.  **默认字符集为 UTF - 8**：Java 18 将默认字符集设置为 UTF - 8，减少了字符编码相关的问题。

2.  **简易 Web 服务器**：引入了简易的 Web 服务器，方便开发人员进行快速测试和原型开发。

    **示例**：启动一个简单的 Web 服务器

```
import com.sun.net.httpserver.HttpExchange;

import com.sun.net.httpserver.HttpHandler;

import com.sun.net.httpserver.HttpServer;

import java.io.IOException;

import java.io.OutputStream;

import java.net.InetSocketAddress;

public class SimpleWebServerExample {

   public static void main(String[] args) throws IOException {

       HttpServer server = HttpServer.create(new InetSocketAddress(8000), 0);

       server.createContext("/", new MyHandler());

       server.start();

       System.out.println("Server started on port 8000");

   }

   static class MyHandler implements HttpHandler {

       @Override

       public void handle(HttpExchange exchange) throws IOException {

           String response = "Hello, from Java 18 Simple Web Server!";

           exchange.sendResponseHeaders(200, response.length());

           OutputStream os = exchange.getResponseBody();

           os.write(response.getBytes());

           os.close();

       }

   }

}
```

### （二）Java 19

1.  **Vector API（第四轮孵化）**：用于高效处理向量计算，为数值计算等领域提供了更强大的支持。

2.  **外部函数 & 内存 API（预览）**：允许 Java 程序安全高效地访问外部存储器，拓展了 Java 与外部系统的交互能力。

### （三）Java 20

1.  **结构化并发（预览）**：通过引入结构化并发，简化了并发编程，将不同线程中运行的相关任务组视为单个工作单元，提升了错误处理和取消操作的便利性，增强了程序的可靠性和可观测性。

### （四）Java 21

1.  **字符串模板（预览）**：字符串模板增强了 Java 编程语言，字符串常量中可包含嵌入式表达式，这些表达式在运行时会被计算和校验。

    **示例**：使用字符串模板

```
public class StringTemplateExample {

   public static void main(String[] args) {

       int num = 10;

       String result = """

                     The number is ${num}, and its square is ${num * num}

                     """;

       System.out.println(result);

   }

}
```

### （五）Java 22

1.  **类文件 API（预览）**：提供了对类文件进行操作的 API，方便开发人员在字节码层面进行一些高级操作。

2.  **流聚合器（预览）**：为流操作提供了更强大的聚合功能，使得处理流数据时可以进行更复杂的聚合操作。

### （六）Java 23（计划特性）

1.  **模式、instanceof 和 switch 中的原始类型（预览）**：该特性将进一步拓展模式匹配在原始类型上的应用，使得在`instanceof`和`switch`语句中对原始类型的处理更加灵活和强大。

## 五、总结

从 Java 8 到 Java 23，Java 语言在语法、性能、库以及编程模型等多个方面都发生了重大变化。这些变化旨在提升开发人员的生产力，增强 Java 在不同领域的竞争力，如大数据处理、并发编程、与外部系统交互等。开发者应根据项目需求和自身技术栈，适时采用新特性，以充分发挥 Java 的优势，构建出更为高效、可靠的应用程序。
