# IEnableableComponent的特点
IEnableableComponent的作用是在Runtime时，启用或禁用继承它的IComponentData类型或者IBufferElement类型的ECS组件。 因为它并不会删除或添加继承它的IComponentData或者IBufferElement类型，所以不会造成Structural Change。

创建示例：
这里创建了一个名为 MyComponent的Component数据组件，它继承了IComponentData是一个最基础的数据组件，然后又继承了IEnableableComponent使得MyComponent可以被启用或者禁用
```C#
public struct MyComponent:IComponentData,IEnableableComponet
{ 
    // fake code 
}
```
如果将IEnableableComponent设置为false。那么MyComponent就被disable了。

被禁用的Component会被Entity视为不存在。用上面的MyComponent来举例，就是当有Query查询的时候，如果Entity身上的MyComponent被disable，那么这个Entity就不会match query，就不会找到它。除非它被启用。 但是对于EntityManager来说这个Entity始终带有MyComponent，不会产生structural change。

==这里也反映出了它产生影响的关键原理，通过它对组件的启用与禁用，来影响Query查询的结果（匹配或不匹配），从而实现部分（或全部）匹配的Entity执行query后的遍历代码内容，以及过滤掉部分（或全部）不匹配的Entity，从而不会执行query后的遍历代码内容==

另外，IEnableableComponent 禁用或者启用另一个Component，并不会改变另一个Component里面的值。



应用示例：

```c#
    [WithAll(typeof(TurretComponent))] // 指定这个IJobEntity引用在所有包含TurretComponet的 Entity上
    // 忽略被查询的Component的状态，否则ShootingComponent被禁用的Entity就不匹配了。我们还需要时刻检查它们是否进入或移动出了safezone
    [WithOptions(EntityQueryOptions.IgnoreComponentEnabledState)] 
    [BurstCompile]
    public partial struct SafeZoneJob : IJobEntity
    {
        public float SquareRadius;
        // 隐式生成一个匹配LocalToWorld组件以及ShootingComponent组件的query，看哪些Entity匹配这个要求
        void Execute(in LocalToWorld transform, EnabledRefRW<ShootingComponent> shootingState)
        {
            // 移动出了范围就开火（大于范围），进入范围就停火。这里的开火与停火就是开启或关闭TurretComponent
            // 这里的开火就是Entity身上的ShootingComponent 为true 相反为false
            // 因为在TurretFiringSystem的Query中，查询的是必须要带ShootingComponent的且满足TurretComponent的Entity
            // 当它为false时，就被Entity视为不存在所以不满足，所以不会执行FiringSystem中的 炮弹实例化代码
            // 当他为 true时，就被视为存在，所以满足，所以就会执行炮弹实例化的代码
            
            shootingState.ValueRW = math.lengthsq(transform.Position) > SquareRadius;
        }
    }
```


