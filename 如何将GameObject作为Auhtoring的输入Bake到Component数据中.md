�����DOTS����ʱ����ʱ�򿪷��߿��ܻ�ϣ������MonoBehaviour������ʵ����GameObjectһ����ECS�ܹ��в�������Entities��Ҫ��ʵ�����Ч�������Խ�GameObject��Transform�ȿ�ת�������������Authoring�ű��ڣ���ΪBaker�����롣Ȼ����Baking�����������IComponentData��

ʾ�����£�
```C#
// Baker��������Authoring�ű��ڲ�,û��ʲôӰ�죬ֻ���������Բ�ͬ��д����
// �������ⲿҲ��һ����
public class TurrentAuthoring : MonoBehaviour
{
    public GameObject prefab;
    public Transform spawnPoint;

    public class TurrentBaker:Baker<TurrentAuthoring>
    {
        public override void Bake(TurrentAuthoring authoring)
        {
            // ����ͨ��Dynamicֵ����Unity���Entity�ᱻ�ƶ��������LocalTransform���
            // �����GetEntity�� IBaker �ķ���
            Entity e = GetEntity(TransformUsageFlags.Dynamic);

            //�����AddComponent�� IBaker �ķ�����������APIҲ��ͬ�����������ǹ�����ͬ
            //https://docs.unity3d.com/Packages/com.unity.entities@1.0/api/Unity.Entities.IBaker.AddComponent.html
            //Ϊentity ���һ�� Component
            AddComponent(e, new TurretComponent 
            { 
                // ��Ϊ������TurretComponent�ж������洢��������Entity���ͣ���������Ҫ��Authoring������ת��ΪEntity
                CannonBallprefab = GetEntity(authoring.prefab,TransformUsageFlags.Dynamic),
                CannonBallSpawnPoint= GetEntity(authoring.spawnPoint,TransformUsageFlags.Dynamic) 
            } );
        }
    }
}
```
�ٿ���������Ӧ��IComponentData�ڲ���
```C#
/// <summary>
/// Ϊ����Turret�ܹ������ڵ�����Turret��һ����tag��Component�����һ���������ݵ�Component
/// ���д洢��2������һ������Ϊ�ڵ���Prefab����һ���ǡ��ڿڵ�λ�á�
/// </summary>
public struct TurretComponent : IComponentData
{
    // ����Ҫע�����,��DOTS�£�ʵ������������Entity������GameObject
    // ���������������Entity����Baker��Baking��ʱ��ҲҪ��Authoring������ת��ΪEntity
    public Entity CannonBallprefab;
    public Entity CannonBallSpawnPoint;
}
```

