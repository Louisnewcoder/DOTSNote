在Tank射炮弹的官方实例中，最后一个步骤里使用System对主摄像机的transform进行赋值，从而实现了摄像机跟随tank的效果。

```C#
   [BurstCompile]
   [UpdateInGroup(typeof(LateSimulationSystemGroup))]
   public partial struct CameraFollowSystem : ISystem
   {
       Entity target;
       Unity.Mathematics.Random rd;
       public void OnCreate(ref SystemState state)
       {
           rd = new Unity.Mathematics.Random(123);
       }

       public void OnUpdate(ref SystemState state)
       {
               // 如果当前还没有看向的目标或者按下了空格就随机找一个目标
           if (target == Entity.Null || Input.GetKeyDown(KeyCode.Space))
           {
               EntityQuery tankQuery = SystemAPI.QueryBuilder().WithAll<TankComponent>().Build();
               var tanks = tankQuery.ToEntityArray(Allocator.Temp);

               if (tanks.Length==0)
               {
                   return; 
               }
               
               target = tanks[rd.NextInt(tanks.Length)];
           }

           var camera = Camera.main.transform; // 获取主摄像机的transform组件，这是MonoBehavior的Component
           var tankTransform = SystemAPI.GetComponent<LocalToWorld>(target); // 获取 随机到的 tank的位置
           // 将tank的位置赋值给 Camera transform的position，这里的float3居然可以直接赋值给 Vector3
           camera.position = tankTransform.Position;
           
           // 基于tank Entity的前进方向对 Camera的位置做修改，放到tank的后面
           // 这里因为Vector3和float3 没有运算符重载，所以要将float3强转乘Vector3
           camera.position -= 10.0f * (Vector3)tankTransform.Forward;
           camera.position += new Vector3(0, 5f, 0);  // 修改高度
           camera.LookAt(tankTransform.Position); // 让Camera一直看向tank的位置
       }
   }
```

在这段代码中我直接在System代码块里使用了MonoBehaviour下的Camera.main.transform API,并使用ECS下Entity的LocalToWorld数据对其进行赋值修改。从而实现了MonoBehaviour 体系下的数据与ECS体系下的数据的互动。
