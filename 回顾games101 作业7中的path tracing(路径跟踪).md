## 回顾games101 作业7中的path tracing(路径跟踪)

先分析代码框架

本文分析主要的流程

首先先创建几种材质

均是漫反射材质，`光源`具有自发光

```cpp
    Material* red = new Material(DIFFUSE, Vector3f(0.0f));
    red->Kd = Vector3f(0.63f, 0.065f, 0.05f);
    Material* green = new Material(DIFFUSE, Vector3f(0.0f));
    green->Kd = Vector3f(0.14f, 0.45f, 0.091f);
    Material* white = new Material(DIFFUSE, Vector3f(0.0f));
    white->Kd = Vector3f(0.725f, 0.71f, 0.68f);
    Material* light = new Material(DIFFUSE, (8.0f * Vector3f(0.747f+0.058f, 0.747f+0.258f, 0.747f) + 15.6f * Vector3f(0.740f+0.287f,0.740f+0.160f,0.740f) + 18.4f *Vector3f(0.737f+0.642f,0.737f+0.159f,0.737f)));
    light->Kd = Vector3f(0.65f);
```

往场景中添加物体，物体均是继承自Object类的`MeshTriangle`，然后构建BVH

```cpp
    MeshTriangle floor("./Homework7/Assignment7/models/cornellbox/floor.obj", white);
    MeshTriangle shortbox("./Homework7/Assignment7/models/cornellbox/shortbox.obj", white);
    MeshTriangle tallbox("./Homework7/Assignment7/models/cornellbox/tallbox.obj", white);
    MeshTriangle left("./Homework7/Assignment7/models/cornellbox/left.obj", red);
    MeshTriangle right("./Homework7/Assignment7/models/cornellbox/right.obj", green);
    MeshTriangle light_("./Homework7/Assignment7/models/cornellbox/light.obj", light);

    scene.Add(&floor);
    scene.Add(&shortbox);
    scene.Add(&tallbox);
    scene.Add(&left);
    scene.Add(&right);
    scene.Add(&light_);

    scene.buildBVH();
```
`MeshTriangle`的getArea是在构造函数对obj加载的所有顶点，三个三个组成三角形，每个三角形通过叉乘求面积加和得出

```cpp
float getArea(){
    return area;
}
```

在MeshTriangle上进行`Sample`就是对Triangle的BVH进行sample

```cpp
void Sample(Intersection &pos, float &pdf){
    bvh->Sample(pos, pdf);
    pos.emit = m->getEmission();
}
```

MeshTriangle分BVH，root为当前BVH树的根节点

```cpp
BVHAccel::BVHAccel(std::vector<Object*> p, int maxPrimsInNode,
                   SplitMethod splitMethod)
    : maxPrimsInNode(std::min(255, maxPrimsInNode)), splitMethod(splitMethod),
      primitives(std::move(p))
{
    time_t start, stop;
    time(&start);
    if (primitives.empty())
        return;

    root = recursiveBuild(primitives);

    time(&stop);
    double diff = difftime(stop, start);
    int hrs = (int)diff / 3600;
    int mins = ((int)diff / 60) - (hrs * 60);
    int secs = (int)diff - (hrs * 3600) - (mins * 60);

    printf(
        "\rBVH Generation complete: \nTime Taken: %i hrs, %i mins, %i secs\n\n",
        hrs, mins, secs);
}
```



root->area是所有BVH树下子物体的area之和

```cpp
void BVHAccel::Sample(Intersection &pos, float &pdf){
    float p = std::sqrt(get_random_float()) * root->area;
    getSample(root, p, pos, pdf);
    pdf /= root->area;
}
```

in BVH.cpp

对BVH进行Sample，这里对`random`的结果进行`sqrt`乘上，我理解是可以平缓p在函数中的上升趋势（个人理解），可能需要后续去掉`sqrt`对比结果再进一步分析

```cpp
void BVHAccel::Sample(Intersection& pos, float& pdf) {
	float p = std::sqrt(get_random_float()) * root->area;
	getSample(root, p, pos, pdf);
	pdf /= root->area;
}
```

