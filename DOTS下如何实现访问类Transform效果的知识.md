# ��ЩComponentsӵ������GameObject Transform�����ֵ

��DOTS����ջ�£�LocalToWorld Component �� LocalTransform Component��������GameObject Transform��������ݡ�

�����ƺ�LocalToWorld�ķ���Ȩ�޸������Ƽ�readonly�� �������Գ�������Position��Rotation��Scale��ֵ���޸�/���£������Ƽ�ʹ��LocalTransform��

## ��η���LocalTransform���

����Subscene�н�һ��GameObjectת��Ϊ Entity��ʱ�����ǻᷢ�֣���EnityĬ�ϵ�Component preview��������û��LocalTransform���Component�ġ���ȡLocalTransform Component ��Ҫ�ֶ���ӡ�

ͨ����ʵ���Զ���Baker��ʱ����Ҫ��ȥEntity��ͬʱ���������°棨1.0.16 ����2023/12/25����API�ĵ�Ҫ��ָ��TransformUsageFlags��ö������ΪDynamic��==Ĭ�ϸ���TransformUsageFlags��ֵ��API��Ҫ��̭��==

`TransformUsageFlags.Dynamic`
��ζ�����Entity��Ҫ���ƶ�����ΪEntity���LocalToWorld �Լ� LocalTransform ����Components��
������ʾ�����룺
ʾ��1
```C#
/// <summary>
/// Baker��������Authoring�ű��ڲ�
/// </summary>
public class TurrentAuthoring : MonoBehaviour
{
    public class TurrentBaker:Baker<TurrentAuthoring>
    {
        public override void Bake(TurrentAuthoring authoring)
        {
            // ����ͨ��Dynamicֵ����Unity���Entity�ᱻ�ƶ��������LocalTransform���
            Entity e = GetEntity(TransformUsageFlags.Dynamic);
            AddComponent<TurretComponent>(e);
        }
    }
}
```

ʾ��2
```C#
public class TurretRotateAuthoring : MonoBehaviour { }
public class TurretRotateBaker2 : Baker<TurretRotateAuthoring>
{

    public override void Bake(TurretRotateAuthoring authoring)
    {
        //ֻ���û�ȡEntity�ķ�����ָ��TransformUsage����Ҳ����
        GetEntity(TransformUsageFlags.Dynamic); 
    }
}
```

## ����TransformUsageFlags��ʹ��

https://docs.unity3d.com/Packages/com.unity.entities@1.0/api/Unity.Entities.TransformUsageFlags.html#fields

![Alt text](image.png)