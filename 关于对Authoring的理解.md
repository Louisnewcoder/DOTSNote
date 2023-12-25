个人理解，Authoring的本质就是编辑用于转化为Entity的“原材料”，也就是编辑GameObject。
在传统的MonoBehaviour模式下开发处理GameObject的操作称之为编辑，而在DOTS技术框架下，在SubScene中GameObject会被转化为Entity，这个过程便称之为Authoring。

与之呼应的Authoring脚本，本质上也是一个MonoBehaviour脚本，是要挂载到“Entity”身上的。
Authoring MonoBheviours 是常规的GameObject Components。它们的作用是为baker提供生成ECS数据的输入。也就是说baking过程，会将Authoring脚本的数据，作为输入，传入给IComponentData类型的数据中。由此得到用于在DOTS技术框架下的ECS data。

下面是官方的示例代码：

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
