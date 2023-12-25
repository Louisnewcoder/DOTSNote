������⣬Authoring�ı��ʾ��Ǳ༭����ת��ΪEntity�ġ�ԭ���ϡ���Ҳ���Ǳ༭GameObject��
�ڴ�ͳ��MonoBehaviourģʽ�¿�������GameObject�Ĳ�����֮Ϊ�༭������DOTS��������£���SubScene��GameObject�ᱻת��ΪEntity��������̱��֮ΪAuthoring��

��֮��Ӧ��Authoring�ű���������Ҳ��һ��MonoBehaviour�ű�����Ҫ���ص���Entity�����ϵġ�
Authoring MonoBheviours �ǳ����GameObject Components�����ǵ�������Ϊbaker�ṩ����ECS���ݵ����롣Ҳ����˵baking���̣��ὫAuthoring�ű������ݣ���Ϊ���룬�����IComponentData���͵������С��ɴ˵õ�������DOTS��������µ�ECS data��

�����ǹٷ���ʾ�����룺

```C#
    // Authoring MonoBehaviours are regular GameObject components.
    // They constitute the inputs for the baking systems which generates ECS data.
    class TurretAuthoring : MonoBehaviour
    {
        // Bakers convert authoring MonoBehaviours into entities and components.
        public GameObject CannonBallPrefab;
        public Transform CannonBallSpawn;

        class Baker : Baker<TurretAuthoring>
        {
            public override void Bake(TurretAuthoring authoring)
            {
                // GetEntity returns the baked Entity form of a GameObject.
                var entity = GetEntity(TransformUsageFlags.Dynamic);
                AddComponent(entity, new Turret
                {
                    CannonBallPrefab = GetEntity(authoring.CannonBallPrefab, TransformUsageFlags.Dynamic),
                    CannonBallSpawn = GetEntity(authoring.CannonBallSpawn, TransformUsageFlags.Dynamic)
                });

                AddComponent<Shooting>(entity);
            }
        }
    }

    public struct Turret : IComponentData
    {

        // This entity will reference the prefab to be instantiated when the cannon shoots.
        public Entity CannonBallPrefab;

        // This entity will reference the nozzle of the cannon, where cannon balls should be spawned.
        public Entity CannonBallSpawn;
    }
```
