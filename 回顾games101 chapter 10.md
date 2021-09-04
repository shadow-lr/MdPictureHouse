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

