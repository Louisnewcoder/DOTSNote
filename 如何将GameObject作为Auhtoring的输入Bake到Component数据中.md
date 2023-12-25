在设计DOTS数据时，有时候开发者可能会希望像在MonoBehaviour环境下实例化GameObject一样在ECS架构中不断生成Entities。要想实现这个效果，可以将GameObject或Transform等可转化的组件声明在Authoring脚本内，作为Baker的输入。然后在Baking过程中输入给IComponentData。

示例如下：
```C#
// Baker定义在了Authoring脚本内部,没有什么影响，只是用来尝试不同的写法；
// 定义在外部也是一样的
public class TurrentAuthoring : MonoBehaviour
{
    public GameObject prefab;
    public Transform spawnPoint;

    public class TurrentBaker:Baker<TurrentAuthoring>
    {
        public override void Bake(TurrentAuthoring authoring)
        {
            // 这里通过Dynamic值告诉Unity这个Entity会被移动，请给我LocalTransform组件
            // 这里的GetEntity是 IBaker 的方法
            Entity e = GetEntity(TransformUsageFlags.Dynamic);

            //这里的AddComponent是 IBaker 的方法。其他的API也有同名方法，但是归属不同
            //https://docs.unity3d.com/Packages/com.unity.entities@1.0/api/Unity.Entities.IBaker.AddComponent.html
            //为entity 添加一个 Component
            AddComponent(e, new TurretComponent 
            { 
                // 因为我们在TurretComponent中定义它存储的数据是Entity类型，所以这里要将Authoring的输入转化为Entity
                CannonBallprefab = GetEntity(authoring.prefab,TransformUsageFlags.Dynamic),
                CannonBallSpawnPoint= GetEntity(authoring.spawnPoint,TransformUsageFlags.Dynamic) 
            } );
        }
    }
}
```
再看上面代码对应的IComponentData内部：
```C#
/// <summary>
/// 为了让Turret能够发射炮弹，将Turret由一个纯tag型Component变成了一个具有数据的Component
/// 其中存储了2个数据一个是作为炮弹的Prefab，另一个是“炮口的位置”
/// </summary>
public struct TurretComponent : IComponentData
{
    // 这里要注意的是,在DOTS下，实例化出来的是Entity而不是GameObject
    // 所以这里的类型是Entity，在Baker中Baking的时候，也要将Authoring的输入转化为Entity
    public Entity CannonBallprefab;
    public Entity CannonBallSpawnPoint;
}
```

