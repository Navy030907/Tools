**关于PAM文件的研究，并非我本人独立完成，它是由我(smallpc)同时漪、PAK向日葵、伊特几人一同破解出来的**

# 使用 SPC-Util 进行PAM文件的解码与编码

SPC-Util 可以将PAM文件转换为XFL文件目录，它等价于FLA动画文件，可使用Adobe Animate打开并编辑。

在编辑XFL完成后，可使用 SPC-Util 将之重新编码为PAM文件，打包到游戏数据包中即可实现动画修改。

出于一些原因，SPC-Util公开版不会将PAM文件直接转换为含有全部信息、能够近乎无损逆向到PAM的JSON数据文件，而是转换成整合过的XFL动画文档。

## 解码PAM动画的方法

1. 解包数据包，得到PAM文件，使用SPC-Util打开该文件，选择 "PAM-Convert-To-XFL-OneStep" 选项(输入它前面的数字)后回车，程序会解码PAM并生成后缀为.xfl的文件夹。解码完毕。

2. 拆分这个动画所对应的Atlas(需要1536规格的图像)，得到分解图(PNG)，将所有分解图复制到第一步中生成的xfl文件夹下的LIBRARY文件夹内。

3. 使用 Adobe Animate 软件打开xfl文件夹下的Anim.xfl文件，即可看到解析出来的动画。

## 编码生成PAM动画的方法

在An中对动画进行修改、保存后，使用SPC-Util打开这个xfl文件夹，选择 "PAM-Convert-From-XFL-OneStep" 选项(输入它前面的数字)后回车，程序会根据其中的内容编码出可在PvZ2游戏中运行的PAM动画文件。

需要说明的是，目前SPC-Util可处理的XFL动画有诸多限制，例如：只能使用PNG位图、不允许渐变、只能进行逐帧制作(可以使用传统补帧、并在要进行转码前在An中将至转化为逐帧动画)。具体说明参考下文。如果违反如下规则，很有可能无法成功将XFL转换为PAM（程序进行转换时将会崩溃）。

# 动画主体结构说明

动画的主场景被命名为 "Anim"，他有四个Layer，从上到下依次为：

* labels : 标签层，用于设置帧标签，游戏程序会通过帧标签定位到需要播放的动画。

* actions : 定义动作命令以及stop指令

	* stop 指令：即指令 stop() ，这会使动画的播放停止，通过帧标签与stop的配合，可以使游戏播放二者之间的动画。

		> 例如：在花园中种植植物，浇水后会触发“浇水动画”，游戏会读取该植物的PAM，得到动画数据，并通过帧标签"water"找到浇水动画所在位置，从该帧开始播放，直至播放到含有 stop指令 的帧，这样就播放了一次浇水动画。

	* 其他动作指令：fscommand命令，游戏可读取这种命令，并使对象做出指定的行为：例如：fscommand("use_action"); 指令，可以使射手类植物射出一枚子弹、使生产型植物生产一次阳光。

* audio : fscommand命令，用于播放音效，形如：fscommand("PlaySample", eventName); ，其中，eventName是SoundBank中定义的Event的字符串ID。

* animation : 动画层，这一层直接调用了 ID为"A_Main"的“影片剪辑”元件，要修改动画内容，应直接步入改元件进行修改，不要改动这一层的数据。

# 元件说明

制作动画时，只能使用“影片剪辑”类型的元件。通过这种元件的嵌套、组合，在A_Main元件中进行修改。

## “影片剪辑”类型元件的分类与命名规则

* M元件：iMage（图像）

	M元件中，只能有一个层级、这个层级上的内容只有一帧，并且这一帧中必须且只能包含一个“位图”类型元件，包含之后，这一位图可以使用如下属性：

	* 指定变换原点、平移、缩放。

	这三种属性以外的属性，会在编码PAM过程中被忽略。

