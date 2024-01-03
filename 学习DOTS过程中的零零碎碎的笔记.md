# ��������
## Job��ص���������
### TransformAccess��TransformAccessArray
��������Unity����������Job�߳��и���Ч����Transform������ݵ�����������͡��򵥵������ǽ�GameObject��Transformת��TransformAccess�Ϳ��Ը���Ч����Job�н��з��ʡ�

ʾ������ - ������洢��
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
                   // ֱ�Ӱ���ͨ��transform ������TransformAccessArray���͵ļ�����
                   transformAccessArray.Add(cube.transform);
               }
           }
       }
```
����Ĵ����У�������һ��TransformAccessArray�ļ��ϣ�����ֱ�ӽ������Transform���˽�ȥ��

��������Ĵ����У������ȥ��Transfromֱ����ΪTransformAccess������Job�б������˷��ʺ����ݸ��¡����ݵĴ��ݣ���������һ��������������Transform��Job������TransformAccessArray��Ϊ��������������Schedule()������

ʾ������ - ������ʹ�ã�
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

## RefRO<T> �� RefRW<T>
�����ǽṹ�壬�洢���� T���͵�IComponentData�����á����Ǽ̳��� `IQueryTypeParameter`��
 T������ struct�ұ�����IComponnetData��

RefRO -- ReadOnlyRef�� �洢һ�� Component data�İ�ȫ��ֻ������

RefRW -- Read-WriteRef, �洢һ�� Component Data �Ŀɶ�д������
��ʾ���������һ�£�
```c#
   foreach (var transform in SystemAPI.Query<RefRW<LocalTransform>>().WithAll<TurretComponent>())
   {
        transform.ValueRW.Rotation = math.mul(r, transform.ValueRO.Rotation);
   }
