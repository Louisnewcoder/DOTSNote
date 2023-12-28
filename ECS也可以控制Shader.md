# 如何通过ECS改变Entity的颜色

核心就是一个意识 ---- 颜色也是数据，ECS的Rendering包里提供了这个Component 和 它的Authoring脚本。

先要在编辑器里找到自己要修改的GameObject或者Prefab。然后为它们或者它身上的某个子物体添加脚本组件：
`URP Material Property Base Color Authoring`
从名字可以看出来这是个Unity提供的用来Bake材质基色的Authoring脚本。可以打开看看：

```C#
#if URP_10_0_0_OR_NEWER
using Unity.Entities;
using Unity.Mathematics;

namespace Unity.Rendering
{
    [MaterialProperty("_BaseColor")]
    public struct URPMaterialPropertyBaseColor : IComponentData
    {
        public float4 Value;
    }

    [UnityEngine.DisallowMultipleComponent]
    public class URPMaterialPropertyBaseColorAuthoring : UnityEngine.MonoBehaviour
    {
        [Unity.Entities.RegisterBinding(typeof(URPMaterialPropertyBaseColor), nameof(URPMaterialPropertyBaseColor.Value))]
        public UnityEngine.Color color;

        class URPMaterialPropertyBaseColorBaker : Unity.Entities.Baker<URPMaterialPropertyBaseColorAuthoring>
        {
            public override void Bake(URPMaterialPropertyBaseColorAuthoring authoring)
            {
                Unity.Rendering.URPMaterialPropertyBaseColor component = default(Unity.Rendering.URPMaterialPropertyBaseColor);
                float4 colorValues;
                colorValues.x = authoring.color.linear.r;
                colorValues.y = authoring.color.linear.g;
                colorValues.z = authoring.color.linear.b;
                colorValues.w = authoring.color.linear.a;
                component.Value = colorValues;
                var entity = GetEntity(TransformUsageFlags.Renderable);
                AddComponent(entity, component);
            }
        }
    }
}
#endif
```

这里值得了解的是 它处理了一个名为URPMaterialPropertyBaseColor的ECS component。这个Component里面存了一个float4的数据。这个也是GPU处理数据的最基本类型；
另外这个Component被`[MaterialProperty]`标记为它作用的GameObject的`_BaseColor` 的input。

这些都是Entities Graphics Package里的东西.

然后，我们只需要在System中，编写处理_BaseColor数据的逻辑即可。
实际上非常简单，就是想办法给URPMaterialPropertyBaseColor 里面的 Value 变量赋值。
下面关键的代码只有一行：

```C#
        foreach (var tank in tanks)
        {
            SetComponentForLinkedEntityGroup(tank, queryMask, new URPMaterialPropertyBaseColor { Value = RandomColor(ref random)});
        }

        // 生成随机颜色 float4的代码
        static float4 RandomColor(ref Unity.Mathematics.Random random)
    {
        // 0.618034005f is inverse of the golden ratio
        var hue = (random.NextFloat() + 0.618034005f) % 1;
        return (Vector4)Color.HSVToRGB(hue, 1.0f, 1.0f);
    }
```



Tank生成的完整片段：
```C#
[BurstCompile]
public partial struct ConfigSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {

        // SystemState的Enabled字段控制当前系统的更新状态
        // 如果设置为false那么 该系统的更新状态则停止，即从下一帧开始，不在执行OnUpdate()的代码
        // 直到Enabled = true。 但是当前帧的OnUpdate中的代码将继续执行完毕
        // 基于这样的机制我们就可以实现一个 ISystem的更新值执行一次。
        // 因为这个案例用来一次性生成Tank实例的，所以这里在这一帧跑完之后，Tank就不在生成了。

        state.Enabled = false;

        // 因为我们只有1个Empty GameObject 挂载了ConfigAuthoring也就是说只有1个ConfigComponent
        // 所以在这里把它作为单例拿就可以了 
        ConfigComponent config = SystemAPI.GetSingleton<ConfigComponent>();

        // 返回一个Random实例，随机种子取决于传入的参数
        var random = new Unity.Mathematics.Random(432);

        // 创建一个Entity query实例
        EntityQuery query = SystemAPI.QueryBuilder().WithAll<URPMaterialPropertyBaseColor>().Build();
        // 再用这个query生成一个 QueryMask，用来过滤符合条件的Entity
        EntityQueryMask queryMask = query.GetEntityQueryMask();

        // 创建一个ECB用于推迟 structural change
        EntityCommandBuffer ecb = new EntityCommandBuffer(Allocator.Temp);

        // 创建一个 NativeArray类型 且用于装 “Entity”类型的容器，这里的Entity就是tank
        NativeArray<Entity> tanks = new NativeArray<Entity>(config.count, Allocator.Temp);
 
        // ECB 记录 Instantiate方法的另一个重载。
        // 第一个参数是entity的原型，第二个是用于装实例化出来的若干entities的容器
        // 要注意的是在ECB playback 这条instantiate命令之前，原型不能销毁
        ecb.Instantiate(config.tankEntity, tanks);

        // 通过遍历为每一个 tank修改颜色
        foreach (var tank in tanks)
        {
            // 这个方法是用来设置Entity的 在这个方法参数中指定的 Component
            // 它基于参数中指定的 QueryMask来筛选符合的Entity，在这里QueryMask用来查找具有 URPMaterialPropertyBaseColor Component的Entity
            // 方法的定义提到，QueryMask会忽略所有query限制条件，包括EnableableComponent和chunk filtering
            // 这条命令，也被记录到了ECB中，会在playback时按顺序执行到它
            
            ecb.SetComponentForLinkedEntityGroup(tank, queryMask, new URPMaterialPropertyBaseColor { Value = RandomColor(ref random)});
            
        }

        // 播放ecb记录的命令，目前这里只有  ecb.Instantiate(config.tankEntity, tanks) 这一条;
        ecb.Playback(state.EntityManager);

       // state.Enabled = false; 在最后设置还是前面设置都是一样的
    }

    // 修改颜色的自定义方法
    // Helper to create any amount of colors as distinct from each other as possible.
    // See https://martin.ankerl.com/2009/12/09/how-to-create-random-colors-programmatically/
    static float4 RandomColor(ref Unity.Mathematics.Random random)
    {
        // 0.618034005f is inverse of the golden ratio
        var hue = (random.NextFloat() + 0.618034005f) % 1;
        return (Vector4)Color.HSVToRGB(hue, 1.0f, 1.0f);
    }
}
```

同样，如果我们也为炮弹设置一个URPMaterialPropertyBaseColorComponent，并在Authoring脚本中Bake给Entity。那么炮弹的颜色也可以被控制。比如设置成与Tank相同。