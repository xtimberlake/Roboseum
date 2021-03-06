#视觉里程计 Visual Odometry

[TOC]

### 1. 里程计
   在机器人导航问题中，里程计是用从运动执行器所获取的数据来对机器人在经历一段时间之后位置的变化进行估计的一种测量设备或仪器。传统的里程计比如安装在小车机器人上的光电码盘，通过脉冲技术可以确定在一段时间内车轮转过的角度，进而推算出小车行驶过的距离乃至方向。虽然这种光电码盘里程计在许多轮式和轨道机动车上应用广泛，但是它不能使用在许多采取非标准机动方式的移动机器人（比如无人机和足式机器人）上。此外，传统的里程计还有严重的精度问题。由于轮式机动车的轮胎和路面有时会发生打滑，所以轮胎转过的角度和机动车实际驶过的距离之间有时并不一致，而在一些坎坷的路面上行驶的时候产生的误差来源则更为复杂。这些误差会在里程计工作的时候不断累积，随着工作的时间越长，里程计的读数也会变得越来越不可靠。

### 2. 视觉里程计
 随着摄像头的普及，人们发明了基于摄像头信息的一种新型里程计，称之为视觉里程计。视觉里程计通过分析处理摄像头所获取的相互关联的图像序列，来确定机器人移动的位置和朝向，从而进行导航。它具有比传统的基于光电码盘的里程计具有更好的导航精度，以及没有运动方式必须是行驶在平整路面上的轮式机器人（机动车）的限定，因此被应用在许多机器人应用中，比如漫游者火星探测器上就采用了视觉里程计[1]。此外在SLAM系统（Simultaneous Localization And Mapping）中，视觉里程计也是不可缺少的一部分。

经过数十年的发展，视觉里程计从硬件设备和软件算法两个层面出发，衍生出了多种分类。
### 2.1硬件设备层面分类
根据采用摄像头类型的不同，视觉里程计可以分为单目视觉里程计与立体视觉里程计。单目视觉里程计基于单个摄像头（称为单目），只能获取单目摄像头所处的三维空间投影到其成像平面上的二维影响。然后从二维影像中通过软件算法恢复出三维的运动信息，这对软件算法的设计和实际运行的实时性、精确性造成了一定的负担。而立体视觉里程计则倾向于将主要负担交给硬件设备，即采用立体摄像头（多目摄像头、深度摄像头、激光雷达都属于这一范畴），直接对摄像头所处的三维空间进行观测，观测得到的信息在计算机中以点云形式存储，然后通过软件算法从点云的时间序列中，估计出运动信息。这种方法虽然增加了获取到信息的丰富程度，在软件算法的设计和运动估计的准确度上比基于单目摄像头的视觉里程计具有一定优势，但是增加了硬件成本和功耗，对于一些对重力和电力载荷比较敏感的机器人应用场景，则不如单目视觉里程计合适。

### 2.2 软件算法层面分类
根据采用的硬件设备的不同，视觉里程计的软件算法也需要相应地设计。因此相应地产生了两种方法，基于特征点提取（Feature Based）的方法和直接法(Direct Method)，即不提取特征点的方法。

#### 2.2.1.1 特征点法
如何根据图像来估计相机运动是视觉里程计的核心。然而，图像在计算机内部是以一个包含亮度和色彩信息组成的矩阵形式存储的，如果直接从矩阵层面考虑运动估计，将会非常困难。所以，我们习惯于采用这样一种做法——首先，从图像中选取比较有代表性的点。这些点在相机视角发生少量变化后会保持不变，所以我们会在各个图像中找到相同的点。然后，在这些点的基础上，讨论相机位姿估计问题，以及这些点的定位问题。在经典 SLAM 模型中，把它们称为路标。而在视觉 SLAM 中，路标则是指图像特征（Features）。

