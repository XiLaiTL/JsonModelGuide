### models:首先理解模型文件，再理解建模

我们先盲猜一下模型应该由几个部分组成。现实生活中，模型必要部分是结构与绘图。在计算机中，渲染并不是由天然的光线进行的，而是由计算的光线组成的。这就需要我们为模型提供有关渲染的信息。因此，游戏中的模型大致分为3个部分，即结构、绘图以及渲染。在minecraft使用的模型中，还包含了模型间的继承关系等。

模型文件放置在assets/namespace/models/下，但是，根据我们第一节讲到的，models/item下的模型有独特的性质。这种性质类似于blockstates下的方块状态文件，用于引导物品模型的渲染。

物品模型文件和方块模型文件是共通的，但又有所区分。其主要原因除上述因素，还有一点是，游戏中的物品与方块有截然不同的性质。

首先，物品分为两个应用场景和多个渲染场景。物品会在物品栏中以"item"形式存在以至于接受/give,/replaceitem(1.17以后为/item),/clear,/data等命令，在掉落物中，以"minecraft:item"这一实体形式存在。除此之外，物品可以被以放入物品展示框的方式渲染出来，相同的效果还有盔甲架的各个位置，实体的头部、左右手工具等。渲染上，还有营火上以及物品栏GUI和第一第三人称左右手。

其次，方块的渲染与方块间的位置关系密切相关。这种位置关系决定了一些面剔除，即面进行渲染的情况，同时决定了方块渲染的阴影、面亮度等。同时方块状态很大程度上的改变是由于方块位置关系决定的，因此方块位置关系及其影响方块模型的构造。而物品渲染则仅仅局限于上述应用情景，不需要考虑模型间的位置关系等。

最后，方块类型的物品模型和方块模型可以相互继承，但是在文件夹中必须至少提供两个文件，即blockstates/状态文件和models/item/物品模型文件。

#### 物品模型

一个物品模型分为6个键值对，即6个属性，其中的"parent":""即继承关系，"textures"以及"elements":[{"faces":{}}]为绘图部分，"elements":[{元素对象}]为结构部分，而剩下的都为渲染部分。

```
{
	"parent":"minecraft:block",
	"textures":{
		"材质变量1":"minecraft:block/stone",
		"材质变量2":"材质的地址",
		"particle":"minecraft:block/stone"
	},
	"elements":[
		{元素对象1},
		{元素对象2}
	],
	"display":{
		"gui": {模型显示对象},
        "ground": {模型显示对象},
        "fixed": {模型显示对象},
        “head”: {模型显示对象},
        "thirdperson_righthand": {模型显示对象},
        "thirdperson_righthand": {模型显示对象},
        "firstperson_righthand": {模型显示对象},
        "firstperson_lefthand": {模型显示对象}
	},
	"gui_light":"front",
	"override":[
		{覆写模型对象1},
		{覆写模型对象2}
	]
}
```

当然如果使用blockbench的话，会再添加两个对象，下文会提到。

- "parent":""前面已经提到过，即父模型的地址，标记本模型继承的对象。物品的继承关系将在下文提到。

- "textures":{}是材质引用地址的对象。里面填充着后续"elements":[]列表内元素使用到的"#材质变量"，但是填写在这里时要去掉#号，其对应的键值是你所要使用的"材质地址"。其中不要忘记了"particle":"粒子材质地址"这一项，这一项将是你破坏方块、掉落物品等产生的粒子效果的材质来源。元素使用到的对应材质变量地址在本文件中可以省略，但一定要在后续的继承中补充上材质变量地址。而如果父级材质变量地址写上之后，子级可以写也可以不写。如果写，则会覆盖父级的材质地址，如果不写，则会继续使用父级的材质地址。如果模型中缺少了使用到的材质变量地址，那么在游戏中会渲染出紫黑色的缺失材质样式。

- "elements":[]是构成模型的元素列表。其值为{元素对象}。就像搭积木一样，我们的整体的模型就是由这样的元素搭建而成。{元素对象}如下


assets/minecraft/block/grass_block.json/"elements":[]