将上一步求出的p，与节点左边和右边area进行比较，再递归下去

因为BVH节点只是帮助我们加速光线与物体的求交，节点本身并不存储关于物体的信息，当递归到叶子节点时对叶子节点上的物体`Sample`并返回

```cpp
void BVHAccel::getSample(BVHBuildNode* node, float p, Intersection &pos, float &pdf){
    if(node->left == nullptr || node->right == nullptr){
        node->object->Sample(pos, pdf);
        pdf *= node->area;
        return;
    }
    if(p < node->left->area) getSample(node->left, p, pos, pdf);
    else getSample(node->right, p - node->left->area, pos, pdf);
}
```

in bvh.cpp

因为这里加载的模型对应MeshTriangle类，其下所有的BVH节点所存储的Object都是Triangle，所以此处对问题进行了转换

对一个MeshTriangle（加载Obj格式的物体）进行采样，转换成对其所属的BVH进行采样，再转换成递归对BVH树上所有叶子节点所对应的物体进行采样，BVH树叶子节点对应的Object就是Triangle，也就是对MeshTriangle上所有的Triangle进行采样，



对Triangle采样，`pdf`为1/area，以下式子涉及到如何在三角形内随机取点，因为直接用random+rejection方法的话，当样本数量很大时，随机点并不会随机分布在三角形内

分析流程可参考这两篇：

- https://blog.csdn.net/pp87018562/article/details/119640145

- https://www.zhihu.com/question/31706710/answer/53131190

关于该算法是否均匀分布可参考这篇：https://math.stackexchange.com/questions/18686/uniform-random-point-in-triangle-in-3d

```CPP
void Sample(Intersection &pos, float &pdf){
        float x = std::sqrt(get_random_float()), y = get_random_float();
        pos.coords = v0 * (1.0f - x) + v1 * (x * (1.0f - y)) + v2 * (x * y);
        pos.normal = this->normal;
        pdf = 1.0f / area;
    }
```

in Triangle.hpp

### 奇怪点

对于叶子节点，pdf为1/area，为什么还要返回乘上node->area，node->area应该就是返回的pdf的倒数，于是这里的结果就是1，由于这种写法导致Object->Sample只会执行一次，所以看成pos就是递归中唯一的叶子节点对其自身物体（此处是Triangle）的采样

```cpp
void BVHAccel::getSample(BVHBuildNode* node, float p, Intersection &pos, float &pdf){
    if(node->left == nullptr || node->right == nullptr){
        node->object->Sample(pos, pdf);
        pdf *= node->area;
        return;
    }
    if(p < node->left->area) getSample(node->left, p, pos, pdf);
    else getSample(node->right, p - node->left->area, pos, pdf);
}
```

### sample函数

sample(const Vector3f wi, const Vector3f N) in Material.cpp: 按照该材质的性质，给定入射方向与法向量，用某种分布采样一个出射方向

根据sample函数，可以看到在某个材质下，光线射向该着色点时，出射方向并不与入射方向有联系，只与该点的法向量有关系，因为漫反射就是有可能向四面八方去进行反弹，所以对半球上所有出射方向进行采样，

```cpp
Vector3f Material::sample(const Vector3f& wi, const Vector3f& N)
{
	switch (m_type)
    {
	    case DIFFUSE:
	    {
		    // uniform sample on the hemisphere
		    float x_1 = get_random_float(), x_2 = get_random_float();
		    // z belongs to [-1, 1]
		    float z = std::fabs(1.0f - 2.0f * x_1);
		    // r belongs to [0, 1]
		    // phi 半球的立体角是2pi
		    float r = std::sqrt(1.0f - z * z), phi = 2 * M_PI * x_2;
            // 均匀平滑r的分布 [0, 1]
            // 采样大小和方向
		    Vector3f localRay(r * std::cos(phi), r * std::sin(phi), z);
		    return toWorld(localRay, N);
            
		    break;
	    }
	}
}
```

