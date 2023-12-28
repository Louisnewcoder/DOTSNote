# IEnableableComponent���ص�
IEnableableComponent����������Runtimeʱ�����û���ü̳�����IComponentData���ͻ���IBufferElement���͵�ECS����� ��Ϊ��������ɾ������Ӽ̳�����IComponentData����IBufferElement���ͣ����Բ������Structural Change��

����ʾ����
���ﴴ����һ����Ϊ MyComponent��Component������������̳���IComponentData��һ������������������Ȼ���ּ̳���IEnableableComponentʹ��MyComponent���Ա����û��߽���
```C#
public struct MyComponent:IComponentData,IEnableableComponet
{ 
    // fake code 
}
```
�����IEnableableComponent����Ϊfalse����ôMyComponent�ͱ�disable�ˡ�

�����õ�Component�ᱻEntity��Ϊ�����ڡ��������MyComponent�����������ǵ���Query��ѯ��ʱ�����Entity���ϵ�MyComponent��disable����ô���Entity�Ͳ���match query���Ͳ����ҵ����������������á� ���Ƕ���EntityManager��˵���Entityʼ�մ���MyComponent���������structural change��

==����Ҳ��ӳ����������Ӱ��Ĺؼ�ԭ��ͨ�������������������ã���Ӱ��Query��ѯ�Ľ����ƥ���ƥ�䣩���Ӷ�ʵ�ֲ��֣���ȫ����ƥ���Entityִ��query��ı����������ݣ��Լ����˵����֣���ȫ������ƥ���Entity���Ӷ�����ִ��query��ı�����������==

���⣬IEnableableComponent ���û���������һ��Component��������ı���һ��Component�����ֵ��



Ӧ��ʾ����

```c#
    [WithAll(typeof(TurretComponent))] // ָ�����IJobEntity���������а���TurretComponet�� Entity��
    // ���Ա���ѯ��Component��״̬������ShootingComponent�����õ�Entity�Ͳ�ƥ���ˡ����ǻ���Ҫʱ�̼�������Ƿ������ƶ�����safezone
    [WithOptions(EntityQueryOptions.IgnoreComponentEnabledState)] 
    [BurstCompile]
    public partial struct SafeZoneJob : IJobEntity
    {
        public float SquareRadius;
        // ��ʽ����һ��ƥ��LocalToWorld����Լ�ShootingComponent�����query������ЩEntityƥ�����Ҫ��
        void Execute(in LocalToWorld transform, EnabledRefRW<ShootingComponent> shootingState)
        {
            // �ƶ����˷�Χ�Ϳ��𣨴��ڷ�Χ�������뷶Χ��ͣ������Ŀ�����ͣ����ǿ�����ر�TurretComponent
            // ����Ŀ������Entity���ϵ�ShootingComponent Ϊtrue �෴Ϊfalse
            // ��Ϊ��TurretFiringSystem��Query�У���ѯ���Ǳ���Ҫ��ShootingComponent��������TurretComponent��Entity
            // ����Ϊfalseʱ���ͱ�Entity��Ϊ���������Բ����㣬���Բ���ִ��FiringSystem�е� �ڵ�ʵ��������
            // ����Ϊ trueʱ���ͱ���Ϊ���ڣ��������㣬���Ծͻ�ִ���ڵ�ʵ�����Ĵ���
            
            shootingState.ValueRW = math.lengthsq(transform.Position) > SquareRadius;
        }
    }
```


