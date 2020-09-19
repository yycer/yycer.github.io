---
layout: post
title: 异常处理机制
category: java-basic
tags: [exception]
excerpt: Java异常处理机制
---

## Update at 2020_0919  

补充两点：`try...catch...finally`语句块的执行流程，并梳理常见异常。  

### Flow Control  

先来看下示意图：  

![](https://yyc-images.oss-cn-beijing.aliyuncs.com/java_exception_flow_control.png)  

首先，我们需要明确这三个语句块的职责分别是什么？  

`try`语句块中包含可能会出现异常的代码，注意粒度切分得尽可能要细。  

`catch`的作用是处理异常，若当前场景下无法处理，则可以记录日志，并向上抛出。  

`finally`语句块一般用于释放某些连接资源。  

来看个例子：  


``` java
public class ExceptionTest {

    public static void main(String[] args) {

        try {
            System.out.println("Start try block.");
            int a = 10 / 0;
            System.out.println("End try block.");
        } catch (ArithmeticException e) {
            System.out.println("Start catch block");
            e.printStackTrace();
        } finally {
            System.out.println("Start finally block");
        }
    }
}

// Start try block.
// Start catch block
// Start finally block
// java.lang.ArithmeticException: / by zero
//     at com.frankie.demo.exception.ExceptionTest.main(ExceptionTest.java:13)
```

但是，当`finally`语句块中包含`return`时，情况就会变得比较复杂，   

它会覆盖`try`和`catch`语句块中的`return`，对于这一点知道即可，实际场景中应严格避免这种情况！  

``` java
String ans2 = allContainsReturn("a");
System.out.println("ans2 = " + ans2);
System.out.println("-----------------------");
String ans3 = allContainsReturn("b");
System.out.println("ans3 = " + ans3);

private static String allContainsReturn(String name){
    try {
        if (name.endsWith("a")){
            System.out.println(name + " from try block.");
        } else{
            int a = 10 / 0;
        }
        return "Return form try block";
    } catch (Exception e){
        System.out.println(name + " from catch block.");
        return "Return form catch block";
    } finally {
        return "Return form finally block";
    }
}

// a from try block.
// ans2 = Return form finally block
// -----------------------
// b from catch block.
// ans3 = Return form finally block
```


### Common Exceptions   

- ArithmeticException  

``` java
// Exception in thread "main" java.lang.ArithmeticException: / by zero
int a = 10 / 0;
```


- ArrayIndexOutOfBoundsException  

``` java
int[] nums = {1, 2, 3};
// java.lang.ArrayIndexOutOfBoundsException: Index 3 out of bounds for length 3
int t = nums[3];
```


- NullPointerException  

``` java
String s = null;
// java.lang.NullPointerException
s.equals("a");
```


- FileNotFoundException & IOException  

``` java
try {
    // java.io.FileNotFoundException: path (系统找不到指定的文件。)
    new FileReader(new File("path"));
} catch (FileNotFoundException e){
    e.printStackTrace();
}
```


- InterruptedException  

``` java
class ChildThread extends Thread{

    @Override
    public void run(){
        try {
            // java.lang.InterruptedException: sleep interrupted
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

ChildThread ct = new ChildThread();
ct.start();
// java.lang.InterruptedException: sleep interrupted
ct.interrupt();
```


- DateTimeParseException  

``` java
String date = "1234";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
// java.time.format.DateTimeParseException: Text '1234' could not be parsed at index 4
LocalDate ld = LocalDate.parse(date, formatter);
```


- NumberFormatException  

``` java
String s = "10a";
// java.lang.NumberFormatException: For input string: "10a"
int n = Integer.parseInt(s); 
```


## 异常类型
#### 1. Exception
>Exception主要分为两种：Runtime Exception、Checked（Compile） Exception。

常见的Runtime Exception，有: NullPointerException、ArithmeticException...

常见的Checked（Compile） Exception，有: IOException、FileNotFoundException...

所谓Checked Exception就是在编译期间，你必须对某条、或多条语句进行异常处理，如： `try`...`catch`...、`throws`语句。

##### 下面介绍下Checked Exception的优缺点：

- 特点与优点： `Java专有`，体现Java的设计哲学，没有完善错误处理的代码根本就不会执行的机会。  

- 缺点：
必须显式捕捉并处理异常，或显式声明抛出异常，增加程序复杂度。
若显式抛出异常，则会增加方法签名与异常的耦合度。  

#### 2. Error
>Error主要表示一些虚拟机内部错误，如：动态链接失败。  

<br>
##  异常处理规则
- 程序的`可读性`：避免过度使用异常处理代码，减少方法签名与异常的耦合度。  

- 异常的`原始性`：捕获并保留原始异常信息。  

- 异常的`针对性`：根据业务需求决定如何处理异常，比如：
当你检查商品库存时发生异常，此时就应终止此次调用，并告诉上层用户详细、明确的原因。
但当你获取用户头像失败时，因为该操作不影响整体订单、支付流程，所以不需要终止此次调用，可与上层用户协商处理，比如：返回一个空字符串。  

<br>
##  相关问题  

> 1.throw 与 throws的区别？  

- 位置：`throws`位于方法签名，而`throw`位于函数体内。
- 语法格式：
`throws`后面跟的是异常类，且一次可以跟多个，只需要以逗号分隔。而`throw`后面跟着的是异常实例，且一次只能跟一个。
- 命中率：
`throws`只是做个防守，可能并不会真正执行。但一旦执行到`throw`语句，必定抛出异常。

> 2.为什么要有异常处理机制？  

- 无法穷举所有的异常情况。若异常处理的代码过多，会导致程序可读性很差。

> 3.为什么要把原始异常封装一层？  

- 安全性，防止恶意用户获得系统内部信息。
- 对上层用户更加友好，让其更加明确、详细地知道异常原因。

> 4.为什么有那么多类需要实现`Closeable`或`AutoCloseable`接口？  

- `Java9`增强了自动关闭资源的`try`语句。

```java
public class ExceptionTest {
    public static void readFile(){
        // Close the resources automatically in try statement.
        try(BufferedReader bufferedReader = new BufferedReader(new FileReader("justForTest.txt")))
        {
            System.out.println(bufferedReader.readLine());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

@Test
public void readFileTest(){
    ExceptionTest.readFile(); // Just for test.
}
```
<br>
<br>
##  实践
 

```java

@Getter
// Custom exception class.
public class ServiceException extends RuntimeException {

    private HttpStatus status;
    private ResultCode resultCode;
    private Object     errorData;

    private ServiceException(HttpStatus status, ResultCode resultCode, Object errorData){
        this.status     = status;
        this.resultCode = resultCode;
        this.errorData  = errorData;
    }

    public static ServiceException badRequest(ResultCode resultCode, Object errorData){
        return new ServiceException(HttpStatus.BAD_REQUEST, resultCode, errorData);
    }
}

@Getter
public enum ResultCode {
    // errorCode
    SUCCESS(0, "SUCCESS"),
    INVALID_PARAMETER(600, "invalid parameter");

    private final int errorCode;
    private final String errorData;

    ResultCode(int errorCode, String errorData){
        this.errorCode = errorCode;
        this.errorData = errorData;
    }
}

@ControllerAdvice
public class GlobalErrorHandler {

    @ExceptionHandler(ServiceException.class)
    public ResponseEntity<ErrorResponse> handleServiceException(ServiceException ex){
        return ResponseEntity
                .status(ex.getStatus())
                .body(ErrorResponse.failed(ex.getResultCode(), ex.getErrorData()));
    }
}


@ApiModel
@Getter
public class ErrorResponse<T> implements Serializable {

    private static final long serialVersionUID = -2254339918462802230L;

    private final int    errorCode;
    private final String errorMsg;
    private final T      errorData;

    private ErrorResponse(ResultCode resultCode, T errorData) {
        this.errorCode = resultCode.getErrorCode();
        this.errorMsg  = resultCode.getErrorData();
        this.errorData = errorData;
    }

    public static <T> ErrorResponse<T> failed(ResultCode resultCode, T data){
        return new ErrorResponse(resultCode, data);
    }
}

@RestController
public class OrderController {

    @GetMapping(value = "/v1/orders/{order_id}"/*, produces = {"application/toString", "application/json"}*/)
    public Order getOrder(@PathVariable("order_id") @NotBlank String orderId){

        Order order     = new Order();
        BigDecimal total = new BigDecimal(-1.00, new MathContext(2, RoundingMode.HALF_UP));

        if (total.compareTo(BigDecimal.ZERO) <= 0){
            throw ServiceException.badRequest(ResultCode.INVALID_PARAMETER,
                    "Total is less than zero!");
        }
        order.setOrderId(orderId);
        order.setTotal(total);
        return order;
    }
}
```
<br>
##  参考
- 疯狂Java讲义（第十章 - 异常处理）  
- JAVA核心知识点整理  
- [Exception Handling – try catch Java blocks](https://javabeginnerstutorial.com/core-java-tutorial/exception-handling-try-catch-java/#:~:text=In%20case%20a%20finally%20block,back%20to%20the%20return%20statement.){:target="_blank"}  
- [Common Java Exceptions](https://www.baeldung.com/java-common-exceptions){:target="_blank"}  