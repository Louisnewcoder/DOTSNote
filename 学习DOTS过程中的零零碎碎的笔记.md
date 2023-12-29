# 数据类型
## RefRO<T> 与 RefRW<T>
它们是结构体，存储的是 T类型的IComponentData的引用。它们继承自 `IQueryTypeParameter`。
 T必须是 struct且必须是IComponnetData。

RefRO -- ReadOnlyRef， 存储一个 Component data的安全的只读引用

RefRW -- Read-WriteRef, 存储一个 Component Data 的可读写的引用
用示例代码理解一下：
```c#
   foreach (var transform in SystemAPI.Query<RefRW<LocalTransform>>().WithAll<TurretComponent>())
   {
        transform.ValueRW.Rotation = math.mul(r, transform.ValueRO.Rotation);
   }
```
这里的transform 就是RefRW<T>类型，它里面存储着 T为LocalTransform 的IComponentData的引用。
找到符合条件的Entity后，对它的Rotation值进行了写入操作。

它们经常作为泛型参数出现在Query<T>方法的调用场景。


# 方法相关
## SystemAPI相关
它的成员方法都几乎是在System内部使用的。
### `Query<T>()`
`Query<T>()`是查询方法，用于查询 T 可为 IAspect, RefRW, RefRO类型。它的返回值是一个可以用来遍历的QueryEnumerable<T>类型的集合。集合里就是在查询语句调用时指定的T类型的数据。

### `QueryBuilder`
用来创建一个类似`EntityQuery`的类型为SystemAPIQueryBuilder 的结构体。使用这个结构体可以创建Query。 调用`QueryBuilder()`几乎不会产生系统开销。

### `SystemAPIQueryBuilder`
这个struct 是 QueryBuilder的返回值类型。它内部也提供了更进一步的定制Query的方法。例如 `WithAll<T>()`等。因为是更进一步的定制Query的方法，所以返回值依旧是SystemAPIQueryBuilder。
==所有不带RW的方法都是返回ReadOnly类型的 SystemAPIQueryBuilder==

==这个方法内的 `.WithXXX()`系列是可以一直链下去的，API内部称之为Chain==


另外还有3个非With系列的方法：

`.WithOptions(EntityQueryOptions options)`
添加一个自己指定的EntityQueryOptions。但是针对每个 query 描述只能调用一次，否则后面的调用会重载前面的调用。如果向组合使用，API建议使用 bitwise（位 与 或 移等操作）或者 ‘|’操作符进行组合

`.AddAdditionalQuery()`
这个方法用来确认终结当前的 query描述。然后将为跟在它后面继续出现的 `WithXXX()`方法将创建一个新的Query

上面这两个都返回 SystemAPIQueryBuilder类型。

`Build()`
获取或者创建 一个符合query描述的 EntityQuery。如果它所在的系统cache中有存放同样的query，那么这个cache中的query就会被赋予这个Build()出来的EntityQuery，否则就按描述在所在的System的cache中创建一个。 

如何使用与理解：
```C#
  if (target == Entity.Null || Input.GetKeyDown(KeyCode.Space))
  {
      // 这行代码右边是query描述并且生成了一个EntityQuery，赋值给了左边的变量
      // 这行代码走完的时候，就已经有匹配或不匹配的结果（就是匹配 query描述的entities存在左边的变量里了
      // 接下来就是如何操作这些数据
      EntityQuery tankQuery = SystemAPI.QueryBuilder().WithAll<TankComponent>().Build();
      // 这里使用ToEntityArray创建了一个NativeArray然后把EntityQuery的结果存进了NativeArray<Entity>集合
      var tanks = tankQuery.ToEntityArray(Allocator.Temp);

      if (tanks.Length==0)
      {
          return; 
      }
      // 然后对集合里的Entity进行访问
      target = tanks[rd.NextInt(tanks.Length)];
  }
```

# ECB相关