根据维基百科中的定义，图像特征是一组与计算任务相关的信息，计算任务取决于具体的应用 [2]。简而言之，特征是图像信息的另一种数字表达形式。一组好的特征对于在指定任务上的最终表现至关重要，因此多年来研究者们花费了大量的精力对特征进行研究。数字图像在计算机中以灰度值矩阵的方式存储，所以最简单的，单个图像像素也是一种 “特征”。但是，在视觉里程计中，我们希望特征点在相机运动之后保持稳定，而灰度值受光照、形变、物体材质的影响严重，在不同图像之间变化非常大，不够稳定。理想的情况是，当场景和相机视角发生少量改变时，我们还能从图像中判断哪些地方是同一个点，因此仅凭灰度值是不够的，我们需要对图像提取特征点。

特征点是图像里一些特别的地方。我们可以把图像中的角点、边缘和区块都当成图像中有代表性的地方。然而，我们更容易精确地指出，某两幅图像当中出现了同一个角点；同一个边缘则稍微困难一些，因为沿着该边缘前进，图像局部是相似的； 同一个区块则是最困难的。可见，图像中的角点、边缘相比于像素区块而言更加“特别”，它们在不同图像之间的辨识度更强。所以，一种直观的提取特征的方式就是在不同图 像间辨认角点，确定它们的对应关系。在这种做法下，角点就是所谓的特征。

然而，在大多数应用中，单纯的角点依然不能满足很多我们的需求。例如，从远处看 上去是角点的地方，当相机走近之后，可能就不显示为角点了。或者，当旋转相机时，角点的外观会发生变化，我们也就不容易辨认出那是同一个角点。为此，计算机视觉领域的研究者在长期研究中，设计了许多更加稳定的局部图像特征，如著名的 SIFT[3], SURF[4], ORB[5] 等等。相比于朴素的角点，这些人工设计的特征点能够拥有如下的性质：

1. 可重复性（Repeatability）：相同的“区域”可以在不同的图像中被找到。
2. 可区别性（Distinctiveness）：不同的“区域”有不同的表达。
3. 高效率（Efficiency）：同一图像中，特征点的数量应远小于像素的数量。 
4. 本地性（Locality）：特征仅与一小片图像区域相关。

特征点由关键点（Key-point）和描述子（Descriptor）两部分组成。比方说，当我们 谈论 SIFT 特征时，是指“提取 SIFT 关键点，并计算 SIFT 描述子”两件事情。关键点是指该特征点在图像里的位置，有些特征点还具有朝向、大小等信息。描述子通常是一个向量，按照某种人为设计的方式，描述了该关键点周围像素的信息。描述子是按照“外观相似的特征应该有相似的描述子”的原则设计的。因此，只要两个特征点的描述子在向量空间上的距离相近，就可以认为它们是同样的特征点。

历史上，研究者提出过许多图像特征。它们有些很精确，在相机的运动和光照变 化下仍具有相似的表达，但相应地需要较大的计算量。 其中，SIFT(尺度不变特征变换，Scale-Invariant Feature Transform) 当属最为经典的一种。它充分考虑了在图像变换过程中 出现的光照，尺度，旋转等变化，但随之而来的是极大的计算量。由于整个 SLAM 过程中，图像特征的提取与匹配仅仅是诸多环节中的一个，到目前（2017 年）为止，普通 PC 的 CPU 还无法实时地计算 SIFT 特征。所以在 视觉里程计算法的设计中我们甚少使用 这种“奢侈”的图像特征。

而另一些特征， 则考虑适当降低精度和鲁棒性，提升计算的速度。 例如 FAST 关键点属于计算特别快的一种特征点（注意这里“关键点”的用词， 说明它没有描述子）。 而 ORB（Oriented FAST and Rotated BRIEF）特征则是目前看来非常具有代表性的实时图像特征。它改进了 FAST 检测子 [6] 不具有方向性的问题，并采用速度极快的二进制描述 子 BRIEF[7]， 使整个图像特征提取的环节大大加速。 根据作者在论文中的测试，在同一幅图像中同时提取约 1000 个特征点的情况下， ORB 约要花费 15.3ms， SURF 约花费 217.3ms，SIFT 约花费 5228.7ms。由此可以看出 ORB 在保持了特征子具有旋转，尺度不 变性的同时，速度方面提升明显，对于实时性要求很高的 SLAM 来说是一个很好的选择。

