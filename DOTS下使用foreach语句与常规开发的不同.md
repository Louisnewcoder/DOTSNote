# һ�������ƶ����������

```C#
    [BurstCompile]
    public partial struct TankMoveSystem:ISystem
    {
        [BurstCompile]
        public void OnUpdate(ref SystemState state)
        {
            // һ�������foreach�ڵ����ı�����ֻ���ģ�������д�����
            // ���ǣ���ʹ��SystemAPI��Query��ѯʱ�������Ŀ��Component���Ϊ RefRW �ɶ�д
            // �Ϳ���ʵ�ֶԵ���������д�����
            foreach (var transform in SystemAPI.Query<RefRW<LocalTransform>>().WithAll<TankComponent>())
            {
                Debug.Log(transform.GetType());
                float3 pos = transform.ValueRO.Position;

                float angle = (0.5f + noise.cnoise(pos / 10f)) * 4.0f * math.PI;
                Debug.Log(angle.GetType());

                var dir = float3.zero;
                math.sincos(angle, out dir.x, out dir.z);

                // ��Ϊ�ڲ�ѯ���LocalTransform���Ϊ RefRW��������Զ�transform��position��ֵ����д��
                // �������ֱ��ʹ�� transform.Position�ͻ��յ�"Ŀ���ǵ�������"�ı���
                transform.ValueRW.Position += dir * SystemAPI.Time.DeltaTime * 5.0f;
                transform.ValueRW.Rotation = quaternion.RotateY(angle);
            }
        }
    }
```

���������չʾ����DOTS�ܹ��£������һ��System��ͨ��query���ҿɿ����ƶ����洢�ƶ�������ݣ������`LocalTransform`����������SystemAPI.Query�������ص㣬��Ŀ������ΪRefRWʵ�ֶԵ���Ŀ��ġ����¡��������޸���Ŀ��Entity��position��rotation��ֵ��ʵ����Ŀ����ƶ���