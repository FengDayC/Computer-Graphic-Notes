# GPU粒子

## Particle Pool
类似于Object Pool，另外定义两个List来存储当前存活的粒子，计算所有在Active List中的粒子，对于生命周期外的粒子就移除并且加入Dead List中。

