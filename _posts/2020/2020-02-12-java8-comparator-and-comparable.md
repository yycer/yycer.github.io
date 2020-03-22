---
layout: post
title: Java8 - 比较器
category: java-basic
tags: [spring, java]
excerpt: Java8 Comparator & Comparable
---

## `2020_0322更新`  

这次更新主要是为了理清楚`Comparable`和`Comparator`的区别。  

先来说`Comparable`，  

首先，它是一个接口，并且只有一个实例方法：  

> public int compareTo(T o);  

一般的用法是将某个类实现`Comparable`接口，并重写`compareTo()`方法，比较指定的某个实例变量。  

举个例子：  

``` java
@Getter
public class Product implements Comparable<Product>{

    private String name;
    private int    price;

    public Product(String name, int price) {
        this.name = name;
        this.price = price;
    }

    @Override
    public int compareTo(Product o) {
        return this.getPrice() - o.getPrice();
    }

    @Override
    public String toString() {
        return "Product{" +
                "name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}

@Test
void comparableTest(){
    Product a = new Product("A", 50);
    Product b = new Product("B", 10);
    Product c = new Product("C", 15);

    ArrayList<Product> products = new ArrayList<>();
    products.add(a);
    products.add(b);
    products.add(c);
    Collections.sort(products);
    System.out.println(products);
    // [Product{name='B', price=10}, Product{name='C', price=15}, Product{name='A', price=50}]
}
```

如果我想比较的对象没有实现`Comparable`接口，怎么办？  

这个时候就可以用到`Comparator`。  

它也是个接口，而且还是`函数式接口`，并且有更为丰富的方法，如：`reversed()`、`thenComparing()`等。  

举个例子：  


``` java
@Getter
public class Order {

    private String userName;
    private double total;

    public Order(String userName, double total) {
        this.userName = userName;
        this.total = total;
    }

    @Override
    public String toString() {
        return "Order{" +
                "userName='" + userName + '\'' +
                ", total=" + total +
                '}';
    }
}

public class OrderComparator implements Comparator<Order> {
    @Override
    public int compare(Order o1, Order o2) {
        return (int) (o1.getTotal() - o2.getTotal());
    }
}

@Test
void comparatorTest1(){
    Order a = new Order("a", 200.00d);
    Order b = new Order("b", 200.00d);
    Order c = new Order("c", 188.00d);
    Order d = new Order("d", 300.00d);

    ArrayList<Order> orders = new ArrayList<>();
    orders.add(a);
    orders.add(b);
    orders.add(c);
    orders.add(d);

    Collections.sort(orders, new OrderComparator());
    System.out.println(orders);
    // [Order{userName='c', total=188.0}, Order{userName='a', total=200.0}, Order{userName='b', total=200.0}, Order{userName='d', total=300.0}]
}

```

我们也可以使用`ArrayList`中的`sort`方法：  

``` java
orders.sort(new OrderComparator());
System.out.println(orders);
```

我们还可以使用匿名类：  

``` java
orders.sort(new Comparator<Order>() {
    @Override
    public int compare(Order o1, Order o2) {
        return (int) (o1.getTotal() - o2.getTotal());
    }
});
System.out.println(orders);
```

继续，`Lambda`表达式：  

``` java
orders.sort((o1, o2) -> (int) (o1.getTotal() - o2.getTotal()));
System.out.println(orders);
```

再来，`comparing()`和`方法引用`：  


``` java
orders.sort(Comparator.comparing(Order::getTotal));
```

最后再来玩下`reversed()`和`thenComparing()`：  

``` java
orders.sort(Comparator.comparing(Order::getTotal)
    .reversed()
    .thenComparing(Order::getUserName));
System.out.println(orders);
// [Order{userName='d', total=300.0}, Order{userName='a', total=200.0}, Order{userName='b', total=200.0}, Order{userName='c', total=188.0}]
```

## 准备数据  

``` java
List<Apple> inventory = Arrays.asList(
        new Apple(80,"green"),
        new Apple(155, "green"),
        new Apple(120, "red"),
        new Apple(80, "red"));
```


## 正序排序  

``` java
@Test
public void complexComparatorLambdaTest(){
    List<Apple> sortedByWeight = inventory.stream()
            .sorted(Comparator.comparing(Apple::getWeight))
            .collect(Collectors.toList());
    System.out.println(sortedByWeight);

    // [Apple{color='green', weight=80}, Apple{color='red', weight=80}, Apple{color='red', weight=120}, Apple{color='green', weight=155}]
}

```


## 倒序排序  

``` java
@Test
public void complexComparatorLambdaTest(){
     List<Apple> sortedByWeightReversely = inventory.stream()
                .sorted(Comparator.comparing(Apple::getWeight).reversed())
                .collect(Collectors.toList());
    System.out.println(sortedByWeightReversely);

    // [Apple{color='green', weight=155}, Apple{color='red', weight=120}, Apple{color='green', weight=80}, Apple{color='red', weight=80}]
}
```


## 比较器链  

比如： 先根据`重量正序`，然后根据`颜色倒序`。  


``` java
@Test
public void complexComparatorLambdaTest(){
    List<Apple> sortedByWeightAndColor = inventory.stream()
                .sorted(Comparator.comparing(Apple::getWeight)
                                  .thenComparing(Comparator.comparing(Apple::getColor)
                                  .reversed()))
                .collect(Collectors.toList());
    System.out.println(sortedByWeightAndColor);

    // [Apple{color='red', weight=80}, Apple{color='green', weight=80}, Apple{color='red', weight=120}, Apple{color='green', weight=155}]
}
```


## `Reference`  
- `《Java8实战》 - 3.8.1 比较器复合`  