大部分特征提取都具有较好的并行性，可以通过 GPU 等设备来加速计算。经过 GPU 加速后的 SIFT，就可以满足实时计算要求。但是，引入 GPU 将带来视觉里程计系统成本的提升。由此带来的性能提升，是否足以抵去付出的计算成本？这是个需要在系统设计时仔细考量的问题。在目前的视觉里程计方案中，ORB 是质量与性能之间较好的折中，其提取特征的整个过程可以在引文[8]中找到。

在提取出特征点后，下一步就是如何建立帧间特征点的关联，也就是特征匹配。特征匹配是视觉里程计中极为关键的一步，宽泛地说，特征匹配解决了视觉里程计中的数据关联问题（data association），即确定当前看到的路标与之前看到的路标之间的对应关系。通过对图像与图像，或者图像与地图之间的描述子进行准确的匹配，我们可以为后续的姿态估计，优化等操作减轻大量负担。然而，由于图像特征的局部特性，误匹配的情况广泛存在，而且长期以来一直没有得到有效解决，目前已经成为视觉里程计中制约性能提升的一大瓶颈。部分原因是因为场景中经常存在大量的重复纹理，使得特征描述非常相似。在这种情况下，仅利用局部特征解决误匹配是非常困难的。

不过，让我们先来看正确匹配的情况，再回头去讨论误匹配问题。考虑两个时刻的图像。如果在图像It中提取到特征点, 在图像It+1中提取到特征点 ，如何寻找这两个集合元素的对应关系呢？最简单的特征 匹配方法就是暴力匹配（Brute-Force Matcher）。即对每一个特征点 ，与所有的  测量描述子的距离，然后排序，取最近的一个作为匹配点。描述子距离表示了两个特征之 间的相似程度，不过在实际运用中还可以取不同的距离度量范数。对于浮点类型的描述子，使用欧氏距离进行度量即可。而对于二进制的描述子（比如 BRIEF 这样的），我们往往使用汉明距离（Hamming distance）做为度量(即两个二进制串之间的不同位数的个数)。

然而，当特征点数量很大时，暴力匹配法的运算量将变得很大，特别是当我们想要匹配一个帧和一张地图的时候。这不符合我们在 SLAM 中的实时性需求。此时快速近似最近邻（FLANN）算法更加适合于匹配点数量极多的情况。由于这些匹配算法理论已经成熟，而且实现上也已集成到开源视觉算法库OpenCV之中，其实现的技术细节可以在引文 [5] 中找到。

现在，假设通过以上的特征点提取与匹配过程，我们已经从两张图像中， 得到了一对配对好的特征点。在得到了相邻两帧图像间的一对完成配对的特征点之后，可以利用匹配的特征点对之间存在的对极约束，采用直接线性变换法（Direct Linear Transform），求解出其本质矩阵E（Essential Matrix）和基础矩阵F（Fundamental Matrix），进而根据的关系得到帧间摄像头的旋转矩阵R和平移向量t。实际中只需要在相邻两帧图像之间找到5对匹配的特征点，即可求解出摄像头在拍摄第二张照片时的位姿相对于第一张照片拍摄时的旋转和平移运动信息。
但由于E 本身具有尺度等价性，它分解得到的 t, R 也有一个尺度等价性。而 R ∈ SO(3) 自身具有约束，所以我们认为 t 具有一个尺度。换言之，在分解过程 中，对 t 乘以任意非零常数，分解都是成立的。直接导致了单目视觉的尺度不确定性（Scale Ambiguity）。换言之，在单目视觉里程计中，对轨迹和地图同时缩放任意倍数，我们得到的图像依然是一样的。为了克服尺度不确定的问题，单目视觉里程计都不可避免地需要有一个初始化的过程。初始化的两张图像必须有一定程度的平移，而后的轨迹和地图都将以此步 的平移为单位。（除了对 t 进行归一化之外，另一种方法是令初始化时所有的特征点平均深度为 1，也可以固定一个尺度。相比于令 t 长度为 1 的做法，把特征点深度归一化可以控制场景的规模大小，使计算在数值上更稳定些。不过这并没有理论上的差别。）
另一方面，从 E 分解到 R, t 的过程中，如果相机发生的是纯旋转，导致 t 为零，那么，得到的 E 也将为零，这将导致我们无从求解 R。因此，单目视觉里程计除了必须要有初始化的过程外，还要求初始化不能只有纯旋转，必须要有一定程度的平移。如果没有平移，单目将无法初始化。这些都是通过匹配特征点之间的对极几何约束估计相机运动的局限之处。

