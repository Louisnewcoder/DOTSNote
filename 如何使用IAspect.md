# DOTS Aspect�� IAspect���͵��ص�
�������Ϳ��԰� �������ڴ���ĳ��Entityʱ��ص�Component��ϵ�һ�𣬷����ڷ��ʵ�ʱ����Ը���ݣ���ʡ����ռ䡣

�������Ļ��ƣ��������ɣ�IAspect��������һ��Archetype������Ҫ��ѯ�������Archetype����Ҫ�������ڲ����������Components����ϡ�
����һ��IAspect ������ Component A B C3�֡� Ȼ��Entity1 ��Component AC�� Entity2��ComponentB��Entity3 ��ABC��
ֻ��Entity3�ᱻ�ҵ���

Ҫע����ǣ������struct��Ϊ����������һ��IAspect���ͣ�����Ҫ��partial�ؼ������Σ����� readonly�������������Ҫ������һ���ֶ��� RefRO��RefRW��ǵ�Component�������ݣ�������Ƕ��һ��aspect��

![Alt text](image-1.png)

ʾ�����룺
```C#
// ����IAspect���͵�ʱ�������struct��Ϊ������ô����Ҫ��partial�ؼ�������
// ͬʱ������readonly ���Σ���������������ڣ�����Ҫ��һ��RefRO����RefRW���͵�����
 readonly partial struct TurretAspect : IAspect
{
    // ����һ��RefRO��ǵ�TurretComponent Component����
    readonly RefRO<TurretComponent> a_Turret;

    // ����������Entity�������涨���a_Turret�ڵ����� Entityֵ���и�ֵ
    
    public Entity CannonBallEntity => a_Turret.ValueRO.CannonBallprefab;
    public Entity CannonBallSpawnPoint => a_Turret.ValueRO.CannonBallSpawnPoint;

    // ���ˣ������߾Ϳ��Խ�ͨ���������IAspect���͵�����ֱ�ӻ�� TurrentComponent ��ص�������

    // ������һ�в��Դ��룬�����������Ͷ��������IAspect���棬�����沢û���κ�ʵ��߱����Component
    // ����Turret�Ͳ��ᷢ���ڵ���
    //readonly RefRO<TestAspectComponent> a_TestA;
}
```

# ��Query��ʹ��IAspect
ֱ�ӿ�����ʾ����

```C#
[BurstCompile]
public partial struct TurretFiringSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        // �����ﳢ��ʹ��IAspect���ͣ��øն����TurretAspect
        // �����Query���ҵ��Ǿ߱� [TurretAspect���涨���ȫ��Components] �Լ� [LocalToWorld Component]��entities
        foreach (var (turret, localToWorld) in SystemAPI.Query<TurretAspect,RefRO<LocalToWorld>>())
        {   
            //����ҵ��ˣ�������EntityManagerͨ�� TurretAspect���CannonBallEntityֵ��ʵ����һ�� CannonBall��entity����
            Entity instance = state.EntityManager.Instantiate(turret.CannonBallEntity);
            // Ϊ��ʵ����������entityʵ����LocalTransform Component���ݽ��и�ֵ
            state.EntityManager.SetComponentData(instance, new LocalTransform
            {
                // ���������˼�����á��ڿڡ�entity��LocalToWorld��Position��ֵ����Ϊ��ʵ����������CannonBall��λ�õ�ֵ
                Position = SystemAPI.GetComponent<LocalToWorld>(turret.CannonBallSpawnPoint).Position,
                Rotation = quaternion.identity,
                // Scale�����Լ���Scale
                Scale = SystemAPI.GetComponent<LocalTransform>(turret.CannonBallEntity).Scale
            });

            // �ٸ�����CannonBallComponent ���ݽ�������
            state.EntityManager.SetComponentData(instance, new CannonBallComponent
            {
                speed = localToWorld.ValueRO.Up * 20f
            });
        }
    }
}
```
�����¼һ�£�
�Ҵ���δ����п�������������ֶ�ͨ������ʵ����Entity�Ļ�����ʵ��������֮����Ҫʹ��EntityManager��SetComponent(Entity entity, ComponentType component) �����entity�����Components�������ǽ������á�


## ��¼��
CannonBall Component������Authoring��Baking
```C#
public struct CannonBallComponent : IComponentData
{
    public float3 speed;
}

// Authoring & Baking
public class CannonBallAuthoring : MonoBehaviour
{
}

public class CannonBallBaker : Baker<CannonBallAuthoring>
{
    public override void Bake(CannonBallAuthoring authoring)
    {
        AddComponent<CannonBallComponent>(GetEntity(TransformUsageFlags.Dynamic));
    }
}

```