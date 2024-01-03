v1.0 2023.12.30 first edition

# DOTS技术架构的关键组成
ECS系统，Jobs系统，Burst编译器
# 关于ECS系统
## Entities
Entities 的本质是一个标识符，并不具备任何数据。它代表着在个DOTS下有若干Components组成的一组数据，而这组数据由Entity表现为游戏世界的一个元素（GameObject）。如果将一个GameObject放入subscene，在subscene中，这个GameObject就是一个Entity的感念而不是一个GameObject。

## Components
Components 的本质是数据。在游戏中代表着某个游戏元素身上的一个特点，可能是速度，可能是攻击力，可能是生命..... 若干Components在一起形成一个Archetype，可以把这个Archetype理解为某一类游戏元素，可能是英雄，可能是坦克，可能是树，也可以是这些游戏元素的一个分支...... 相同的Components组合属于一类Archetype，这Components的种类和数量有关，与它们的存储的值的大小多少无关。

## Systems
Systems 的本质是游戏逻辑或者数据计算规则。同时在于Jobs系统互动时， 负责管理和调度Job。它负责对符合开发者指定的query描述的entities按照开发者的代码进行操作。在DOTS架构内担任着类似MonoBehaviour的角色，但是却不需要挂载到GameObject身上。

另外需要注意的是，system只能处理它所归属的World中的entities

query描述 是开发者指示 System 对哪些==数据即Component==进行操作的指示。这与面向对象开发是完全不同的。同样的Components组合可能会出现在不同的Archetypes中，所以query描述的设计必须要准确，否则可能很容易会出现意外的现象。比如，如果在面向对象的开发模式中，我想操作飞机只需要明确指定飞机这个类的实例就可以了，不会影响到坦克。但是在DOTS着这种面向数据的开发模式中，我要操作的是移动Component和转向Component，如果我只查询这两个组件，那么飞机，坦克，人类都有可能带有这两个组件，都会被影响。所以要在为游戏元素设计Components组合的时候要考虑在未来query的时候如何能够更轻松的排除不必要的Archetypes，精准的找到自己想要的。

# Jobs系统
因为传统MonoBehaviour架构下的Unity API几乎都是执行在主线程的，游戏应用很难充分发挥硬件平台的多线程多核性能优势。Jobs系统就是帮助开发者和游戏应用充分利用多核多线程优势，提高游戏性能的。 当于ECS的System配合时，Job负责执行具体的游戏逻辑，然后由System管理与分配。

# Burst编译器
通过将代码编译为native code从而实现提高性能的效果。