in Material.hpp



侧边绿色不显示是 因为没有修改成tenter<=text，看了bbs的帖子，因为此处的绿色墙壁是一个平面，包围盒的高度为0，所以存在**tenter＝texit的情况**

![image-20210915090130981](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210915090130981.png)

![image-20210915090134895](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210915090134895.png)



需修改材质的`pdf`函数，修改dotProduct(wo, N) > 0为dotProduct(wo, N) > -EPSILON

```cpp
float Material::pdf(const Vector3f &wi, const Vector3f &wo, const Vector3f &N){
    switch(m_type){
        case DIFFUSE:
        {
            // uniform sample probability 1 / (2 * PI)
            if (dotProduct(wo, N) > -EPSILON)
                return 0.5f / M_PI;
            else
                return 0.0f;
            break;
        }
    }
}
```

思考题，光线弹射停止的时候的期望

![img](http://games-cn.org/wp-content/uploads/2020/04/E7731D15-CADD-4F04-92CE-C1DE800E4110.jpeg)

![binary_pt_test](D:\FFOutput\binary_pt_test.png)

提高题：



#### 公式中BRDF的分析：



![image-20210917125922194](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210917125922194.png)

f_r：brdf项，在作业框架中可以看到是与材质相关，本作业中所有的材质均是漫反射材质，所以初始化的时候设置了一个漫反射项系数Kd（`Vector3f`类型），这里`Vector3f diffuse = Kd / M_PI`是取x，y，z每个分量在半球上的每度的均值，我理解为Kd是指材质本身在`red、green、blue`上能吸收多少能量在半球上，那么在某个三维角度（x，y，z）上吸收多少能量，故每个分量除以`PI`（个人理解）



漫反射项除以`PI`，是为了**能量守恒**，更详细的分析过程请看这篇

**为什么PBR中Lambert光照要除PI?**：https://zhuanlan.zhihu.com/p/29837458

```cpp
/// <param name="wi">入射</param>
/// <param name="wo">出射</param>
/// <param name="N">法向量</param>
/// <returns>return diffuse; Vector3f diffuse = Kd / M_PI;</returns>
Vector3f Material::eval(const Vector3f &wi, const Vector3f &wo, const Vector3f &N){
    switch(m_type){
        case DIFFUSE:
        {
            // calculate the contribution of diffuse   model
            float cosalpha = dotProduct(N, wo);
            if (cosalpha > -EPSILON) {
                Vector3f diffuse = Kd / M_PI;
                return diffuse;
            }
            else
                return Vector3f(0.0f);
            break;
        }
        case SPECULAR:
        {
            break;
        }
        case REFRECT:
        {
            break;
        }
    }
}
```

#### 在半球上采样的方法

一种比较好的办法是`cosine-weighted-sampling`，关于此处采样是否均有分布，网上找到了类似的写法，但没看到完全相同的，可能需要后续加以验证此处是否真的`分布均匀`

![image-20210917140323859](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210917140323859.png)

可以看看这几篇博文：

通过 concentric 映射在单位圆盘上生成具有良好分布的样本：https://blog.csdn.net/u010333737/article/details/104561622

PBRT学习笔记：在单位圆内部均匀采样：https://blog.csdn.net/codeboycjy/article/details/6225886?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link

圆面、球面上的采样方法：https://blog.csdn.net/yjr3426619/article/details/102706968?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link

#### 基于着色点的半球随机向量生成

作业框架中涉及这部分的代码如下

```cpp
	// uniform sample on the hemisphere
	float x_1 = get_random_float(), x_2 = get_random_float();
	// z belongs to [-1, 1]
	float z = std::fabs(1.0f - 2.0f * x_1);
	// r belongs to [0, 1]
	// phi 半球的立体角是2pi
	float r = std::sqrt(1.0f - z * z), phi = 2 * M_PI * x_2;
    // 均匀平滑r的分布 [0, 1]
    // 采样大小和方向
	Vector3f localRay(r * std::cos(phi), r * std::sin(phi), z);
	return toWorld(localRay, N);
```

in Material.hpp

这部分代码在 [Ray Tracing: The Rest of Your Life](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html)中有所涉及，不过与作业框架中的代码不同

```cpp
	float r_1 = get_random_float(), r_2 = get_random_float();
	float x = std::cos(2 * M_PI * r_1) * 2 * std::sqrt(r_2 * (1 - r_2));
	float y = std::sin(2 * M_PI * r_1) * 2 * std::sqrt(r_2 * (1 - r_2));
	float z = 1 - 2 * r_2;
	Vector3f random_point(x, y, z);
```

对于统一随机数 r1 和 r2，一维MC集成一章的内容如下:
$$
r1=\int_0^{\theta}{\frac{1}{2\pi}dt}=\frac{\phi}{2\pi}
$$
求解$\phi$我们得到：
$$
\phi =2\pi ·r_1
$$
对于θ我们有：
$$
r_2=\int_0^{\theta}{2\pi f\left( t \right) \sin \left( t \right) \mathrm{d}t}
$$
首先让我们在球体上尝试均匀密度。单位球的面积是$4Π$，所以单位球上的一个均匀的$p(direction)=\frac{1}{4pi}$。
$$
r2=\int_0^{\theta}{2\pi \frac{1}{4\pi}\sin \left( t \right) \mathrm{d}t=\int_0^{\theta}{\frac{1}{2}\sin \left( t \right) \mathrm{d}t=\frac{-\cos \left( \theta \right)}{2}}-\frac{-\cos \left( 0 \right)}{2}}=\frac{1-\cos \left( \theta \right)}{2}
$$
求解cos(θ)给出：
$$
\cos \left( \theta \right) =1-2r_2
$$
我们不求解，因为我们可能只需要知道cos(θ)，而不希望不必要的arccos()调用到处运行

为了生成指向(θ, φ)的单位矢量，我们转换为笛卡尔坐标
$$
x=\cos \left( \phi \right) ·\sin \left( \theta \right) 
\\
y=\sin \left( \phi \right) ·\sin \left( \theta \right) 
\\
z=\cos \left( \theta \right)
$$
使用恒等式$cos^2+sin^2=1$，我们得到以下随机的$(r_1,r_2)$
$$
x=\cos \left( 2\pi ·r_1 \right) \sqrt{1-\left( 1-2r_2 \right) ^2}
\\
y=\sin \left( 2\pi ·r_1 \right) \sqrt{1-\left( 1-2r_2 \right) ^2}
\\
z=1-2r_2
$$
参考自：

- https://zhuanlan.zhihu.com/p/371231759

- https://blog.csdn.net/freesouths/article/details/90733738

- https://www.freesion.com/article/44151278901/

#### ToWorld函数

此处的BCN看成是TBN就好了，将切线空间下的随机单位向量变换成世界空间坐标系的三维向量，然后再参与运算

![image-20210918183246880](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210918183246880.png)



### Cook-Torrance光照模型

该光照模型是基于物理材质的光照模型。光照射到物体表面发生漫反射、镜面反射、折射、透射等现象，在这里我们只考虑漫反射和镜面反射，Cook-Torrance是用来模拟不同材质的镜面反射效果。


$$
f_{cook-torrance}=\frac{DFG}{4\left( w_o·n \right) \left( w_i·n \right)}
$$
D：(Normal Distribution Function)法线分布函数

F：(Fresnel)菲涅尔方程

G：(Geometry Function)几何函数

- **正态分布函数**：估算在受到表面粗糙度的影响下，法线方向与中间向量一致的微平面的数量。这是用来估算微平面的主要函数。
- **几何函数**：描述了微平面自成阴影的属性。当一个平面相对比较粗糙的时候，平面表面上的微平面有可能挡住其他的微平面从而减少表面所反射的光线。
- **菲涅尔方程**：菲涅尔方程描述的是在不同的表面角下表面所反射的光线所占的比率（不同介质之间折射和反射的比率）。

介绍可参考这篇博客：

- https://zhuanlan.zhihu.com/p/160804623
- 图形学｜PBR：Schlick近似方法：https://zhuanlan.zhihu.com/p/372110183
- 基于物理着色（二）- Microfacet材质和多层材质：https://zhuanlan.zhihu.com/p/20119162
- LearnOpenglPBR：https://learnopengl-cn.github.io/07%20PBR/01%20Theory/
- 物理渲染学习笔记（三）——Cook-Torrance微表面模型：https://gameinstitute.qq.com/community/detail/123254

### 存在的问题

- Cook-Torrance模型无法表示，在粗糙度为0的情况下，物体表面绝对光滑（呈现镜面渲染的效果）

在BBS上翻到遇到类似问题的人：https://schuttejoe.github.io/post/ggximportancesamplingpart1/



#### 解决过程

<blockquote>
    The first technique I want to write about is to importance sample only D(ω^m). This is quite effective since the distribution of normals does have a significant impact on shape of the entire BRDF. To importance sample D(ω^m) we will use the inverse of the CDF of D(ω^m) to generate a microfacet normal ω^m. If you are familar with using GGX in game rendering this can be thought of as creating a half vector in tangent space. The derivation of the inverse of the CDF is covered in detail in Cao Jaiyin’s post here so I’ll just refer you to that for the details and list the relevant equations here:
</blockquote>

我想写的第一个技术是只对D(ω^m)进行重要性采样。这是非常有效的，因为法线的分布对整个BRDF的形状有很大影响。为了对D(ω^m)进行重要性采样，我们将使用D(ω^m)的CDF的逆值来生成一个微面正态ω^m。如果你对在游戏渲染中使用GGX很熟悉的话，这可以被认为是在切线空间中创建一个半矢量。CDF的逆推法在曹佳音的帖子里有详细的介绍，所以我只想让你参考一下，并在这里列出相关的方程式。
$$
\theta _m=arc\cos \sqrt{\frac{1-\xi _0}{\xi \left( \alpha ^2-1 \right) +1}}\,\,       \phi =2\pi \xi _1
$$

<blockquote>
    Now, if you remember how importance sampling works, you’ll know that we are eventually going to need to divide our result by the pdf of the sample we take. As we will see the convienance of later, the pdf for generating the spherical coordinates is very similar to the distribution of normals itself. From Cao’s post we have:
</blockquote>

现在，如果你还记得重要性抽样的工作原理，你就会知道，我们最终需要用我们的结果除以我们抽取的样本的pdf。正如我们稍后将看到的便利，生成球面坐标的pdf与法线分布本身非常相似。从Cao的帖子中我们可以看到。
$$
p\left( \theta _m,\phi \right) =\frac{\alpha ^2\cos \theta _m\sin \theta _m}{\pi \left( \left( \alpha ^2-1 \right) \cos ^2\theta _m+1 \right) ^2}
$$

<blockquote>
    The next step we’ll want to take is to transform these spherical coordinates into Cartesian coordinates. I think if you are familar with everything I’ve said so far you probably already know the equations for this but I’ll list them for the sake of completeness. With one last reminder that I am using Y-up, here they are:<br>
    where r = 1 in this case. This gives us ω^m.

</blockquote>

下一步我们要做的是将这些球面坐标转换为直角坐标。我想如果你对我到目前为止所说的一切都很熟悉的话，你可能已经知道了这方面的方程式，但为了完整起见，我还是要把它们列出来。最后提醒一下，我使用的是Y-up，它们在这里。
$$
x=r\sin \theta \cos \phi 
\\
y=r\cos \theta 
\\
z=r\sin \theta \sin \phi
$$

在这种情况下，r=1。这就给了我们$ω^m$






#### OpenMp实践记录

OpenMp并行化执行代码可参考这几篇：

- https://blog.csdn.net/drzhouweiming/article/details/1175848

- https://zhuanlan.zhihu.com/p/51173703

- https://www.cnblogs.com/gooneybird/p/13509307.html

