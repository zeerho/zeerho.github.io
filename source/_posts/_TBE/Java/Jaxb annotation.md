# Jaxb annotation

标签： !Java

[TOC]

---

## Jaxb 处理 Java 对象和 xml 之间转换常用的 annotation

- @XmlType
- @XmlElement
- @XmlRootElement
- @XmlAttribute
- @XmlAccessorType
- @XmlAccessorOrder
- @XmlTransient
- @XmlJavaTypeAdapter

## 常用 annotation 使用说明

### @XmlType
`@XmlType` 用在 class 类的注解，常与 `@XmlRootElement`，`@XmlAccessorType` 一起使用。它有三个属性：*name*、*propOrder*、*namespace*。如：
```xml
@XmlType(name = "basicStruct", propOrder = {
    "intValue",
    "stringArray",
    "stringValue"
)
```
在使用 `@XmlType` 的 *propOrder* 属性时，必须列出 JavaBean 对象中的所有属性，否则会报错。

### @XmlElement
`@XmlElement` 将 Java 对象的属性映射为 xml 的节点，在使用 `@XmlElement` 时，可通过 *name* 属性指定 java 对象属性在 xml 中显示的名称，缺省值为 bean 中的属性名。如：
```xml
@XmlElement(name="Address")　　
private String yourAddress;
```

### @XmlRootElement
`@XmlRootElement` 用于类级别的注解，对应 xml 的根元素，常与 `@XmlType` 和 `@XmlAccessorType` 一起使用。如：
```xml
@XmlType
@XmlAccessorType(XmlAccessType.FIELD)
@XmlRootElement
public class Address {}
```

### @XmlAttribute
`@XmlAttribute` 用于把 java 对象的属性映射为 xml 的属性，并可通过 name 属性为生成的 xml 属性指定别名。如：
```xml
@XmlAttribute(name="Country")
private String state;
```

### @XmlAccessorType
`@XmlAccessorType` 用于指定由 java 对象生成 xml 文件时对 java 对象属性的访问方式。常与 `@XmlRootElement`、`@XmlType` 一起使用。它的属性值是 `XmlAccessType` 的4个枚举值，分别为：

- XmlAccessType.FIELD：java 对象中的所有成员变量
- XmlAccessType.PROPERTY：java 对象中所有通过 getter/setter 方式访问的成员变量
- XmlAccessType.PUBLIC_MEMBER：java对象中所有的 public 访问权限的成员变量和通过 getter/setter 方式访问的成员变量
- XmlAccessType.NONE：java 对象的所有属性都不映射为 xml 的元素

注意：@XmlAccessorType 的默认访问级别是 XmlAccessType.PUBLIC_MEMBER，因此，如果 java 对象中的 private 成员变量设置了 public 权限的 getter/setter 方法，就不要在 private变量上使用 `@XmlElement` 和 `@XmlAttribute` 注解，否则在由 java 对象生成 xml 时会报同一个属性在 java 类里存在两次的错误。同理，如果 `@XmlAccessorType` 的访问权限为 XmlAccessType.NONE，如果在 java 的成员变量上使用了 `@XmlElement` 或 `@XmlAttribute` 注解，这些成员变量依然可以映射到 xml 文件。

### @XmlAccessorOrder

` @XmlAccessorOrder` 用于对 java 对象生成的 xml 元素进行排序。它有两个属性值：

- AccessorOrder.ALPHABETICAL：对生成的 xml 元素按字母书序排序
- XmlAccessOrder.UNDEFINED：不排序

### @XmlTransient

`@XmlTransient` 用于标示在由 java 对象映射 xml 时，忽略此属性。即在生成的 xml 文件中不出现此元素。

### @XmlJavaTypeAdapter

`@XmlJavaTypeAdapter` 常用在转换比较复杂的对象时，如 map 类型或者格式化日期等。使用此注解时，需要自己写一个 adapter 类继承 XmlAdapter 抽象类，并实现里面的方法。

`@XmlJavaTypeAdapter(value=xxx.class)`，value 为自己定义的 adapter 类。
　　XmlAdapter如下：
　　
```
public abstract class XmlAdapter<ValueType,BoundType> {
    // Do-nothing constructor for the derived classes.
    protected XmlAdapter() {}
    // Convert a value type to a bound type.
    public abstract BoundType unmarshal(ValueType v);
    // Convert a bound type to a value type.
    public abstract ValueType marshal(BoundType v);
 }
```



