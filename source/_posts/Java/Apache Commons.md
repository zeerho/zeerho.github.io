---
title: Apache Commons
date: 2017-10-13 10:00:00
tags: [Java]
---

# Apache Commons

<!-- more -->

## Collections

### Transformer

**StringValueTransformer**
返回 `String.valueOf(input)`。

**ClosureTransformer**
返回 `closure.execute(input)` 之后的 input。

**ChainedTransformer**
持有一个 Transformer[] 数组，对 input 依次执行 `transform()` 方法。

**PredicateTransformer**
返回 `predicate.evaluate(input) ? Boolean.TRUE : Boolean.FALSE`。

**SwitchTransformer**
持有一个 Predicate[] 数组、一个 Transformer[] 数组、一个缺省 Transformer。
若 Predicate[i] 断言为真，则返回 `Transformer[i].transform(input)`。若断言全为假，则返回缺省 Transformer 的转换结果。

**InvokerTransformer**
持有一个方法名 methodName、该方法对应的参数类型数组 paramTypes[]、待用的方法参数数组 args[]。
通过 methodName + paramTypes[] 来确定 input 对象中的一个方法，然后传入 args[] 来调用。

**ExceptionTransformer**
`throw new FunctorException("ExceptionTransformer invoked");`

**InstantiateTransformer**
input 对象只能为 Class，调用其构造方法来实例化对象。

**FactoryTransformer**
持有一个 org.apache.commons.collections.Factory，返回 `factory.create()`。

**CloneTransformer**
`return PrototypeFactory.getInstance(input).create();`

**MapTransformer**
持有一个 Map，返回 `map.get(input)`。

**BeanToPropertyValueTransformer**
持有一个属性名 propertyName，返回 input 的指定属性。

**NOPTransformer**
返回 input。

**ConstantTransformer**
返回一个固定的对象。

---
### Predicate

**AllPredicate**
持有一个 Predicate[] 数组，对输入对象应用所有断言，任一假即为假。

**AndPredicate**
持有两个 Predicate，取两个断言的交集。

**AnyPredicate**
持有一个 Predicate[] 数组，对输入对象应用所有断言，任一真即为真。

**BeanPredicate**
持有一个 Predicate 和 Bean 的一个属性名，取出输入对象的指定属性并应用断言。

**BeanPropertyValueEqualsPredicate**
持有一个属性值和 Bean 的一个属性名，比较属性值和输入对象的指定属性是否相等。

**EqualPredicate**
持有一个对象，比较输入对象与之是否相等。

**ExceptionPredicate**
固定抛出一个异常 FunctorException。

**FalsePredicate**
固定返回假。

**IdentityPredicate**
持有一个对象引用，比较输入引用与之是否指向同一个对象。

**InstanceOfPredicate**
持有一个 Class，判断输入对象是否为该类型的实例。

**NonePredicate**
持有一个 Predicate[] 数组，对输入对象应用所有断言，任一为真即为假。

**NotNullPredicate**
判断输入对象是否非 null。

**NotPredicate**
持有一个 Predicate，对输入对象应用断言，为假即为真。

**NullIsExceptionPredicate**
持有一个 Predicate，若输入对象为空则抛出异常 IllegalArgumentException，否则应用该断言。

**NullIsFalsePredicate**
持有一个 Predicate，若输入对象为空则为假，否则应用该断言。

**NullIsTruePredicate**
持有一个 Predicate，若输入对象为空则为真，否则应用该断言。

**NullPredicate**
判断输入对象是否为 null。

**OnePredicate**
持有一个 Predicate[] 数组，若其中有且仅有一个断言为真即为真。

**OrPredicate**
持有两个 Predicate，取两个断言的并集。

**TransformedPredicate**
持有一个 Transformer 和一个 Predicate，先对输入对象应用 Transformer，再应用断言。

**TransformerPredicate**
持有一个 Transformer，该 Transformer 的转化结果必须为 Boolean 对象，返回该转换结果。

**TruePredicate**
固定返回真。

**UniquePredicate**
持有一个 HashSet，判断输入对象是否唯一。

---
