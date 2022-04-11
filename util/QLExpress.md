# QLExpress

## 目录

### 简介

>  QLExpress从一开始就是从复杂的阿里电商业务系统出发，并且不断完善的脚本语言解析引擎框架，在不追求java语法的完整性的前提下（比如异常处理，foreach循环，lambda表达式，这些都是groovy是强项），定制了很多普遍存在的业务需求解决方案（比如变量解析，spring打通，函数封装，操作符定制，宏替换），同时在高性能、高并发、线程安全等方面也下足了功夫，久经考验。

### 支持特性

* 线程安全

  > 引擎运算过程中的产生的临时变量都是threadlocal类型。

* 高效执行

  > 比较耗时的脚本编译过程可以缓存在本地机器，运行时的临时变量创建采用了缓冲池的技术，和groovy性能相当。

* 弱类型脚本语言

  > 和groovy，javascript语法类似，虽然比强类型脚本语言要慢一些，但是使业务的灵活度大大增强。

* 安全控制

  > 可以通过设置相关运行参数，预防死循环、高危系统api调用等情况。

* 代码精简

  > 依赖最小，250k的jar包适合所有java的运行环境，在android系统的低端pos机也得到广泛运用

### Maven 引入

```xml
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>QLExpress</artifactId>
  <version>3.2.2</version>
</dependency>
```



### 例子 Github 源码