```
�����transform ����RefRW<T>���ͣ�������洢�� TΪLocalTransform ��IComponentData�����á�
�ҵ�����������Entity�󣬶�����Rotationֵ������д�������

���Ǿ�����Ϊ���Ͳ���������Query<T>�����ĵ��ó�����

## Unity.Transforms�����ռ���
### LocalToWorld
namespace Unity.Transforms
�Ķ��ٷ�API�ĵ����Ƭ����⣺

LocalToWorld��һ���������µ�Component���ݷ�Ӧentity��world space�е�״̬���ǻ���LocalTransform����ԭ����Ϊ��rendering system����ġ���ֻ����TransformSystemGroup����ʱ�Ż���£�ʵ�ʵ�ֵ���������Ϸ�߼���SimulationSystemGroup���п����ǹ�ʱ�ģ����ģ���UnityҲ�ṩ�˻�þ�ȷ��LocalTOWorldֵ�ķ�����

��֮ǰ��ʹ�þ�����������Ϊ��ֻ�ṩ��ֻ�������ԣ������������ķ�����û�С���Ӧ�ø����Ӧ���ڲ���ȷ�ĳ����£�������Ϊentity ����λ�ã��Ǿ��Ծ�ȷ�������롣
����û��Parent��entity��ֱ����LocalTransform�Ϳ��ԣ���LocalToWorldSyste API�еõ�����Ϣ��

ps��Ҳ��ר�����ڼ���LocalToWorld��System��`LocalToWorldSystem`

### LocalTransform
���ʾ��൱��MonoBehviour��ϵ�µ�transform��==�������з������Ƿ���һ��==ĳ������==�� ������ֵ���������µ�ǰ��LocalTransfromʵ��==��

��������������Ƚϼ򵥾�͵�������ˣ���Ҫ�Ͳ�飺
https://docs.unity3d.com/Packages/com.unity.entities@1.0/api/Unity.Transforms.LocalTransform.html

### Parent
Parent���������� ==������Parent��== **������**������Parent����

�����׻����������ĸ�entity�����������������п���������һ��parent��Child�������Valueֵָ��������� ��entity����Ȼ��Ҳ�����Ǳ��Child��entity��

��Parent��Ӧ��Child Component��ParentSystem�Զ���ӵġ�Unity�������ֶ���ӻ�ɾ��Child buffer�����ǿ��Զ���

# �������
## SystemAPI���
���ĳ�Ա��������������System�ڲ�ʹ�õġ�
### `Query<T>()`
`Query<T>()`�ǲ�ѯ���������ڲ�ѯ T ��Ϊ IAspect, RefRW, RefRO���͡����ķ���ֵ��һ����������������QueryEnumerable<T>���͵ļ��ϡ�����������ڲ�ѯ������ʱָ����T���͵����ݡ�

### `QueryBuilder`
��������һ������`EntityQuery`������ΪSystemAPIQueryBuilder �Ľṹ�塣ʹ������ṹ����Դ���Query�� ����`QueryBuilder()`�����������ϵͳ������

### `SystemAPIQueryBuilder`
���struct �� QueryBuilder�ķ���ֵ���͡����ڲ�Ҳ�ṩ�˸���һ���Ķ���Query�ķ��������� `WithAll<T>()`�ȡ���Ϊ�Ǹ���һ���Ķ���Query�ķ��������Է���ֵ������SystemAPIQueryBuilder��
==���в���RW�ķ������Ƿ���ReadOnly���͵� SystemAPIQueryBuilder==

==��������ڵ� `.WithXXX()`ϵ���ǿ���һֱ����ȥ�ģ�API�ڲ���֮ΪChain==


���⻹��3����Withϵ�еķ�����

`.WithOptions(EntityQueryOptions options)`
���һ���Լ�ָ����EntityQueryOptions���������ÿ�� query ����ֻ�ܵ���һ�Σ��������ĵ��û�����ǰ��ĵ��á���������ʹ�ã�API����ʹ�� bitwise��λ �� �� �ƵȲ��������� ��|���������������

`.AddAdditionalQuery()`
�����������ȷ���սᵱǰ�� query������Ȼ��Ϊ����������������ֵ� `WithXXX()`����������һ���µ�Query

���������������� SystemAPIQueryBuilder���͡�

`Build()`
��ȡ���ߴ��� һ������query������ EntityQuery����������ڵ�ϵͳcache���д��ͬ����query����ô���cache�е�query�ͻᱻ�������Build()������EntityQuery������Ͱ����������ڵ�System��cache�д���һ���� 

���ʹ������⣺
```C#
  if (target == Entity.Null || Input.GetKeyDown(KeyCode.Space))
  {
      // ���д����ұ���query��������������һ��EntityQuery����ֵ������ߵı���
      // ���д��������ʱ�򣬾��Ѿ���ƥ���ƥ��Ľ��������ƥ�� query������entities������ߵı�������
      // ������������β�����Щ����
      EntityQuery tankQuery = SystemAPI.QueryBuilder().WithAll<TankComponent>().Build();
      // ����ʹ��ToEntityArray������һ��NativeArrayȻ���EntityQuery�Ľ�������NativeArray<Entity>����
      var tanks = tankQuery.ToEntityArray(Allocator.Temp);

      if (tanks.Length==0)
      {
          return; 
      }
      // Ȼ��Լ������Entity���з���
      target = tanks[rd.NextInt(tanks.Length)];
  }
