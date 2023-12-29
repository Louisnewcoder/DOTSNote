��Tank���ڵ��Ĺٷ�ʵ���У����һ��������ʹ��System�����������transform���и�ֵ���Ӷ�ʵ�������������tank��Ч����

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
               // �����ǰ��û�п����Ŀ����߰����˿ո�������һ��Ŀ��
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

           var camera = Camera.main.transform; // ��ȡ���������transform���������MonoBehavior��Component
           var tankTransform = SystemAPI.GetComponent<LocalToWorld>(target); // ��ȡ ������� tank��λ��
           // ��tank��λ�ø�ֵ�� Camera transform��position�������float3��Ȼ����ֱ�Ӹ�ֵ�� Vector3
           camera.position = tankTransform.Position;
           
           // ����tank Entity��ǰ������� Camera��λ�����޸ģ��ŵ�tank�ĺ���
           // ������ΪVector3��float3 û����������أ�����Ҫ��float3ǿת��Vector3
           camera.position -= 10.0f * (Vector3)tankTransform.Forward;
           camera.position += new Vector3(0, 5f, 0);  // �޸ĸ߶�
           camera.LookAt(tankTransform.Position); // ��Cameraһֱ����tank��λ��
       }
   }
```

����δ�������ֱ����System�������ʹ����MonoBehaviour�µ�Camera.main.transform API,��ʹ��ECS��Entity��LocalToWorld���ݶ�����и�ֵ�޸ġ��Ӷ�ʵ����MonoBehaviour ��ϵ�µ�������ECS��ϵ�µ����ݵĻ�����
