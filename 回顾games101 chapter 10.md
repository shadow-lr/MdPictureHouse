### 回顾games101 chapter 10-12

纹理可以用来表示环境光

Spherical Environment Map

用一个球上的点记录来自四面八方的光线

把环境光看成一个球体，然后向四周发散 拉大，这样虽然可以表示所有球上的点，但靠近极点的时候会出现扭曲

![image-20210831194614624](https://cdn.jsdelivr.net/gh/shadow-lr/MdPictureHouse/cnblogs/img/games101/image-20210831194802973.png)



把光照信息记录不记录在球上，而是记录在正方体上

![image-20210831194802973](https://cdn.jsdelivr.net/gh/shadow-lr/MdPictureHouse/cnblogs/img/games101/image-20210831194802973.png)

立方体有六个方形纹理

蓝色波浪线由凹凸贴图定义出来

假设表面法线是(0,1)

![image-20210831201723625](https://cdn.jsdelivr.net/gh/shadow-lr/MdPictureHouse/cnblogs/img/games101/image-20210831201723625.png)
$$
dp=c*[h(p+1)-h(p)]
$$
由凹凸贴图，可先确认切线，然后逆时针旋转90，最后归一化求得法线



法线贴图如何扰乱法线（How to perturb the normal (in 3D)）

- Original surface normal n(p) = (0,0,1)
- Derivatives at p are
  - dp/du = c1 * [h(u + 1) - h(u)]
  - dp/dv = c2 * [h(v + 1)  - h(v)]
- Perturbed normal is n = (-dp/du, -dp/dv, 1).normalized()



#### Bump / Normal mapping（凹凸贴图）



#### Displacement mapping（位移映射）

真的移动顶点，改变几何信息



动态细分（DirectX）

算进纹理，然后贴上，就是乘上rgb

![image-20210901010901491](D:\Typora\MdPictureHouse\cnblogs\img\games101\image-20210901010901491.png)



#### 几何



显式的表达方法

要么**直接给出**，或者**参数映射**

隐式表达方式

![image-20210901014000832](D:\Typora\MdPictureHouse\cnblogs\img\games101\image-20210901014000832.png)

SDG：signed distance function（有向距离函数）



任何一个点到这条边的最短距离，此图中可认为作垂线

blend它的边界，到中间

![image-20210901015230962](D:\Typora\MdPictureHouse\cnblogs\img\games101\image-20210901015230962.png)



水滴和水滴进行blend，然后提取出表面是怎样



### BRDF

射向一块区域dA，入射光的radiance会有多少，反射到某个方向的出射光的radiance呢

我知道有多少能量会发射到四面八方去，但我不知道有多少能量会被发散到**这个方向**去，所以此处是**能量的百分之多少**会被发射到这个方向呢

一个微小面积dA，从某个立体角dΩi接受到的Irradiance，会如何分配到各个立体角下去

已知入射光和受光点，求反射光分布的函数
$$
BRDF=\frac{dL_r\left( \omega _r \right)}{dE_i\left( \omega _i \right)}
$$

$$
f_r\left( \omega _i\rightarrow \omega _r \right) =\frac{dL_r\left( \omega _r \right)}{dE_i\left( \omega _i \right)}=\frac{dL_r\left( \omega _r \right)}{L_i\left( \omega _i \right) \cos \theta _id\omega _i}\left[ \frac{1}{sr} \right]
$$

其中式子，小块接受到的Irradiance×上BRDF，就是出射的Radiance
$$
\text{小块所接受到的dA}Irradiance\text{：}L_i\left( \omega _i \right) \cos \theta _id\omega _i
$$
![image-20210917104117990](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210917104117990.png)

### Monte Carlo Integration

黎曼积分



在积分域里面不断地采样

Monte Carlo基于大数定理



### probability density function

对定积分地估计，对随机变量采样
$$
\frac{f\left( Xi \right)}{P\left( Xi \right)}\\
P\left( Xi \right) =\frac{1}{b-a}
$$

$$
\int{f\left( x \right) dx}=\frac{1}{N}\sum_{i=1}^N{\frac{f\left( Xi \right)}{P\left( Xi \right)}}\,\,    Xi\,\,~p\left( x \right)
$$
蒙特卡洛积分做的事，我只需要在积分域内以1个pdf采样，这个样本的f(X)是多少

这里除上概率的原因是，当p越大时，就会有越来越多的fx值，那为了公平求平均时就要除以出现次数

p（Xi）可理解为在x = Xi该点，在probability density function 上对应的采样密度是多少（如何在积分域上进行采样）

解定积分

当某个区间内的p变为原来的k倍时，随机变量出现在这个区间的概率也扩大k倍，因此相当于在这个区间内采样次数扩大k倍，求平均时要除以k

如果是均匀采样，则p(x)取N/(b - a)所得

蒙特卡洛积分：不用关心积分域是多少，因为pdf已经给我们进行了提供

也可理解为平均高度*（b-a）

除以采样概率就是求一个以当前采样值为高的长方形的面积

![13](D:\文件\光线追踪\GAMES101_Lecture_16_\13.png)



### Whitted-Style Ray Tracing vs Path Tracing

Whitted-Style Ray Tracing的问题：

- 有些Glossy的物体，表面有一定的粗糙程度，不应按照specular一样，去处理出射光线



![17](D:\文件\光线追踪\GAMES101_Lecture_16_\17.png)



随机取个方向（随机变量）

对半球立体角进行采样

![22](D:\文件\光线追踪\GAMES101_Lecture_16_\22.png)

可通过蒙特卡洛积分加入到算法中，可写出对于**直接光照**来说的正确算法

![23](D:\文件\光线追踪\GAMES101_Lecture_16_\23.png)



### 直接光照

直接光照 只考虑打到光源的情况



![24](D:\文件\光线追踪\GAMES101_Lecture_16_\24.png)



Q点到P点的Radiance，就是P点所看到Q点的直接光照

![25](D:\文件\光线追踪\GAMES101_Lecture_16_\25.png)

涉及的Probability density function 没变

![26](D:\文件\光线追踪\GAMES101_Lecture_16_\26.png)

计算量会特别大

N大的时候 噪声小，N小的时候噪声大



N = 1时做蒙特卡洛积分就是path tracing，反之N != 1时做蒙特卡洛积分就是ray tracing

path tracing不仅能够解决锯齿问题，还能解决texture mapping的走样问题

![28](D:\文件\光线追踪\GAMES101_Lecture_16_\28.png)

此时忽略掉pdf(wi)是因为，pdf所能选择方向只有一个（即只bounces一个随机方向）也就是100%

![31](D:\文件\光线追踪\GAMES101_Lecture_16_\31.png)

但是此时还有个问题，光线打到某点再进行bounce的时候，在什么情况下stop呢，没有递归出口，所以也实现不了，这里可以换个概念思考，光线是有能量的，当打在某个shading point的时候，视为该点吸收了一部分能量，然后bounce，此时光线的能量（Irradiance）会削减

Russian Roulette（RR）

除以生存的概率![34](D:\文件\光线追踪\GAMES101_Lecture_16_\34.png)

按照某个特定的概率stop bounce，以一定的概率去bounce

这样求出之后，数学期望仍然为Lo
$$
E=P*\left( \frac{Lo}{P} \right) +\left( 1-P \right) *0=Lo
$$

保持期望相同，所以除p



SPP（samples per pixel）

![37](D:\文件\光线追踪\GAMES101_Lecture_16_\37.png)

很多光线浪费掉了

按照光源均匀采样

### 如何不浪费光源的采样方法呢

直接对光源进行采样，因为是对光源进行采样，蒙特卡洛积分要求积分域需要一致，也就是在同一块区域进行pdf采样和渲染方程的书写

![39](D:\文件\光线追踪\GAMES101_Lecture_16_\39.png)



dw定义是面积除以距离的平方（?）回去复习看看

对A进行采样，对A进行积分，极度舒适！

![41](D:\文件\光线追踪\GAMES101_Lecture_16_\41.png)

path tracing很难处理点光源，推荐的建议是把点光源看作成一个面积很小的面光源



### ray tracing是一个光线传输的集合

- 双向path tracing
- VCM：结合双向path tracing和photon mapping
- 如何采样（函数要怎么做采样）

![47](D:\文件\光线追踪\GAMES101_Lecture_16_\47.png)

- 选什么PDF-（重要性采样）importance sampling



### 选取随机数好的方法

- low discrepancy sequences（低差异序列）

### 多采样

- 即可以着色点往各个不同方向进行采样，也可以直接对光源进行采样（multiple important. sampling），简称MIS（多重重要性采样）

### 像素重建滤波器pixel reconstruction filter

- 对一个像素打出很多根光线，形成不同的path，计算这些path的radiance，是否是简单的平均，还是加权平均起来，那么权重要如何甄选呢，比如靠近像素中心点的权重大，靠像素两旁的权重小

### Is the radiance of a pixel the color of a pixel

- 算出来的radiance≠颜色，需要经过伽马矫正的过程（因为亮度和实际光强并不是线性关系而是对数关系）



## chapter 17

cos角度的积分

得出fr=α/pi

如何对cos的半球积分

![img](https://pic4.zhimg.com/80/v2-e993b2554da516d395c130a0eafb8423_720w.jpg)



![image-20210915163921712](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210915163921712.png)

把立体角dw换完之后，二倍角公式

反射率

![image-20210915164923436](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210915164923436.png)

抛光的金属 albert

很像镜面反射，但又不是镜面反射

入射角和出射角的方位角（三维下）

![image-20210915170102970](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210915170102970.png)



Snell's Window / Circle



### BT（transmit）DF

BRDF和BTDF统称BS（散射）DF

当入射光与物体表面法线几乎平行时，更多能量会直接穿过去 发生折射

![image-20210915182231423](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210915182231423.png)

导体的折射率是负数，导体折射率不仅要n还要k

### Microfacet Material

微表面模型（微表面材质）

**物体表面的粗糙程度可以用物体微表面的法线分布表示**

微表面模型方程
$$
f\left( i,o \right) =\frac{F\left( i,h \right) G\left( i,o,h \right) D\left( h \right)}{4\left( n,i \right) \left( n,o \right)}\\
F=Fresnel\\
G=shadowing-masking term\\
D=distribution\ of\ normals
$$
物体有多少微表面能将入射方向的光反射到某个出射方向上去，只有当微表面的法线方向与`half vector`方向完全一致的时候，这样的微表面才能把入射方向反射到出射方向

微表面被认为完全的镜子（物体的某个细微表面可看作是平面）



G又叫几何项，有可能互相遮挡

**Grazing Angle**，什么情况下，比较容易发生自遮挡、自投影的情况呢，比如光线几乎是**平着**打向平面的时候



如果光线打向一个球的时候，几乎是平行的，来修正**Grazing Angle**的

Microfacet



Isotropic：各向异性

Anisotropic：各向同性

![image-20210915204208500](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210915204208500.png)

各向同性，认为微表面并不存在一定的方向性或方向性很弱

各向异性，认为微表面存在一定的方向性或方向性强

![image-20210915204345854](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210915204345854.png)



i 输入方向 r输出方向
$$
fr\left( \theta i,\phi i,\theta r,\phi r \right) \ne fr\left( \theta i,\theta r,\phi r,-\phi i \right)
$$
如果在方位角上旋转，能得到相同的BRDF，就是各向同性，如果在方位角上旋转得到不同的BRDF，就叫各向异性，以上解释是根据BRDF去判断材质是否是各向异性或各向同性的

#### 各向同性：

- 渲染在所有方向上均匀地散射光

#### 各向异性：

- 在不同地方位角及根据表面颗粒形状的纹理而改变，一般在拉丝钢、CD光盘、头发、等划痕表面产生垂直于纹理方向的高光

![img](https://pic3.zhimg.com/80/v2-070a1a4623af5168cf50d0dd80af7922_720w.jpg)

### 天鹅绒

人为造成了各向异性的样子



### BRDF

#### non-negativity

#### Linearity：

线性，Blinn-Phong

#### Reciprocity principle 可逆性

#### Energy conservation

可能有些能量被吸收

相对方位角的绝对值

![image-20210915210815239](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210915210815239.png)

#### 各向同性vs各向异性



#### 如何是真的BRDF

- Isotropic surfaces reduce dimensionality from 4D to 3D

- Reciprocity reduces # of measurements by half
- Clever optical systems

MERL BRDF Database现成的BRDF库

Store regularly-spaced samples in (xi,xo,abs|yi-yo|)

- Better：reparameterize angles to better match specularities

Generally need to resample measured values to table

Very high storage requirements

测量材质的BRDF

