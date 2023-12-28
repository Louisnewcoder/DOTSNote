# 关于SystemState的Enabled字段
 SystemState的Enabled字段控制当前系统的更新状态。如果设置为false那么 该系统的更新状态则停止，即从下一帧开始，不在执行OnUpdate()的代码，直到Enabled = true。 但是当前帧的OnUpdate中的代码将继续执行完毕。基于这样的机制我们就可以实现一个 ISystem的更新仅执行一次。

因为这个案例用来一次性生成Tank实例的，所以这里在这一帧跑完之后，Tank就不在生成了。

示例代码如下：
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
        
        // 创建一个ECB用于退出 structural change
        EntityCommandBuffer ecb = new EntityCommandBuffer(Allocator.Temp);

        // 创建一个 NativeArray类型 且用于装 “Entity”类型的容器，这里的Entity就是tank
        NativeArray<Entity> tanks = new NativeArray<Entity>(config.count, Allocator.Temp);
 
        // ECB 记录 Instantiate方法的另一个重载。
        // 第一个参数是entity的原型，第二个是用于装实例化出来的若干entities的容器
        // 要注意的是在ECB playback 这条instantiate命令之前，原型不能销毁
        ecb.Instantiate(config.tankEntity, tanks);

        // 播放ecb记录的命令，目前这里只有  ecb.Instantiate(config.tankEntity, tanks) 这一条
        ecb.Playback(state.EntityManager);

       // state.Enabled = false; 在最后设置还是前面设置都是一样的
    }
}
```

