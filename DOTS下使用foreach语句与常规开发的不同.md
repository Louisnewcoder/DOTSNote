# 一个用于移动物体的例子

```C#
    [BurstCompile]
    public partial struct TankMoveSystem:ISystem
    {
        [BurstCompile]
        public void OnUpdate(ref SystemState state)
        {
            // 一般情况下foreach内迭代的变量是只读的，不能做写入操作
            // 但是，在使用SystemAPI的Query查询时，如果将目标Component标记为 RefRW 可读写
            // 就可以实现对迭代变量的写入操作
            foreach (var transform in SystemAPI.Query<RefRW<LocalTransform>>().WithAll<TankComponent>())
            {
                Debug.Log(transform.GetType());
                float3 pos = transform.ValueRO.Position;

                float angle = (0.5f + noise.cnoise(pos / 10f)) * 4.0f * math.PI;
                Debug.Log(angle.GetType());

                var dir = float3.zero;
                math.sincos(angle, out dir.x, out dir.z);

                // 因为在查询语句LocalTransform标记为 RefRW，这里可以对transform的position的值进行写入
                // 否则如果直接使用 transform.Position就会收到"目标是迭代变量"的报错
                transform.ValueRW.Position += dir * SystemAPI.Time.DeltaTime * 5.0f;
                transform.ValueRW.Rotation = quaternion.RotateY(angle);
            }
        }
    }
```

在上面代码展示了在DOTS架构下，如何在一个System里通过query查找可控制移动（存储移动相关数据）的组件`LocalTransform`。并且利用SystemAPI.Query方法的特点，将目标物标记为RefRW实现对迭代目标的“更新”。最终修改了目标Entity的position与rotation的值，实现了目标的移动。