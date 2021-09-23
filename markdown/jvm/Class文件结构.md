# Class文件结构
class 本质是一张表。 

其中主要包含了一下信息： 

* class文件的魔数
* 支持的最小和最大虚拟机版本
* 常量池
* 类访问标志
* 父类和this类信息、接口信息
* 字段表集合
* 方法表集合
* 属性表集合

## 常量池
常量池包含一下三类内容： 

* 字面常量
* 类和接口的全限定名
* 字段的名称和描述符
* 方法的名称和描述符

## 类访问标志 

限定类的access_flags：

* 枚举--ACC_ENUM
* 接口--ACC_INTERFACE
* 注解--ACC_ANNOTATION
* 抽象类--ACC_ABSTRACT
* 机器自动生成类--ACC_SYNTHETIC
* public类型--ACC_PUBLIC
* final类型--ACC_FINAL
* 是否允许使用invokespecial标识--ACC_SUPER

## 字段表集合
 
 字段表的每一个字段包含一下四类信息：
  
  * 字段的access_flags--双字节控制
  * 字段的名称  --引用常量池中的表项
  * 字段的描述符 --引用常量池中的表项
  * 属性表 
    * ConstantValue -- final 字面常量
    * Signature -- 支持泛型运行时反射获取具体类型
  
 描述字段的access_flags与描述类的access_flags不一样
 
## 方法表集合

方法表的每个方法包含的信息：

  * 字段的access_flags--双字节控制
  * 字段的名称  --引用常量池中的表项
  * 字段的描述符 --引用常量池中的表项
  * 属性表 
     *  Code
        *  max_stack -- 定义操作数栈深度最大值
        *  max_locals -- 代表了局部标量所需的存储空间
        *  code -- 字节码指令的字节流
        *  exception_table -- 显示异常处理表（try catch异常处理表）
        *  LineNumberTable -- Java源码行号与字节码行号
        *  LocalVariableTable -- 描述栈帧中局部变量表中的变量与Java源码中定义的变量之间的关系
        *  StackMapTable -- 字节码校验
        *   ...
     *  Exceptions 方法抛出的异常
