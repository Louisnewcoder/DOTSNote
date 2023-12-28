# ����SystemState��Enabled�ֶ�
 SystemState��Enabled�ֶο��Ƶ�ǰϵͳ�ĸ���״̬���������Ϊfalse��ô ��ϵͳ�ĸ���״̬��ֹͣ��������һ֡��ʼ������ִ��OnUpdate()�Ĵ��룬ֱ��Enabled = true�� ���ǵ�ǰ֡��OnUpdate�еĴ��뽫����ִ����ϡ����������Ļ������ǾͿ���ʵ��һ�� ISystem�ĸ��½�ִ��һ�Ρ�

��Ϊ�����������һ��������Tankʵ���ģ�������������һ֡����֮��Tank�Ͳ��������ˡ�

ʾ���������£�
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
        
        // ����һ��ECB�����˳� structural change
        EntityCommandBuffer ecb = new EntityCommandBuffer(Allocator.Temp);

        // ����һ�� NativeArray���� ������װ ��Entity�����͵������������Entity����tank
        NativeArray<Entity> tanks = new NativeArray<Entity>(config.count, Allocator.Temp);
 
        // ECB ��¼ Instantiate��������һ�����ء�
        // ��һ��������entity��ԭ�ͣ��ڶ���������װʵ��������������entities������
        // Ҫע�������ECB playback ����instantiate����֮ǰ��ԭ�Ͳ�������
        ecb.Instantiate(config.tankEntity, tanks);

        // ����ecb��¼�����Ŀǰ����ֻ��  ecb.Instantiate(config.tankEntity, tanks) ��һ��
        ecb.Playback(state.EntityManager);

       // state.Enabled = false; ��������û���ǰ�����ö���һ����
    }
}
```

