# 数据类型
## Job相关的数据类型
### TransformAccess和TransformAccessArray
这两个是Unity允许开发者在Job线程中更高效处理Transform相关数据的组件数据类型。简单点理解就是将GameObject的Transform转成TransformAccess就可以更高效的在Job中进行访问。

示例代码 - 声明与存储：
```C#        
       private TransformAccessArray transformAccessArray;

       void Start()
       {
           transformAccessArray = new TransformAccessArray(4 * xHalfCount * zHalfCount);
           for (var z = -zHalfCount; z < zHalfCount; z++)
           {
               for (var x = -xHalfCount; x < xHalfCount; x++)
               {
                   var cube = Instantiate(cubeAchetype);
                   cube.transform.position = new Vector3(x * 1.1f, 0, z * 1.1f);
                   // 直接把普通的transform 存入了TransformAccessArray类型的集合中
                   transformAccessArray.Add(cube.transform);
               }
           }
       }
```
上面的代码中，声明了一个TransformAccessArray的集合，并且直接将常规的Transform存了进去。

而在下面的代码中，被存进去的Transfrom直接作为TransformAccess类型在Job中被进行了访问和数据更新。数据的传递，是由声明一个用来处理这组Transform的Job，并将TransformAccessArray作为参数传给了它的Schedule()方法。

示例代码 - 访问与使用：
```c#
    [BurstCompile]
    struct WaveCubesJob : IJobParallelForTransform
    {
        [ReadOnly] public float elapsedTime;

        public void Execute(int index, TransformAccess transform)
        {
            var distance = Vector3.Distance(transform.position, Vector3.zero);
            transform.localPosition += Vector3.up * math.sin(elapsedTime * 3f + distance * 0.2f);
        }
    }

    void Update()
    {
        using (profilerMarker.Auto(transformAccessArray.length))
        {
            var waveCubesJob = new WaveCubesJob
            {
                elapsedTime = Time.time,
            };
            var waveCubesJobhandle = waveCubesJob.Schedule(transformAccessArray);
            waveCubesJobhandle.Complete();
        }
    }
```

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

## Unity.Transforms命名空间下
### LocalToWorld
namespace Unity.Transforms
阅读官方API文档后的片面理解：

LocalToWorld是一个被动更新的Component数据反应entity在world space中的状态。是基于LocalTransform。它原本是为了rendering system服务的。它只有在TransformSystemGroup运行时才会更新，实际的值（相对于游戏逻辑的SimulationSystemGroup）有可能是过时的（落后的）。Unity也提供了获得精确的LocalTOWorld值的方法。

从之前的使用经验来看，因为它只提供了只读的属性，连对它操作的方法都没有。它应该更多的应用在不精确的场景下，用于做为entity 世界位置（非绝对精确）的输入。
对于没有Parent的entity，直接用LocalTransform就可以（从LocalToWorldSyste API中得到的信息）

ps：也有专门用于计算LocalToWorld的System，`LocalToWorldSystem`

### LocalTransform
性质就相当于MonoBehviour体系下的transform。==它的所有方法都是返回一个==某个类型==的 单独的值，并不更新当前的LocalTransfrom实例==。

方法理解起来都比较简单就偷懒不记了，需要就查查：
https://docs.unity3d.com/Packages/com.unity.entities@1.0/api/Unity.Transforms.LocalTransform.html

### Parent
Parent组件代表的是 ==“我有Parent”== **而不是**“我是Parent”。

很容易混淆，所以哪个entity挂了这个组件，他就有可能是另外一个parent的Child，组件的Value值指向的是它的 父entity。当然它也可以是别的Child的entity。

与Parent呼应的Child Component是ParentSystem自动添加的。Unity不建议手动添加或删除Child buffer。但是可以读。

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

# Unity.Mathmatics相关
## math相关
### .Conjugate(Quaternion rotation)
这个方法返回一个参数 rotation的共轭，就是相反的旋转。可以实现一个旋转的镜像效果