在通过提取特征点、特征点匹配、基于对极几何估计相机运动之后，还可通过三角测量（Triangulation）的方法来估计像素特征点的深度信息。从而从2D的投影图像的信息中恢复出3D的空间点。在恢复出3D的空间点之后，可以通过PnP问题（Perspective-n-Point——即当我们知道 n 个 3D 空间点以及它们的投影位置时，如何估计相机所在的位姿的问题），利用P3P法、DLT法、EPnP法、UPnP法、以及通过非线性优化来最小化重投影误差的方法, 反过来求解3D到2D点对的运动。这些方法使得基于特征点提取的运动信息估计，在对极几何求解的基础上更加准确，同时也获得了对于特征点在三维空间中位置的估计。

#### 2.2.1.2特征点法的优缺点
特征点法发展比较成熟，运行比较稳定，并且相较于直接法具有对光照、动态物体不敏感的有点，目前在视觉里程计中占据主流地位。
尽管如此，研究者们认为它至少有以下几个缺点：
1.	关键点的提取与描述子的计算非常耗时。实践当中，SIFT目前在CPU上是无法实时计算的，而ORB也需要近20毫秒的计算。如果整个SLAM以30毫秒/帧的速度运行，那么一大半时间都花在计算特征点上。
2.	使用特征点时，忽略了除特征点以外的所有信息。一张图像有几十万个像素，而特征点只有几百个。只使用特征点丢弃了大部分可能有用的图像信息。
3.	相机有时会运动到特征缺失的地方，往往这些地方都没有什么明显的纹理信息。例如，有时我们会面对一堵白墙，或者一个空荡荡的走廓。这些场景下特征点数量会明显减少，我们可能找不到足够的匹配点来计算相机运动。


#### 2.2.2.1直接法
近几年，随着深度摄像头和激光雷达等新型立体摄像头的兴起给视觉里程计的算法设计带来了新的思路，使人们有条件抛开特征点提取及其所引起的麻烦，另辟新路，基于光流提出了直接计算的方法，即根据图像的像素信息来计算相机运动，从而避免了特征的计算时间，也避免了特征缺失的情况。只要场景中存在明暗变化（可以是渐变，不形成局部的图像特征），直接法就能工作。

使用特征点法估计相机运动时，我们把特征点看作固定在三维空间的不动点。根据它们在相机中的投影位置，通过最小化重投影误差（Reprojection error）来优化相机运动。在这个过程中，我们需要精确地知道空间点在两个相机中投影后的像素位置——这也就是我们为何要对特征进行匹配或跟踪的理由。而在直接法中，最小化的不再是重投影误差，而是测量误差（Phometric error）。而根据使用图像中像素数量的不同，直接法可以分为稀疏、稠密和半稠密三种，具有恢复稠密结构的能力。相比于特征点法通常只能重构稀疏特征点，直接法和稠密重建有更紧密的联系。

##### 2.2.2.2直接法的优缺点：
相比于特征点法，直接法具有以下优点：

- 可以省去计算特征点、描述子的时间。 
- 利用了全部图像中的大部分信息。由于特征点法图像中提取的特征点信息都是稀疏的（换句话说，在图像中只有一些感兴趣的点被提取了出来，通常一幅图像中提取的信息点数目在50~500之间） 通过这一步预拣选的操作，许多有价值的信息都丢失了。
- 只要求有像素梯度即可，无须特征点。因此，直接法可以在特征缺失的场合下使用（一个极端的例子是只有渐变的一张图像。它可能无法提取角点类特征，但可以用直接法估计它的运动）。
- 可以构建半稠密乃至稠密的地图，这是特征点法无法做到的。

然而，直接法也受到传感器设备的限制，由于直接法需要以获得像素点深度为前提，所以无法在单目摄像头上使用，只能应用于深度摄像头或者激光雷达的场景。虽然计算复杂度比起特征点法显著降低了，但是对于硬件设备的要求提高了。