```

# ECB���

# Unity.Mathmatics���
## math���
### .Conjugate(Quaternion rotation)
�����������һ������ rotation�Ĺ�������෴����ת������ʵ��һ����ת�ľ���Ч��

### .Mul()
��ʱһ���˷��������ܳ˺ܶණ��......��
������������ˣ���������
Ԫ����Ԫ����ˣ�����Ԫ�飬 ==�����Ԫ����ָ Xά �����ĸо�==
������������ ���ؾ���
==Quaternion * Quaternion== ���ر� ����1�ı�Ĳ���2�Ľ�����������1��ÿ��ı�ĽǶȣ�����2�ǵ�ǰtransform��rotation
Quaternion * Ԫ�飨������ ���ر�Quaternion�ı������
�����
https://docs.unity3d.com/Packages/com.unity.mathematics@1.2/api/Unity.Mathematics.math.mul.html#Unity_Mathematics_math_mul_Unity_Mathematics_RigidTransform_Unity_Mathematics_float4_

### .Sincos(input, Out sinValueOftheInput,Out cosValueOftheInput)
���ص�һ�������� sine��cosineֵ��
����������floatҲ�����Ǿ���

==**�����BurstCompile����������ȵ�����sin����cos����**==

### .select(a, b,bool c)
���c��true����b��false����a


# Job �������
�����Job������ָ�Լ��� **Unity.Jobs����UnityEngine.Jobs�Լ�Unity.Entities��** ����Job���͵���⡣

## ͨ����Ϣ
���Ƕ���ʵ�ֶ��̵߳Ļ����������ͣ�
���Ƕ���Ҫͨ��ʵ��Execute������ָ��Ҫִ�еĹ����߼���
���Ƕ����Ա�Schedule����Handle��Ҫ��Complete����Dependency��
���Ƕ�����Jobϵͳ���ŵ��ȹ����߳�ִ�еġ�
Run������������ִ�еģ�����һ��û��Dependency����

## ����JobHandle
JobHandle����Job��Schedule��֮��ķ���ֵ���ͣ����ڹ���Schedule�˵����Job��

����ʵ������Complete()���ڱ����õ�ʱ�����ȷ�����Job�Ѿ�����ˡ�������������������������������߳��ϣ�û������������Ҫ���Job�ڲ������ݵĻ���Ӧ�þ�����ĵ��������������Schedule��Complete֮�䣬ֻҪ���漰ʹ�ø�job���ڴ�������ݣ���дʲô�����дʲô���롣
���߻���Ϊ���������������ݴ������̻�����������ֵ��

���ľ�̬����CombineDependencies���Խ�����JobHandle�����1��JobHandle��ʵ�ֶ�������ǰ��Job��Լ����CompleteAll��������ȷ������Job������ˡ�

## �����Job����
### Unity.Jobs����Job����
**IJob** 
���ڴ��� ����Job����������Execute����û���������ֵ����ļ��ϻ���������
Schedule������͵�Job������Execute������ִ����һ���������ϡ�

**IJobFor**
���ڴ������е�ÿһ��Ԫ�ػ���һ�������ĵ�������������Execute������һ�� int i������Ϊ�������������������������ϵͳ���ŵġ�ʵ����Execute�����󿪷��߲���Ҫ�ֶ�ָ��i��

���ֱ�ӵ�������Run(int IterationCount)��������Job������ִ�У����������߳�Ҳ�����ڹ����̡߳� 


����ǵ���Schedule(Int IterationCount, jobDependency)������ᰲ���ڹ����߳���ִ�С�

����ǵ���ScheduleParallel(Int InterationCount, batchSize, jobDependency)���������ᰲ�������ɹ����߳���ִ�С�

�����IterationCount��������ָһ���������ٴΣ�����Ǵ����ϵ�ÿһ��Ԫ�أ�һ�㶼�����뼯�ϵ�length�� jobDependency��ָҪ���ĸ�Job��ɺ���ִ�����Job��

**IJobParallelFor**
�������ר�������������ϻ������ɴε������߼����ŵ����߳�ȥִ�еġ�
��û��Run()������ֱ��Schedule()

**����IJobExtensions, IJobForExtensions, IJobParallelForExtensions**
����3����������3��Job�ӿ����͵ľ�̬�ࡣʹ�÷�ʽ�Ǿ�̬����þ�̬��
Run(T JobType)��
��
Schedule(T JobType, int InterationCount, BatchSize, jobDependency)
�﷨����ͨʵ���������ƣ�����Ҫ��Job��ʵ����Ϊ��һ����������ȥ��
��Ҫ���Ƿ����ڴ�����

### UnityEngine.Jobs����Job����
**IJobParallelForTransform**
����ר�����ڴ���Transform�����Job���ͣ�Ҳ��ʵ��Execute�������ɡ�
Schedule������͵�ʱ����Ĳ����� Tranform�ļ��ϣ��������������TransformAccessArray���ͣ�TransformAccess������ΪԪ�ء�

TransformAccess��TransformAccessArray��Unity�ṩ������MonoBehevior��Jobϵͳ��ͨ���������͡�ֱ�Ӱ�Transform����TAA��������Զ���Job��ΪTransformAccess�ˡ�

### Unity.Entities����Job����
**IJobChunk**
���ڴ�������Chunk�����Job

**IJobEntity��IJobEntityExtension**
���ڴ�����entity�����Job���ײ��װ�Ļ���IJobChunk��

