## Java序列化

#### Serializable

​        在分布式场景下,当进行远程通信时,无论是何种类型的数据,都会以二进制序列的形式在网络上传送.序列化是一种将对象转换成字节序列的过程,用于解决在对对象流进行读写操作时所引发的问题.

​        Java中所有要实现序列化的类都必须实现**java.lang.Serializable**接口,它里面没有任何方法.

序列化有如下特点:

- 如果一个类可以被序列化,那么它的子类也能够被序列化
- 由于**static**代表类的成员,**transient**代表对象的临时数据,因此被声明为这两类的数据成员是不能被序列化的
- 子类实现了序列化接口,父类没有,父类中的属性不能序列化,但子类中的属性仍可以正确序列化

#### Extend

​        Externalizable接口extends Serializable接口，支持自定义序列化与反序列化操作,而且在其基础上增加了两个方法：writeExternal()和readExternal()。这两个方法会在序列化和反序列化还原的过程中被自动调用，以便执行一些特殊的操作。

#### FAQ

- Java中可否将A类的序列化结果反序列化为B类(A\B类部分属性值相同)?

- 序列化id-serialVersionUID的作用?

  > Java的序列化机制是通过在运行时判断类的serialVersionUID来验证版本一致性的
  >
  > 在进行反序列化时，JVM会把传来的字节流中的serialVersionUID与本地相应实体（类）的serialVersionUID进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常。 
  >
  > 类的serialVersionUID的**默认值完全依赖于Java编译器的实现**，对于同一个类，用不同的Java编译器编译，有可能会导致不同的serialVersionUID。

- 所有可序列化的类的序列化id都一样会有什么问题?