```json
{
    "from": [0, 0, 0],
    "to": [16, 16, 16],
    "faces": {
        "north": {"uv": [0, 0, 16, 16], "texture": "#side", "cullface": "north"},
        "east": {"uv": [0, 0, 16, 16], "texture": "#side", "cullface": "east"},
        "south": {"uv": [0, 0, 16, 16], "texture": "#side", "cullface": "south"},
        "west": {"uv": [0, 0, 16, 16], "texture": "#side", "cullface": "west"},
        "up": {
        	"uv": [0, 0, 16, 16], 
        	"texture": "#top", 
        	"cullface": "up", 
        	"tintindex": 0
        },
        "down": {"uv": [0, 0, 16, 16], "texture": "#bottom", "cullface": "down"}
    }
}
```

其实还包括旋转条目，完整的对象如下

```
{
    "from": [x1, y1, z1],
    "to": [x2, y2, z2],
    "faces": {
        "north": {材质渲染对象},
        "east": {材质渲染对象},
        "south": {材质渲染对象},
        "west": {材质渲染对象},
        "up": {
        	"uv": [x3, y3, x4, y4], 
        	"texture": "#材质变量", 
        	"cullface": "up", 
        	"tintindex": 0,
        	"rotation":90
        },
        "down": {材质渲染对象}
    },
    "rotation":{
    	"origin":[x5,y5,z5],
    	"axis":"x",
    	"angle":22.5,
    	"rescale":"false"
    }
}
```

可以看出，一个元素对象拥有4个属性。但是在方块模型中，还增添了一个属性，"shader":true，但是加了"shader"这一条目后并不影响这个模型是方块模型还是物品模型。后续讲解。

