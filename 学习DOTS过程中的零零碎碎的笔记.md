# ��������
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
