# ���ͨ��ECS�ı�Entity����ɫ

���ľ���һ����ʶ ---- ��ɫҲ�����ݣ�ECS��Rendering�����ṩ�����Component �� ����Authoring�ű���

��Ҫ�ڱ༭�����ҵ��Լ�Ҫ�޸ĵ�GameObject����Prefab��Ȼ��Ϊ���ǻ��������ϵ�ĳ����������ӽű������
`URP Material Property Base Color Authoring`
�����ֿ��Կ��������Ǹ�Unity�ṩ������Bake���ʻ�ɫ��Authoring�ű������Դ򿪿�����

```C#
#if URP_10_0_0_OR_NEWER
using Unity.Entities;
using Unity.Mathematics;

namespace Unity.Rendering
{
    [MaterialProperty("_BaseColor")]
    public struct URPMaterialPropertyBaseColor : IComponentData
    {
        public float4 Value;
    }

    [UnityEngine.DisallowMultipleComponent]
    public class URPMaterialPropertyBaseColorAuthoring : UnityEngine.MonoBehaviour
    {
        [Unity.Entities.RegisterBinding(typeof(URPMaterialPropertyBaseColor), nameof(URPMaterialPropertyBaseColor.Value))]
        public UnityEngine.Color color;

        class URPMaterialPropertyBaseColorBaker : Unity.Entities.Baker<URPMaterialPropertyBaseColorAuthoring>
        {
            public override void Bake(URPMaterialPropertyBaseColorAuthoring authoring)
            {
                Unity.Rendering.URPMaterialPropertyBaseColor component = default(Unity.Rendering.URPMaterialPropertyBaseColor);
                float4 colorValues;
                colorValues.x = authoring.color.linear.r;
                colorValues.y = authoring.color.linear.g;
                colorValues.z = authoring.color.linear.b;
                colorValues.w = authoring.color.linear.a;
                component.Value = colorValues;
                var entity = GetEntity(TransformUsageFlags.Renderable);
                AddComponent(entity, component);
            }
        }
    }
}
#endif
```

����ֵ���˽���� ��������һ����ΪURPMaterialPropertyBaseColor��ECS component�����Component�������һ��float4�����ݡ����Ҳ��GPU�������ݵ���������ͣ�
�������Component��`[MaterialProperty]`���Ϊ�����õ�GameObject��`_BaseColor` ��input��

��Щ����Entities Graphics Package��Ķ���.

Ȼ������ֻ��Ҫ��System�У���д����_BaseColor���ݵ��߼����ɡ�
ʵ���Ϸǳ��򵥣�������취��URPMaterialPropertyBaseColor ����� Value ������ֵ��
����ؼ��Ĵ���ֻ��һ�У�

```C#
        foreach (var tank in tanks)
        {
            SetComponentForLinkedEntityGroup(tank, queryMask, new URPMaterialPropertyBaseColor { Value = RandomColor(ref random)});
        }

        // ���������ɫ float4�Ĵ���
        static float4 RandomColor(ref Unity.Mathematics.Random random)
    {
        // 0.618034005f is inverse of the golden ratio
        var hue = (random.NextFloat() + 0.618034005f) % 1;
        return (Vector4)Color.HSVToRGB(hue, 1.0f, 1.0f);
    }
```



Tank���ɵ�����Ƭ�Σ�
```C#
[BurstCompile]
public partial struct ConfigSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {

        // SystemState��Enabled�ֶο��Ƶ�ǰϵͳ�ĸ���״̬
        // �������Ϊfalse��ô ��ϵͳ�ĸ���״̬��ֹͣ��������һ֡��ʼ������ִ��OnUpdate()�Ĵ���
        // ֱ��Enabled = true�� ���ǵ�ǰ֡��OnUpdate�еĴ��뽫����ִ�����
        // ���������Ļ������ǾͿ���ʵ��һ�� ISystem�ĸ���ִֵ��һ�Ρ�
        // ��Ϊ�����������һ��������Tankʵ���ģ�������������һ֡����֮��Tank�Ͳ��������ˡ�

        state.Enabled = false;

        // ��Ϊ����ֻ��1��Empty GameObject ������ConfigAuthoringҲ����˵ֻ��1��ConfigComponent
        // ���������������Ϊ�����þͿ����� 
        ConfigComponent config = SystemAPI.GetSingleton<ConfigComponent>();

        // ����һ��Randomʵ�����������ȡ���ڴ���Ĳ���
        var random = new Unity.Mathematics.Random(432);

        // ����һ��Entity queryʵ��
        EntityQuery query = SystemAPI.QueryBuilder().WithAll<URPMaterialPropertyBaseColor>().Build();
        // �������query����һ�� QueryMask���������˷���������Entity
        EntityQueryMask queryMask = query.GetEntityQueryMask();

        // ����һ��ECB�����Ƴ� structural change
        EntityCommandBuffer ecb = new EntityCommandBuffer(Allocator.Temp);

        // ����һ�� NativeArray���� ������װ ��Entity�����͵������������Entity����tank
        NativeArray<Entity> tanks = new NativeArray<Entity>(config.count, Allocator.Temp);
 
        // ECB ��¼ Instantiate��������һ�����ء�
        // ��һ��������entity��ԭ�ͣ��ڶ���������װʵ��������������entities������
        // Ҫע�������ECB playback ����instantiate����֮ǰ��ԭ�Ͳ�������
        ecb.Instantiate(config.tankEntity, tanks);

        // ͨ������Ϊÿһ�� tank�޸���ɫ
        foreach (var tank in tanks)
        {
            // �����������������Entity�� ���������������ָ���� Component
            // �����ڲ�����ָ���� QueryMask��ɸѡ���ϵ�Entity��������QueryMask�������Ҿ��� URPMaterialPropertyBaseColor Component��Entity
            // �����Ķ����ᵽ��QueryMask���������query��������������EnableableComponent��chunk filtering
            // �������Ҳ����¼����ECB�У�����playbackʱ��˳��ִ�е���
            
            ecb.SetComponentForLinkedEntityGroup(tank, queryMask, new URPMaterialPropertyBaseColor { Value = RandomColor(ref random)});
            
        }

        // ����ecb��¼�����Ŀǰ����ֻ��  ecb.Instantiate(config.tankEntity, tanks) ��һ��;
        ecb.Playback(state.EntityManager);

       // state.Enabled = false; ��������û���ǰ�����ö���һ����
    }

    // �޸���ɫ���Զ��巽��
    // Helper to create any amount of colors as distinct from each other as possible.
    // See https://martin.ankerl.com/2009/12/09/how-to-create-random-colors-programmatically/
    static float4 RandomColor(ref Unity.Mathematics.Random random)
    {
        // 0.618034005f is inverse of the golden ratio
        var hue = (random.NextFloat() + 0.618034005f) % 1;
        return (Vector4)Color.HSVToRGB(hue, 1.0f, 1.0f);
    }
}
```

ͬ�����������ҲΪ�ڵ�����һ��URPMaterialPropertyBaseColorComponent������Authoring�ű���Bake��Entity����ô�ڵ�����ɫҲ���Ա����ơ��������ó���Tank��ͬ��