- 

  - "from"与"to"的值都是一个列表[]，分别是元素起点的坐标[x1, y1, z1]和终点的坐标[x2, y2, z2]。两个坐标即可决定一个长方体元素，这也侧面说明了一个元素就是一个长方体或者长方形平面。然而，minecraft的体素模型是限制在3x3x3的方格范围内的，这意味着，"from"与"to"的坐标范围是-16~32。通过"from"与"to"的两个坐标，我们既决定了元素的坐标位置——即"from"的坐标，又决定了元素的大小——即"to"坐标与"from"坐标的差值。

    "from"的坐标理应该小于"to"的坐标值(x,y,z都应该小于)。然而当"from"与"to"的某坐标值相同时，元素为一平面；某两个坐标值相同时，元素为一直线(无意义)；三个坐标值都相同时，元素为一点(无意义)。在minecraft中，渲染是以面的形式进行的，因此面是渲染的最小单位。线与点都没有意义。当"from"中有一个坐标值小于"to"的时候，这时候模型的渲染翻转到了立方体的内部，"to"坐标与"from"坐标的差值为负数，因此被称为负模型。负模型在一些地方可以代替正面4个元素的渲染，可以减少元素个数起到优化作用；当"from"中有两个坐标值小于"to"时，这时候模型的渲染又翻转回来， 是“正”模型；当"from"中三个坐标值都小于"to"时，模型还是翻转到了立方体内部去了。

  - "faces":{}记录着六个面的渲染信息，也因此，里面直接是6个属性："north","south","west","east","up","down"。每一个属性值都是一个{材质渲染对象}。

    {材质渲染对象}中有5个属性。为元素的某一个面完整地贴上材质贴图。因此会涉及到引用材质的哪个部位啊，引用哪个材质啊，怎么引用材质啊等关系。

    - 
      - "uv":[x3, y3, x4, y4] 即是引用材质贴图的坐标位置。从贴图的左下角(x3,y3)引用到贴图的右上角(x4,y4)。注意，对于高分辨率材质来说，是缩放的位置，坐标范围为0~16。而对于组合类型的材质，还未验证[wiki](https://wiki.biligame.com/mc/%E6%A8%A1%E5%9E%8B#.E7.A4.BA.E4.BE.8B.EF.BC.9A.E5.B0.86.E5.A4.9A.E4.B8.AA.E6.9D.90.E8.B4.A8.E7.BB.84.E5.90.88.E5.88.B0.E4.B8.80.E4.B8.AA.E6.96.87.E4.BB.B6)上这样写是否正确。如果左坐标填写了在右坐标上面的位置或者右边的位置，贴图方向将被翻转。这也是对贴图进行上下镜像操作、左右镜像操作的方法。
      - "texture":"#材质变量" 这一个属性与上级"textures":{"材质变量":"地址"}相互对应，这里填写的"#材质变量"是属性值，而上级"材质变量"是键名，且少了一个#号。在上级必须填写出地址，这里才能够保证材质被渲染到模型的相应部件，否则会渲染出紫黑块，在游戏日志中报错材质丢失。
      - "rotation":90 其值为90度的倍数。这个操作是作用于贴图渲染的，但不会真正改变原材质本身的方向，仅仅是对渲染的结果进行旋转，即将uv所指定的位置的材质贴图旋转多少度，再贴到模型上。贴图渲染的旋转即是在这里操作的。
      - "cullface"与"tintindex"将在后文介绍。

  - "rotation":{}此rotation非上面那个rotation。这个{旋转对象}是作用于模型的。

    - "origin":[x5,y5,z5]枢纽点坐标。枢纽点的意思是进行旋转的中心点，这个点可以在模型外，也可以在模型内，并且坐标范围可以扩展到无限大。也就是说，枢纽点是可以突破模型限制的！
    
    - "axis":"x"可选值为"x","y","z"也就是绕着经过枢纽点的某个轴进行旋转。
    
    - "angle":22.5旋转22.5度。可选值为22.5度的倍数，范围为-45~45度。经测试，为逆时针旋转。
    
    - "rescale":false是否进行模型的缩放。(缩放的原理和效果有待研究)。经过测试，"rescale":true进行的模型缩放与上述三个属性都有关系。如果"axis"的属性不存在，则默认为"y"。经过测试，模型的缩放方向取决于"axis"，缩放大小取决于"angle"。其中，对于一个大小为16x16的元素面"angle":22.5将缩放为接近完整方块的大小；"angle":45将缩放为完整方块的大小，也就是相当于将底边变为16*1.414的长度。如图所示。
    
      <img src="https://z3.ax1x.com/2021/07/01/RyAQm9.png" style="zoom:67%;" />



**override:物品模型显性的支配者**

事实上，物品模型也有类似方块状态的“物品状态”，记录在物品的NBT中。所有物品共有的“物品状态”为CustomModelData，而其他的“物品状态”则为特定物品才拥有。这里的“物品状态”(NBT)在模型文件中对应的键值名被称为物品谓词，例如，CustomModelData对应"custom_model_data"。而特定的物品与谓词大概有：指南针的角度、时钟的时间、工具的损耗与损耗值、末影珍珠与紫颂果的冷却时间、弓和弩的拉与拉伸程度、鞘翅是否破损、盾是否在格挡、钓鱼竿是否抛掷。所有有效的物品谓词以及其值见[物品谓词](https://wiki.biligame.com/mc/%E6%A8%A1%E5%9E%8B#.E7.89.A9.E5.93.81.E6.A0.87.E7.AD.BE.E8.B0.93.E8.AF.8D)。特殊的是，"lefthanded"对应左手玩家使用的模型，但我还未知其的效用。

- "override":[]是覆写模型情况列表，其值为{覆写模型对象}。这一属性里的对象决定了物品模型什么情况渲染什么模型。{覆写模型对象}如下。

```
{ "predicate": { "物品谓词": 物品谓词值 }, "model": "模型地址" },
{ "predicate": { "物品谓词": 物品谓词值,"物品谓词": 物品谓词值 }, "model": "模型地址" }
```

当满足"predicate":{}下所有物品谓词情况时，显示对应的模型，也就是说，这些"物品谓词":""的条件是一种“且”的关系。

当然，只有规定好的models/item/下的某些物品模型文件可以填写这一列表，详见第二节。其表现为，如果列表中的对象引用的模型也有覆写模型情况列表的话，该引用的模型的列表无效。也就是说，这里规定好的物品模型文件就相当于方块状态文件，用于决定物品模型的渲染情况。也因此可以极大地拓展物品模型。

**利用CustomModelData拓展物品模型**

本节内容请见 [森罗万象](https://sqwatermark.gitee.io/resguide/vanilla/model/item_tags.html) ，例如在第二节中我们讲到

assets/minecraft/models/item/carrot_on_a_stick.json

```
{
  "parent": "minecraft:item/handheld_rod",
  "textures": {
    "layer0": "minecraft:item/carrot_on_a_stick"
  },
  "overrides": [
    { "predicate": { "custom_model_data": 9960001 }, "model": "minecraft:ibt/magic_stick"},
    { "predicate": { "custom_model_data": 9960002 }, "model": "minecraft:ibt/magic_stick_1"},
    { "predicate": { "custom_model_data": 9960003 }, "model": "minecraft:ibt/magic_stick_2"}
  ]
}
```

我们通过对胡萝卜钓竿的物品模型的修改，添加了"custom_model_data": 9960001条目。

第一个物品magic_stick可以通过命令/give @p carrot_on_a_stick{CustomModelData:9960001}得到。也可以通过编写数据包中的战利品表以使物品通过/loot命令得到。详见[原版模组入门教程](https://zhangshenxing.gitee.io/vanillamodtutorial/#%E7%89%A9%E5%93%81%E8%AE%BE%E8%AE%A1)。

**以物品方式渲染**

这里重点挑出有关模型以物品方式渲染的有关条目。

```
	"display":{
		"gui": {模型显示对象},物品栏、背包等gui中显示的模型
        "ground": {模型显示对象},掉落物显示的模型
        "fixed": {模型显示对象},展示框显示的模型
        “head”: {模型显示对象},用命令带在头上显示的模型(含盔甲架)
        "thirdperson_righthand": {模型显示对象},第三人称右手显示的模型
        "thirdperson_lefthand": {模型显示对象},第三人称左手显示的模型
        "firstperson_righthand": {模型显示对象},第一人称右手显示的模型
        "firstperson_lefthand": {模型显示对象}第一人称左手显示的模型
	},
	"gui_light":"front"
```

- "display":{}中的属性为各种情况与该情况下的{模型显示对象}，这里用于调整模型在各个情况下的缩放状况、位置和旋转角度。模型显示对象如下

assets/minecraft/models/item/generated.json/"display":{}

```
"ground": {
            "rotation": [ 0, 0, 0 ],
            "translation": [ 0, 2, 0],
            "scale":[ 0.5, 0.5, 0.5 ]
        }
```

- 

  -  

    - "rotation": [x1,y1,z1]模型分别以x,y,z为旋转轴旋转的角度，填入[x1,y1,z1]中。大小为-180~180。在这里模型突破了前面旋转中的22.5°的倍数的限制！也就是说可以旋转任意角度！

    - "translation":[x2,y2,z2]模型分别在x,y,z轴上移动多少距离，填入[x2,y2,z2]中。大小为-80~80。大于80则当成80。

    - "scale":[x3,y3,z3]模型分别在x,y,z轴上缩放多少倍，填入[x3,y3,z3]中。大小为-4~4其中负数为镜像后再缩放。也就是说，缩放倍数最大可以放大到原模型的4倍！这里的原模型的大小是在各个情况下渲染的大小，各个情况不尽相同。例如掉落物的模型大小的一倍与方块大小相同，而展示框的模型大小一倍为方块大小的一半。(各边长的一半)

      例如原来minecraft中限制模型大小为3X3X3，经过缩放在展示框中可以为6X6X6，而掉落物直接就是12X12X12，盔甲架的头部为7.5X7.5X7.5,小盔甲架大约为6X6X6。

      也就是说，利用"display":{}可以提供展示框和盔甲架中的相比方块限制中更小限制的物品模型！

- "gui_light":""在GUI中的光照渲染模式。可选值有"side"和"front"。"side"即侧置光照，光源从物品侧面照过去，留下立体的物品渲染；"front"即前置光照，光源从物品前面照过去，留下正面的物品渲染，绕y轴旋转时模型受光照的方向不变。

**物品继承关系**

方块的继承关系我们前面已经讲到，这里我们讲讲物品的继承关系。

物品继承关系表。(from chyx)

```
{
    "builtin/generated":{提供拉伸模型,相当于elements
        "item/generated": {提供display的样式,为item提供显示，跟block.json作用一样
            "item/bow": {},弓
            "item/crossbow": {},弩
            "item/handheld": {工具
                "item/handheld_rod": {钓鱼竿
                    "item/carrot_on_a_stick": {},
                    "item/fishing_rod": {}
                }
            },
            "item/template_spawn_egg": {}刷怪蛋
        }
    }
}
```

首先是内建生成物品拉伸模型，就是你只要画张贴图，会给你建一个增厚一层的模型

这里的"builtin/generated"为物品模型提供了"elements":[],而且元素使用的贴图的材质变量为"layer0"。而"item/generated"与"block/block"一样，为模型提供了"display":{}的所有属性，相当于说，如果你继承了"item/generated",你只要在你的物品文件里写上：

```json
{
  "parent": "minecraft:item/generated",
  "textures": {
    "layer0": "minecraft:texture_name"
  }
}
```

就可以完成一个物品模型的构建了！绝大多数的非方块物品的模型是这样的，也是因此，你可以通过修改"item/generated"的"display":{}以达到修改大多数非方块物品的渲染样式的目的，当然，弓、弩、钓鱼竿和工具的渲染也得手动修改。修改方式将在后续章节讲述。

物品继承关系表。(from chyx)

```
{
    "builtin/entity":{
        "item/chest": {},箱子
        "item/template_banner": {},旗帜
        "item/template_bed": {},床
        "item/template_shulker_box": {},潜影盒
        "item/template_skull": {}头颅
    }
}
```

其次是内建生成的实体渲染模型，将实体模型渲染成物品模型，这些类型是有限的，局限于上述5个大类和盾牌、三叉戟、潮涌核心，并且严格限定使用于指定名称的文件。同样地，这些物品模型中都只是提供"display"，如没有下级，则还得提供"textures":{"particle":"粒子材质地址"}

#### 方块模型

方块模型有5个属性，少了"override"。而且少了"gui_light"并且多了"ambientocclusion"。事实上两者都填写并不干扰模型的正常运作，只是渲染的时候用在的地方不一样罢了。这个将会在后文讲解。在元素列表的元素对象中多了"shader"，不填写也不影响模型正常运作。

```json
{
	"parent":"minecraft:block",
	"ambientocclusion":true,
	"textures":{},
	"elements":[
        {
            ........,
            "shader":true
        }
    ],
	"display":{}
}
```

**以方块方式渲染**

这里重点挑出有关模型以方块方式渲染的有关条目。

```json
{
	"ambientocclusion":true,
	"elements":[
		{
            "from": [x1, y1, z1],
            "to": [x2, y2, z2],
            "faces": {
                "up": {
                    "uv": [x3, y3, x4, y4], 
                    "texture": "#材质变量", 
                    "cullface": "up", 
                    "tintindex": 0
                },
                "down": {材质渲染对象}
            },
            "shader":false
        }
	]

}
```

![](https://z3.ax1x.com/2021/07/04/RW1SB9.png)

- "ambientocclusion":true调整环境光遮蔽。true为开启环境光遮蔽，默认开启，且本模型被继承的时候，本模型的环境光遮蔽效果如果被开启了，后续模型再关闭也没用。有关环境光遮蔽的详细信息请看 [森罗万象](https://sqwatermark.gitee.io/resguide/vanilla/model/ambientocclusion.html) [相关博文](https://www.mcbbs.net/thread-1062742-1-1.html)。环境光遮蔽效果很大程度上决定了模型的光照渲染。

- 

  - 

    -  "cullface":"up"进行面剔除的面，值可以是"up","down","north","south","west","east"。它的作用是，当本方块(注意是方块而不是元素)对应方向有[固体方块](https://wiki.biligame.com/mc/%E5%9B%BA%E4%BD%93%E6%96%B9%E5%9D%97)存在时，该元素的这个面不进行渲染，而不管元素这个面是哪个面。(事实上并不只是固体方块，[详见](https://www.mcbbs.net/forum.php?mod=redirect&goto=findpost&ptid=1079039&pid=19183273))例如"down":{......,cullface":"up"}这里元素的地面开启了对上面的面剔除，当本方块上面放置有固体方块时，本方块的本元素的地面将不会被渲染。一般情况下选择面剔除指定的面都是相应面，例如"up":{......,cullface":"up"}，但是，如果面进行旋转的话，大多数情况下无法这样一一对应，这里要主义一下。

      如果不进行设置的话，将不开启面剔除。

      当环境光遮蔽关闭时，决定面渲染的将是"cullface"。如果没有开启面剔除，则光照会变得奇怪；如果开启面剔除，则"cullface"指定的朝向面的亮度将作为本面的亮度([WIKI如此](https://minecraft.fandom.com/wiki/Model) "It also determines the side of the block to use the light level from for lighting the face, and if unset, defaults to the side. ")。因此，如果为了将模型的奇怪光照关闭的话，记得为模型的突出部分添加"cullface"以使光线更加自然。

    - "tintindex":0着色序数。有关着色的详细信息请看 [森罗万象](https://sqwatermark.gitee.io/resguide/vanilla/model/tintindex.html#%E7%A1%AC%E7%BC%96%E7%A0%81%E7%9D%80%E8%89%B2%E6%B5%81%E7%A8%8B) 。据我的观察与经验，原版中使用到着色的分为两类，而使用到的序数均为0，这一项只要存在，即使值为1也会硬编码着色。一类是植物根据生物群系染色，染色效果和原理详见森罗万象上述章节，另一类是物品的染色，包括刷怪蛋和药水等。当然还有红石粉的染色这样特殊的存在。

      硬编码只能适用于特定的blockstates下指定的方块模型和models/item下特定的物品模型，其他模型即使填写该项也不会进行染色。当编辑植物的方块模型的时候，如果不填写这一项，则该元素不会被染色而是保留材质原本的颜色。例如草方块模型中，只有顶部和边缘的贴图才会被染色，而泥土部分不会被染色。

  -  "shade":true开启阴影，默认开启。环境光遮蔽是相对于方块来说的差异效果；而阴影更多的还与元素有关系，对元素进行进一步的阴影渲染。如果开启环境光遮蔽的情况下关闭阴影，则不会渲染一个元素从上到下的阴影效果，而会继续渲染不同元素间的光照差异。然而如果关闭环境光遮蔽，开启阴影一定程度上可以提供较为真实的光照阴影效果。

**利用“多余”的方块物品模型文件拓展物品模型**

与物品模型相同的是，方块模型也可以添加"display"。这一项目对于方块模型来说没有作用。但是可以被继承。在minecraft原版文件中，方块物品的物品模型就是直接一句{"parent":"minecraft:block/stone"}。简单直接粗暴。

于此同时，我们可以看出来，方块模型和物品模型文件是可以共用的，然而对于游戏的注册来说，并没有必然联系。也就是说，同一个方块放置出来的效果和以物品显示的效果根本没有关系，仅仅只是因为原版文件中物品继承了方块模型而已。

因此，我们可以利用这一特点，利用方块物品这一“无用”的模型来制作模型的拓展包。例如[ITP](https://www.mcbbs.net/thread-461822-1-1.html)。

#### blockbench增加的属性

Blockbench为模型的json文件提供了组与命名的操作，而组还可以规定旋转的枢纽点以整体旋转、整体位移等。

```json
{
	"credit": "Made with Blockbench",
	"groups": [0,
		{
			"name": "wart_block_1",
			"origin": [8, 8, 8],
			"children": [
				{
					"name": "group",
					"origin": [9, 27, 18],
					"children": [
						{
							"name": "group",
							"origin": [9, 27, 18],
							"children": [1, 2]
						},
						{
							"name": "group",
							"origin": [9, 27, 18],
							"children": [3, 4]
						}
					]
				}
			]
		}
	]
}
```

"credit":""是每次使用blockbench编辑都会产生的，你也可以改成你的昵称，例如"By Xiao2. From Petter Foliage. Made with Blockbench"。不过你用完blockbench还是会给你改掉就是了。(还没找到内置修改credit的方法)

"group":[]用于归并元素的组合，元素的组将显示在blockbench的右下方。列表的值为数字和{组对象}。这个数字是元素的组序号，也就是元素在"elements":[]列表中的序号。组对象如下:

```
{
    "name": "组名",
    "origin": [9, 27, 18],
    "children": [1, 2]
}
```

"name":""是显示在blockbench右下方的组名。

"origin":[x,y,z]是一个有3个数字的列表，这三个数就是本组总枢纽点，用于相对于组的旋转操作。

"children":[]本组的成员，可以是数字，也可以是{组对象}，跟递归有点像，文件夹嘛。同上。

了解Blockbench里面的组键值对关系有助于之后手动进行组的合并以及排序等操作，手动操作有时候更加方便。

