### blockstates:模型显性的支配者

什么是方块状态？我们以非常通俗的方式去说就是，方块放在什么位置，出现了什么情况，这就是方块状态。

然而，游戏不可能提供6个面所对应所有方块的情况，因此，方块所处的位置情况被抽象为了几个特定的方块状态。

但是，并不是所有的方块都需要这样去判断情况，总的来说，6个面有某些不一样的方块、有朝向的方块、受某些因素控制的方块、特殊模型特殊放置的方块、需要连接的方块 才拥有方块状态。

因此，方块状态数量是内定的，是有限的。其类型和值的类型是根据方块种类固定的，详见[方块状态列表](https://wiki.biligame.com/mc/%E6%96%B9%E5%9D%97%E7%8A%B6%E6%80%81)

方块状态组成方式是：*方块状态名:方块状态值*。

![](https://z3.ax1x.com/2021/06/14/273E1U.png)

在游戏中，可以通过F3界面，查看准星所指的方块的状态。

通常情况下，一种方块状态对应多个方块状态值。方块状态值的类型一般有：枚举型(以字符串来表示)、数值型(int型整数或者float型浮点小数)、布尔型(true和false)、方向状态型(north,south,west,east,up,down)。

很容易可以得出这样的结论：只有一个方块状态的时候，组合的是线性的。如果有多个方块状态，则组合是乘积之值。

除此之外，如果需要根据方块状态来制作连接模型，很容易得出一个结论：你要穷举到死，同时模型也要写到死。

因此，minecraft给出了一个人性化的解决方案，分为两种方块状态对应模型模式：穷举(variants)和组装(multipart)。

穷举，顾名思义，就是将状态对应数值都罗列出来，然后写出对应的模型。



#### JSON基础

这里先插播一下JSON基础。

```
{

}
```

这是JSON表头，JSON必须规定在{}里，称为一个JSON{对象}。

```json
{
	"键名1":"键值",
	"键名2":"键值"
}
```

这里的键名就相当于表格的项目名，是唯一标识的，键名在同一个{}里是不可重复的。键值就是表格填入的值。键名必须严格英文双引号,接下来是一个英文冒号，键值带不带双引号根据键值类型决定。*"键名":"键值"*是一个标准的JSON形式键值对，而其他类型的键值对还有 *键名=键值* ，将会在下文以及后续的optifine属性文件中出现，请注意鉴别。

JSON提供的类型有

```json
{
    "数值型举例":1.234,
    "布尔型举例":true,
    "字符串型举例":"Hello World",
    "对象型举例":{
    
    },
    "列表型举例":[1,2,3],
    "空值型举例":null
}
```

对象{}相当于塞进了一个JSON(理论上这就叫JSON对象)，里面可以继续像这样填充键值对，即里面必须放入键值对。

列表[]用逗号隔开，且不可以是键值对，而只能是键值。(在编程中引用键值的键名就是列表名和序号，因此不需要键名)。例如"list1":[1,2,true,"haha",{"isObject": true}]在这个列表中塞入了不同种键值，甚至塞入了{}。

#### variants型状态文件

assets/minecraft/blockstates/acacia_leaves.json

```json
{
  "variants": {
    "distance=7,persistent=false":{"model": "petterfoliage:block/leaves/acacia_leaves"},
    "distance=6,persistent=false":{"model": "petterfoliage:block/leaves/acacia_leaves"},
    "distance=5,persistent=false":{"model": "petterfoliage:block/leaves/acacia_leaves"},
    "distance=4,persistent=false":{"model": "petterfoliage:block/leaves/acacia_leaves"},
    "distance=3,persistent=false":{"model": "petterfoliage:block/leaves/acacia_leaves"},
    "distance=2,persistent=false":{"model": "petterfoliage:block/leaves/acacia_leaves_1"},
    "distance=1,persistent=false":[{"model": "petterfoliage:block/leaves/acacia_leaves_1"},{"model": "petterfoliage:block/leaves/acacia_leaves_2"}],
    "persistent=true":{"model": "petterfoliage:block/leaves/acacia_leaves"}
  }
}
```

其结构大致如下

```
{
  "variants": {
    "方块处于什么状态":{"model": "模型地址"},
    "方块处于什么状态":[{"model": "模型地址1"},{"model": "模型地址2"}],
  }
}
```

可以看出，"方块处于什么状态"这一键名直接就是一个非JSON形式键值对"状态名=状态值"。请注意，这里的状态值无论是字符串类型还是其他类型，都不需要加引号。

例如"distance=1,persistent=false"，其中的逗号代表左右两边的键值对都必须匹配，是一种“且”的关系。其中"distance=1"这一状态表示的是树叶与原木的距离为1，"persistent=false"表示的是树叶是否保持不掉落的状态为否，我们在这里穷举出了自然生成的树叶(persistent=false)的所有状态，使离树干不同距离的叶子保持不同的样式。

如果方块有两个状态，省略掉其中一个，则相当于宣布，只要满足我写的这一个的状态，就要用我后面写的模型，而不管省略掉的那一个。例如上方写的"persistent=true"，而不含"distance"，表示只要满足"persistent=true"，无论"distance"等于多少，都要使用后面写的那个模型{"model": "petterfoliage:block/leaves/acacia_leaves"}。

"方块处于什么状态"这一个键名在没有状态的情况下可以省略成""。事实上默认为"normal"。例如：

assets/minecraft/blockstates/air.json

```json
{
  "variants": {
    "": {
      "model": "minecraft:block/air"
    }
  }
}
```

"方块处于什么状态"的键值是一个{模型匹配对象}，也可以是{模型匹配对象}的列表[]。

所谓{模型匹配对象}指的是

```json
{ 
    "model": "minecraft:grass_normal", 
    "x": 90, 
    "y": 270, 
    "uvlock": true, 
    "weight": 10 
}
```

<img src="https://z3.ax1x.com/2021/06/13/2o7OL8.png" style="zoom:67%;" />

其中"x"是将模型以x轴为旋转轴，从x轴正向看进去，顺时针旋转90°的倍数。"y"是以y轴为旋转轴。

"uvlock":true是锁定它的贴图，在进行"x":90,"y":270操作时，不对它的贴图进行同步的旋转，而保持原有的方向。

"weight":10是该"模型匹配对象的"的权重值。这一项仅在使用了列表时有效，默认是1。权重值可以调整该项模型被渲染的概率。也就是说，同一方块状态可以设置随机的模型。

##### 利用列表和权重设置随机模型

例如生灵早期的草

assets/minecraft/blockstates/grass.json

```json
{
    "variants": {
        "": [
			{ "model": "block/grass/grass_setaria_viridis", "weight": 3 },
			{ "model": "block/grass/grass_setaria_viridis_flower", "weight": 3 },
			{ "model": "block/grass/grass_setaria_viridis_flower2", "weight": 3 },
			{ "model": "block/grass/grass_pterocypsela_laciniata", "weight": 3 },
			{ "model": "block/grass/wild_carrot", "weight": 3 },
            { "model": "block/grass/grass_cyperaceae", "weight": 1 },
			{ "model": "block/grass/grass_chicken", "weight": 1 },
			{ "model": "block/grass/grass_minecraft", "weight": 1033 },
			{ "model": "block/grass/grass_butterfly1", "weight": 2 },
			{ "model": "block/grass/grass_butterfly1", "weight": 1, "y": 270 },
			{ "model": "block/grass/grass_butterfly1_s", "weight": 2 },
			{ "model": "block/grass/grass_butterfly1_s", "weight": 2, "y": 270 },
			{ "model": "block/grass/grass_butterfly2", "weight": 2 },
			{ "model": "block/grass/grass_butterfly2", "weight": 1, "y": 270 },
			{ "model": "block/grass/grass_butterfly2_s", "weight": 2 },
			{ "model": "block/grass/grass_butterfly2_s", "weight": 1, "y": 270 },
			{ "model": "block/lying_log", "weight": 1 },
			{ "model": "block/lying_log", "weight": 1, "y": 270 },
			{ "model": "block/grass/grass_minecraft_glowworm", "weight": 3 },
			{ "model": "block/grass/grass_minecraft_glowworm", "weight": 5, "y": 90 },
			{ "model": "block/grass/grass_minecraft_glowworm", "weight": 3, "y": 180 },
			{ "model": "block/grass/grass_minecraft_glowworm", "weight": 5, "y": 270 },
			{ "model": "block/grass/grass_minecraft_glowworm1", "weight": 2 },
			{ "model": "block/grass/grass_minecraft_glowworm1", "weight": 3, "y": 90 },
			{ "model": "block/grass/grass_minecraft_glowworm1", "weight": 3, "y": 180 },
			{ "model": "block/grass/grass_minecraft_glowworm1", "weight": 2, "y": 270 },
			{ "model": "block/grass/grass_minecraft_glowworm2", "weight": 1 }
        ]
    }
}
```

列表下的每一项显示概率为(本项权重/列表权重之和),，即权重的加权平均归一化概率。

生灵这里的设定让block/grass/grass_minecraft为概率最大的显示模型，而其余项则是偶尔出现的小惊喜。

#### multipart型状态文件

assets/minecraft/blockstates/bamboo.json

```json
{
  "multipart": [
    {
      "when": {"age": "0"},
      "apply": [
        {"model": "minecraft:block/bamboo1_age0"},
        {"model": "minecraft:block/bamboo2_age0"},
        {"model": "minecraft:block/bamboo3_age0"},
        {"model": "minecraft:block/bamboo4_age0"}
      ]
    },
    {
      "when": {"age": "1"},
      "apply": [
        {"model": "minecraft:block/bamboo1_age1"},
        {"model": "minecraft:block/bamboo2_age1"},
        {"model": "minecraft:block/bamboo3_age1"},
        {"model": "minecraft:block/bamboo4_age1"}
      ]
    },
    {
      "when": {"leaves": "small"},
      "apply": {"model": "minecraft:block/bamboo_small_leaves"}
    },
    {
      "when": {"leaves": "large"},
      "apply": {"model": "minecraft:block/bamboo_large_leaves"}
    }
  ]
}
```

这是原版的竹子的状态文件。

其中，"multipart"的键值为一个列表[]，列表的值为一个对象{}，我称呼它为{模型情况应用对象}。

所谓 {模型情况应用对象}是因为它由"when"与"apply"两个对象型键值对组成。以下是大致结构。

```
{
	"multipart": [
		{模型情况应用对象},
		{模型情况应用对象},
        {
          "when": {模型情况对象},
          "apply": [
            {模型匹配对象},
            {模型匹配对象},
            {模型匹配对象}
          ]
        },
        {模型情况应用对象}
	]
}
```

这里的"when"的值对象举出了方块状态情况，被我称作{模型情况对象}，与"variants"的"方块处于什么状态"有异曲同工之妙。不同的是，"方块处于什么状态"是一个键名，也就是说，"状态名=状态值"这是个键名罢了。而"when"是一个对象型键值对*"when":{}*，而在它的值对象下面，以"状态名"为键名，"状态值"为键值，形成一个*"状态名":"状态值"*的JSON型键值对。请注意，这里的"状态值"无论状态是字符串还是数值型、布尔型，都需要加英文双引号。这里与JSON的形式稍有不同。

```json
"when": {
	"状态1":"状态值",
	"状态2":"状态值"
}
```

这里的*"状态1":"状态值"*,*"状态2":"状态值"*是“且”的关系，当这两个情况同时发生时，这个"when"的判断成立。

状态值的键值""内，还可以用|分开两个不同的状态值，例如下面的"side|up"，表示“或”的关系，当这个状态处于side或者up时，判断都成立。

```json
"when": {
        "OR": [
          {
            "north": "none",
            "south": "none",
            "west": "none",
            "east": "none"
          },
          {
            "north": "side|up",
            "east": "side|up"
          },
          {
            "south": "side|up",
            "east": "side|up"
          },
          {
            "south": "side|up",
            "west": "side|up"
          },
          {
            "north": "side|up",
            "west": "side|up"
          }
        ]
}
```

这里的"when"的对象{}下面仅有一个"OR":[]列表，因为选择使用"OR":[]时，不可以再在"when"的对象内{}添加"状态名":"状态值"的键对值。"OR":[]列表的值也是{模型情况对象}。这里的逻辑是，每个{模型情况对象}是"或"的关系，只要条件满足"OR":[]列表内任何一个{模型情况对象}，都可以让判断成立。

"multipart"的列表[]内的每一个{模型情况应用对象}都将被判断，如果"when":{模型情况对象}判断成立，则"apply":{模型匹配对象}将被渲染到游戏中，最终由判断情况组合成新的模型。也就是说，如果选择"multipart"的状态文件，模型是散装的，不是完整的，需要把需求的模型按照方块状态拆分成各个部分。例如上方的竹子的方块状态文件中，由两个不同状态决定："age"和"leaves"。通过对不同"age"的模型与不同"leaves"的模型的拼接，最终组合成一个竹子的模型。

我们讲到"when"的值对象举出了方块状态情况，然而，举出的情况可以是互质的(对立事件，一个发生一个不发生)，例如判断同一个方块状态的时候；也可以是相互独立的(独立事件，一个发生和另一个发生无因果关系)，例如在判断两个或者多个方块状态的时候。当"when"这一键值对不存在时，被当作该{模型情况应用对象}总是发生，因而"apply"的值对象总是添加进组合后的完整模型中。

"apply"的值可以是一个{模型匹配对象}，也可以是{模型匹配对象}的列表[]。跟"variants"的一致，也是可以通过权重进行衡量出现几率。也可以通过权重来制作随机出现的模型。

```
"apply": [
    {模型匹配对象},
    { 
        "model": "minecraft:grass_normal", 
        "x": 90, 
        "y": 270, 
        "uvlock": true, 
        "weight": 10
    }
]
```