[QLExpress Github 地址](https://github.com/alibaba/QLExpress)

### 图分解

### 提示

### 例子

#### 简单使用

```java
package com.ql.util.express.myblog;


import com.ql.util.express.DefaultContext;
import com.ql.util.express.ExpressRunner;
import org.junit.Test;

/**
 * @author White
 * @date 2021/3/3
 */
public class TestQLExpress {

    @Test
    public void testQLExpression() {
        //语法分析和计算的入口类
        ExpressRunner runner = new ExpressRunner();
        //表达式计算的数据注入接口
        DefaultContext<String, Object> context = new DefaultContext<String, Object>();
        context.put("a",1);
        context.put("b",2);
        context.put("c",3);
        //定义计算表达式
        String express = "a+b*c";
        try {
            //解析规则、执行规则
            Object execute = runner.execute(express, context, null, true, true);
            System.out.println(execute);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```



##### Runner 执行器设置

* aIsPrecise 是否需要高精度计算支持

* aIstrace 是否跟踪执行指令的过程

构造方法

```java
	/**
	 *
	 * @param aIsPrecise 是否需要高精度计算支持
	 * @param aIstrace 是否跟踪执行指令的过程
	 */
	public ExpressRunner(boolean aIsPrecise,boolean aIstrace){
		this(aIsPrecise,aIstrace,new DefaultExpressResourceLoader(),null);
	}
```

##### Runner 执行命令的设置

* expressString 程序文本，即要执行的表达式或规则。

* context 执行上下文。【IExpressContext对象(如果是Spring的Bean，则创建SpringBeanContext对象) 表示执行上下文】
* errorList 输出的错误信息List
* isCache 是否使用Cache中的指令集，多次执行同一语句的情况下用以提高执行效率
* isTrace 是否输出详细的执行指令信息
* log 输出的log

构造方法

```java
/**
 * 执行一段文本
 * @param expressString 程序文本
 * @param context 执行上下文
 * @param errorList 输出的错误信息List
 * @param isCache 是否使用Cache中的指令集
 * @param isTrace 是否输出详细的执行指令信息
 * @return
 * @throws Exception
 */
	public Object execute(String expressString, IExpressContext<String,Object> context,
			List<String> errorList, boolean isCache, boolean isTrace) throws Exception {
		return this.execute(expressString, context, errorList, isCache, isTrace, null);
	}
```

##### 备注

从上述的例子，结合官方文档可以看出，QLExpress 的运行过程可以拆分为以下步骤：

1. 单词分解
2. 单词分析
3. 构建语法树进行语法分析
4. 生成运行期指令集合
5. 执行生成的指令集合

当我们在调用 Runner 的 execute 方法时，将  isCache 设置为 true 即可开启缓存(缓存前4个过程)，能够提升性能，特别是在**同一个表达式的循环**中计算，能够极大的提升计算性能。这里的缓存是指**本地缓存**。

#### 支持普通的 Java 语法执行

##### 运算符支持

| 算术运算符                       | Java运算符                     | 备注                                                         | 示例                                                         |
| -------------------------------- | ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| +,-,*,/,<,>,≤,≥,==,!=,%,++,–     | +,-,*,/,<,>,<=,>=,==,<>,%,++,– | mod`等同于`%                                                 | a*b `等同于` a 乘以 b                                        |
| for,break、continue,if then else |                                | 不支持try{}catch{}<br/>不支持java8的lambda表达式<br/>不支持for循环集合操作<br/>弱类型语言，请不要定义类型声明,更不要用Templete<br/>array的声明不一样<br/>min,max,round,print,println,like,in 都是系统默认函数的关键字，请不要作为变量名<br/> | int n=10; <br/>int sum=0;<br/>for(int i=0;i<n;i++){<br/>sum=sum+i;}<br/> return sum; |

###### 算术运算示例

```java
    /**
     * 简单的数学运算
     */
    @Test
    public void testSimpleMathExpression() {
        //语法分析和计算的入口类
        ExpressRunner runner = new ExpressRunner();
        //表达式计算的数据注入接口
        DefaultContext<String, Object> context = new DefaultContext<String, Object>();
        context.put("a",1);
        context.put("b",2);
        context.put("c",3);
        //定义计算表达式
        String express = "a+b*c";
        try {
            //解析规则、执行规则
            Object execute = runner.execute(express, context, null, true, true);
            System.out.println(execute);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

###### for循环示例

```java
    /**
     * java for 循环运算
     */
    @Test
    public void testJavaFor() {
        StringBuilder sb = new StringBuilder();
        sb.append("n=10;");
        sb.append("\n");
        sb.append("sum=0;");
        sb.append("\n");
        sb.append("for(i=0;i<n;i++){");
        sb.append("\n");
        sb.append("sum=sum+i;");
        sb.append("\n");
        sb.append("}");
        sb.append("\n");
        sb.append("return sum;");
        ExpressRunner runner = new ExpressRunner();
        DefaultContext<String, Object> context = new DefaultContext<String, Object>();
        String express = sb.toString();
        try {
            Object execute = runner.execute(express, context, null, true, true);
            System.out.println(execute);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```



##### 运算符分类

| 分类        | 运算符                                    | 示例      |
| ----------- | ----------------------------------------- | --------- |
| 位运算      | `^`，`~`，`&`，`|`，`<<`，`>>`            | `1<<2`    |
| 四则运算    | `+`，`-`，`*`，`/`,`%`,`++`,`--`          | `3%2`     |
| Boolean运算 | `!`,`<`,`>`,`<=`,`>=`,`==`,`!=`,`&&`,`||` | `2==3`    |
| 其他运算    | `=`,`?:`                                  | `2>3?1:0` |

###### 双目运算示例

```java
    /**
     * java 三目运算
     */
    @Test
    public void testJavaThreeEyes() {
        StringBuilder sb = new StringBuilder();
        sb.append("a=1;");
        sb.append("\n");
        sb.append("b=2;");
        sb.append("\n");
        sb.append("maxNum = a>b?a:b");
        String express = sb.toString();
        ExpressRunner runner = new ExpressRunner();
        DefaultContext<String, Object> context = new DefaultContext<String, Object>();
        try {
            Object execute = runner.execute(express, context, null, true, true);
            System.out.println(execute);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

##### 部分运算符分类

| 运算符      | 描述                       | 运算符   | 描述           |
| ----------- | -------------------------- | -------- | -------------- |
| mod         | 与 %等同                   | for      | 环语句控制符   |
| return      | 进行值返回                 | if       | 条件语句控制符 |
| in          | 类似sql语句的in            | then     | 与if同用       |
| exportAlias | 创建别名，并转换为全局别名 | else     | 条件语句控制符 |
| alias       | 创建别名                   | break    | 退出循环操作符 |
| macro       | 定义宏                     | continue | 继续循环操作符 |
| exportDef   | 将局部变量转换为全局变量   | function | 进行函数定义   |
| like        | 类似sql语句的like          | new      | 创建一个对象   |
| import      | 引入包或类，需在脚本最前头 | class    | 定义类         |
| NewMap      | 创建Map                    | NewList  | 创建集合       |

##### 部分说明

```apl
include:在一个表达式中引入其它表达式。例如： include com.taobao.upp.b; 资源的转载可以自定义接口IExpressResourceLoader来实现，缺省是从文件中装载
[]:匿名创建数组.int[][] abc = [[11,12,13],[21,22,23]];
NewMap:创建HashMap. Map abc = NewMap(1:1,2:2);Map abc = NewMap("a":1,"b":2)
NewList:串接ArrayList.List abc = NewList(1,2,3);
exportDef: 将局部变量转换为全局变量，例如：exportDef long userId
alias:创建别名，例如： alias 用户ID user.userId
macro: 定义宏，例如： macro 降级  {level = level - 1}
in: 操作符号，例如： 3 in (3,4,5)
like:操作符号，例如： "abc" like ‘ab%‘
```

##### **自定义系统函数**

```apl
max:取最大值max(3,4,5)
min:最最小值min(2,9,1)
round:四舍五入round(19.08,1)
print:输出信息不换行print("abc")
println:输出信息并换行 println("abc")
```