* A元件：Animate（动画）

	A元件中，可以包含任意数量的层级（即使是0），每一层级中，必须且只能有一段连续的动画，这段连续动画中，每帧都必须始终引用同一“影片剪辑”元件，“连续动画”之前，可以存在任意数量的空白帧，使前N帧保持空白。“连续动画”的最后一帧之后，不可再存在任何帧(空白帧、或引用其他元件的非空白帧)。

	* 可以调用任何M元件，也可以调用A元件，但所调用A元件的序号必须小于当前A元件的序号。例如 有 M_0 ~ M_20 与 A_0 ~ A_10 元件，A_5元件可以调用所有M元件与A_0 ~ A_4 元件，不能调用 A_5 ~ A_10 元件。

	A元件可以有以下几种属性：
	
	* 平移、缩放、旋转、倾斜。

	这四种属性以外的属性，会在编码PAM过程中被忽略。

* 命名规则：

	M元件的元件名，必须严格按照格式 M_%u 填写，其中 %u 表示这个M元件的序号，序号从0开始计数，且不可中断，例如：M_0、M_1、M_2......M_9。

	A元件的命名同理。A_Main元件是特殊的A元件，他始终存在，并只能使用 A_Main 这个名称。

* 导入一张位图

可转换为PAM的XFL动画中，只能包含位图（并且为PNG格式）。

位图导入XFL库的步骤如下：

1. 在AN中，将一张PNG图像导入到库中，并进行重命名，规则为：只能由数字、字母、下划线组成，不包含文件后缀(如.png)。

2. 根据上文中的M元件命名规则，新建一个“影片剪辑”类型的元件；新建成功后，在这个M元件的第一层、第一帧中，加入第一步中添加到库里的位图，可以为其指定缩放与平移属性。

3. 打开 xfl 文件夹 下的 OtherInfo.json 文件，在 "ImageSize" 对象的最后一个成员之后，添加新的成员，成员Key为图像ID，对应resources.json中的"ID"，图像ID只由大写字母、数组、下划线组成；成员值是由两个无符号整数(不可带小数点)组成的数组，两个数分别表示图像的宽高。

	> 例如： "IMAGE_TEST_1": [ 11, 22 ] 。表示ID为IMAGE_TEST_1、高度11像素、宽度11像素的图像。

4. 在 "ImageMapper" 对象的最后一个成员之后，添加新的成员，成员Key为第二版新建的M元件名，成员Value为第三步所添加的图像ID。

# 创建新的A元件

1. 在AN中，创建一个“影片剪辑”元件，其命名符合A元件命名规则。

2. 打开 OtherInfo.json ，在 "SubAnimMapper" 对象的最后一个成员之后，添加新的成员，成员Key为第一步所新建的A元件的名称，成员值为该A元件的标签名。

	> A元件的标签可用于<元件屏蔽>。游戏可在播放动画时，指定某些标签的A元件不被显示。游戏中的“植物装扮切换”就是这个原理。

# 注意点

## 转换XFL为PAM时的注意事项

在转换为PAM之前，需要打开XFL文件夹内等 DOMDocument.xml 文件，并在第一行 DOMDocument 文字后面添加文字： __ABOUT__="This XFL is convert from PAM file, By SPC-Util." (ABOUT两边有双下划线)，才能使用SU进行转换。

## 关于图像文件的尺寸

1. 标准尺寸：OtherInfo中指定的图像尺寸：是按照 标准尺寸常数 定义的宽高属性，必须是一个大于0的整数。

2. 分辨率缩放比：XFL库中的位图文件，都应使用同一分辨率，默认情况下由转换出的XFL，使用的是 1536 分辨率。分辨率值会与 标准尺寸常数 复合，在M元件中定义了图像的缩放数值。

> 图像文件实际尺寸应等于 标准尺寸 乘以 分辨率缩放比的倒数，计算结果舍去小数部分。
>
> 1536 分辨率的缩放比是 78.125 % (百分数)。所以 若导入一张 标准尺寸 为 100x100 的位图，实际图像文件的尺寸应为 100x100 乘以 1/0.78165(约为1.28) = 128x128