### .Mul()
这时一个乘法函数，能乘很多东西......：
数字与数字相乘，返回数字
元组与元组相乘，返回元组， ==这里的元组是指 X维 向量的感觉==
矩阵与矩阵相乘 返回矩阵
==Quaternion * Quaternion== 返回被 参数1改变的参数2的结果，例如参数1是每秒改变的角度，参数2是当前transform的rotation
Quaternion * 元组（向量） 返回被Quaternion改变的向量
详见：
https://docs.unity3d.com/Packages/com.unity.mathematics@1.2/api/Unity.Mathematics.math.mul.html#Unity_Mathematics_math_mul_Unity_Mathematics_RigidTransform_Unity_Mathematics_float4_

### .Sincos(input, Out sinValueOftheInput,Out cosValueOftheInput)
返回第一个参数的 sine和cosine值。
参数可以是float也可以是矩阵。

==**如果是BurstCompile用这个方法比单独用sin或者cos还快**==

### .select(a, b,bool c)
如果c是true返回b，false返回a


# Job 概念相关
这里的Job概念是指自己对 **Unity.Jobs包，UnityEngine.Jobs以及Unity.Entities包** 里面Job类型的理解。

## 通用信息
它们都是实现多线程的基础数据类型；
它们都需要通过实现Execute方法来指定要执行的工作逻辑；
它们都可以被Schedule，有Handle，要被Complete，有Dependency；
它们都是由Job系统安排调度工作线程执行的。
Run方法都是立刻执行的，所以一般没有Dependency参数

## 关于JobHandle
JobHandle就是Job被Schedule了之后的返回值类型，用于管理被Schedule了的这个Job。

它的实例方法Complete()，在被调用的时候可以确保这个Job已经完成了。但是它属于阻塞方法，所以如果主线程上，没有其他代码需要这个Job内部的数据的话，应该尽量晚的调用这个方法。在Schedule和Complete之间，只要不涉及使用该job正在处理的内容，该写什么代码就写什么代码。
否者会因为被阻塞而导致数据处理流程缓慢。这样不值。

它的静态方法CombineDependencies可以将若干JobHandle打包成1个JobHandle，实现多条件（前置Job）约束；CompleteAll方法可以确保所有Job都完成了。

## 具体的Job类型
### Unity.Jobs包的Job类型
**IJob** 
用于处理 单个Job。而且它的Execute方法没有用于体现迭代的集合或者索引。
Schedule这个类型的Job后，它的Execute方法会执行在一个工作线上。

**IJobFor**
用于处理集合中的每一个元素或者一定次数的迭代操作。它的Execute方法有一个 int i参数作为被迭代的索引。这个参数是由系统安排的。实现了Execute方法后开发者不需要手动指定i。

如果直接调用它的Run(int IterationCount)方法这类Job会立即执行，可能在主线程也可能在工作线程。 


如果是调用Schedule(Int IterationCount, jobDependency)方法则会安排在工作线程上执行。

如果是调用ScheduleParallel(Int InterationCount, batchSize, jobDependency)方法，怎会安排在若干工作线程上执行。

这里的IterationCount参数，是指一共迭代多少次，如果是处理结合的每一个元素，一般都是输入集合的length。 jobDependency是指要在哪个Job完成后在执行这个Job。

**IJobParallelFor**
这个就是专门用来将处理集合或者若干次迭代的逻辑安排到分线程去执行的。
它没有Run()方法。直接Schedule()

**关于IJobExtensions, IJobForExtensions, IJobParallelForExtensions**
它们3个就是上面3个Job接口类型的静态类。使用方式是静态类调用静态的
Run(T JobType)；
或
Schedule(T JobType, int InterationCount, BatchSize, jobDependency)
语法与普通实例方法类似，但是要把Job的实例作为第一个参数传进去。
主要还是服务于代码风格。

### UnityEngine.Jobs包的Job类型
**IJobParallelForTransform**
它是专门用于处理Transform组件的Job类型，也是实现Execute方法即可。
Schedule这个类型的时候传入的参数是 Tranform的集合，但是这个集合是TransformAccessArray类型，TransformAccess类型作为元素。

TransformAccess和TransformAccessArray是Unity提供的用于MonoBehevior和Job系统沟通的数据类型。直接把Transform存入TAA集合里就自动被Job视为TransformAccess了。

### Unity.Entities包的Job类型
**IJobChunk**
用于处理整个Chunk层面的Job

**IJobEntity与IJobEntityExtension**
用于处理单个entity层面的Job。底层封装的还是IJobChunk。

