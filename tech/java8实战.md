## chapter 1

Java8的核心新特性：Lambda（匿名函数）、流、默认方法

谓词（predicate）在数学上常常用来代表一个类似函数的东西，它接受一个参数值，并返回true或false。



## chapter 2

Collection主要是为了存储和访问数据，而Stream则主要用于描述对数据的计算



## chapter 3

Lambda的基本语法是: 

```java
(parameters) -> expression

或者(请注意花括号)

(parameters) -> {
  statements;
}

```

函数式接口 (@FunctionalInterface注解)

* 函数式接口就是只定义一个抽象方法的接口

* 函数式接口的抽象方法的签名基本上就是Lambda表达式的签名，我们将这种抽象方法叫做**函数描述符**

* 只要知道Lambda表达式可以被赋给一个变量，或传递给一个接受函数式接口作为参数的方法就好了，当然这个**Lambda表达式的签名**要和函数式接口的抽象方法一样。

* Java 8引入的几个新的函数式接口：**Predicate**、**Consumer**和**Function**

* Apple::getWeight就是引用了Apple类中定义的方法getWeight。请记住，不需要括号，因为你没有实际调用这个方法。方法引用就是Lambda表达式(Applea)>a.getWeight()的快捷写法。

  > Lambda表达式签名：参数列表，返回值



## chapter 4



## chapter 6

### 分组

#### 一级分组

```java
Map<Dish.Type,List<Dish>>dishesByType=menu.stream().collect(groupingBy(Dish::getType));
```

groupingBy接收一个Function，我们把这个Function叫做**分类函数**，因为它用来把流中的元素分成不同的组。

分组操作返回的是一个Map，把**分组函数**返回的值作为映射的键，把流中所有具有这个分类值的项目的列表作为对应的映射值。

#### 多级分组

```java
Map<Dish.Type,Map<CaloricLevel,List<Dish>>>dishesByTypeCaloricLevel=menu.stream().collect(
groupingBy(Dish::getType,    //级分类函数
groupingBy(dish -> {        //二级分类函数
if(dish.getCalories()<=400)return CaloricLevel.DIET;
  elseif(dish.getCalories()<=700)return CaloricLevel.NORMAL;
  else return CaloricLevel.FAT;})));
```



### 分区

```java
Map<Boolean,List<Dish>>partitionedMenu=menu.stream().collect(
  partitioningBy(Dish::isVegetarian));
```

分区是分组的特殊情况：由一个谓词（返回一个布尔值的函数）作为分类函数，它称分区函数。分区函数返回一个布尔值，这意味着得到的分组Map的键类型是Boolean，于是它最多可以分为两组——true是一组，false是一组。



## chapter 8

### lambda重构代码

#### 匿名类到lambda表达式的转换

将匿名类转换为Lambda表达式可能是一个比较复杂的过程。

首先，匿名类和Lambda表达式中的this和super的含义是不同的。**在匿名类中，this代表的是类自身，但是在Lambda中，它代表的是包含类**。

其次，匿名类可以屏蔽包含类的变量，而Lambda表达式不能（它们会导致编译错误），譬如下面这段代码：

```java
int a=10;
Runnable r1=() -> {
  int a=2; 	//编译错误！
  System.out.println(a);
};
Runnable r2=new Runnable(){
  public void run(){
    int a=2; 	//←─一切正常
    System.out.println(a);
  }
};
```

最后，在涉及重载的上下文里，将匿名类转换为Lambda表达式可能导致最终的代码更加晦涩。

Lambda表达式非常适用于需要传递代码片段的场景

**lambda表达式作为参数，传递的是一种行为，与通常理解的参数不一样**



## chapter 9 默认方法

Java8中的**接口**现在支持在声明方法的同时提供实现，这听起来让人惊讶！通过两种方式可以完成这种操作。

其一，Java8允许在接口内声明静态方法

其二，Java8引入了一个新功能，叫默认方法，通过默认方法你可以指定接口方法的默认实现



## chapter 10 用Optional取代null

```java
//如果car是一个null，这段代码会立即抛出一个NullPointerException，而不是等到你试图访问car的属性值时才返回一个错误
Optional<Car> optCar=Optional.of(car);

//以创建一个允许null值的Optional对象
Optional<Car> optCar=Optional.ofNullable(car);

//从对象中提取信息
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name=optInsurance.map(Insurance::getName);

public String getCarInsuranceName(Optional<Person> person){
  return person.flatMap(Person::getCar)
    .flatMap(Car::getInsurance)
    .map(Insurance::getName).orElse("Unknown");
```



## chapter 11 CompletableFuture

```java
public Future<Double> getPriceAsync(String product){
  return CompletableFuture.supplyAsync( () -> calculatePrice(product));
}

```



> **线程池大小的设置方式**
>
> $ Nthreads=NCPU*UCPU*(1+W/C)$
>
> 其中：
>
> - $NCPU$是处理器的核的数目，可以通过Runtime.getRuntime().availableProcessors()得到
> - $UCPU$是期望的CPU利用率（该值应该介于0和1之间）
> - $W/C$是等待时间与计算时间的比率



### 并行——使用流还是CompletableFutures？

* 如果你进行的是计算密集型的操作，并且没有I/O，那么推荐使用Stream接口，因为实现简单，同时效率也可能是最高的（如果所有的线程都是计算密集型的，那就没有必要创建比处理器核数更多的线程）。

* 反之，如果你并行的工作单元还涉及等待I/O的操作（包括网络连接等待），那么使用CompletableFuture灵活性更好，你可以像前文讨论的那样，依据等待/计算，或者W/C的比率设定需要使用的线程数。这种情况不使用并行流的另一个原因是，处理流的流水线中如果发生I/O等待，流的延迟特性会让我们很难判断到底什么时候触发了等待。

### CompletableFuture的thenCompose和thenCombine

如果两个CompletableFuture是有前后顺序关系，即第一个计算出来的结果作为第二个的参数，那么使用thenCompose或者thenComposeAsync。

如果是可以并行执行的两个任务，则使用thenCombine或者thenCombineAsync



## chapter 12 新的时间和日志API

LocalDate

```java
LocalDate date=LocalDate.of(2014,3,18);	//20140318
int year=date.getYear();
Month month=date.getMonth();
int day=date.getDayOfMonth();
DayOfWeek dow=date.getDayOfWeek();
int len=date.lengthOfMonth();
boolean leap=date.isLeapYear();

LocalDate today=LocalDate.now();
```

