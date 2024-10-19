
# 技术背景


在前面的一篇[博客](https://github.com)中，我们介绍了拉格朗日插值法的基本由来和表示形式。这里我们要介绍一种拉格朗日插值法的应用场景：格点拉格朗日插值法。这种场景的优势在于，如果我们要对整个实数空间进行求和或者积分，计算量是随着变量的形状增长的。例如分子动力学模拟中[计算静电势能](https://github.com)，光是计算电荷分布函数都是一个O(N2)的计算量，其中N表示点电荷数量。而如果我们对空间进行离散化，划分成一系列的格点，再对邻近的常数个格点进行插值，那么我们的求和计算量可以缩减到O(N)。


# 格点拉格朗日插值


给定一个函数y\=f(x−xr)，我们可以将其插值到最近的4个整数格点上：⌊xr⌋−1\.5,⌊xr⌋−0\.5,⌊xr⌋\+0\.5,⌊xr⌋\+1\.5，根据拉格朗日插值形式有：


yinterp\=c1(x)f(⌊xr⌋−1\.5−xr)\+c2(x)f(⌊xr⌋−0\.5−xr)\+c3(x)f(⌊xr⌋\+0\.5−xr)\+c4(x)f(⌊xr⌋\+1\.5−xr)如果以⌊xr⌋最近的中心点为原点，即⌊xr⌋\=0，则其系数有：


c1(x)\=(x−⌊xr⌋\+0\.5)(x−⌊xr⌋−0\.5)(x−⌊xr⌋−1\.5)−6\=148(−8x3\+12x2\+2x−3)c2(x)\=(x−⌊xr⌋\+1\.5)(x−⌊xr⌋−0\.5)(x−⌊xr⌋−1\.5)2\=116(8x3−4x2−18x\+9)c3(x)\=(x−⌊xr⌋\+1\.5)(x−⌊xr⌋\+0\.5)(x−⌊xr⌋−1\.5)−2\=116(−8x3−4x2\+18x\+9)c4(x)\=(x−⌊xr⌋\+1\.5)(x−⌊xr⌋\+0\.5)(x−⌊xr⌋−0\.5)6\=148(8x3\+12x2−2x−3)其图像大致如下图所示（图片来自于参考链接1）：



![](https://img2024.cnblogs.com/blog/2277440/202410/2277440-20241017095907185-562800121.png)

对于多维的格点拉格朗日插值，则是一个叉乘的关系，其图像为：

![](https://img2024.cnblogs.com/blog/2277440/202410/2277440-20241017101756639-1767135991.png)

# 远程相互作用项的截断


我们把上面得到的这个格点拉格朗日插值应用到静电势能的计算中。在前面一篇[博客](https://github.com)介绍的静电势计算中，有一项电荷分布函数是这样的：


s(k)\=\|S(k)\|2\=N−1∑i\=0qie−jkriN−1∑l\=0qlejkrl其中S(k)\=∑N−1i\=0qiejkri\=∑N−1i\=0qiejkxxiejkyyiejkzzi。把后面这几个指数项用格点拉格朗日插值替代得：


S(k)\=N−1∑i\=0qi∑x,y,z\[c1(x)f(⌊xi⌋−1\.5−xi)\+c2(x)f(⌊xi⌋−0\.5−xi)\+c3(x)f(⌊xi⌋\+0\.5−xi)\+c4(x)f(⌊xi⌋\+1\.5−xi)]\[c1(y)f(⌊yi⌋−1\.5−yi)\+c2(y)f(⌊yi⌋−0\.5−yi)\+c3(y)f(⌊yi⌋\+0\.5−yi)\+c4(y)f(⌊yi⌋\+1\.5−yi)]\[c1(z)f(⌊zi⌋−1\.5−zi)\+c2(z)f(⌊zi⌋−0\.5−zi)\+c3(z)f(⌊zi⌋\+0\.5−zi)\+c4(z)f(⌊zi⌋\+1\.5−zi)]有了函数形式以后，我们可以简写S(k)为一个关于三维空间格点的求和：


S(k)\=N−1∑i\=0qi⌊xmax⌋\+1\.5∑mx\=⌊xmin⌋−1\.5⌊ymax⌋\+1\.5∑my\=⌊ymin⌋−1\.5⌊zmax⌋\+1\.5∑mz\=⌊zmin⌋−1\.5cmx(mx)ejkxmxcmy(my)ejkymycmz(mz)ejkzmz再把系数项单独拿出来：


Q(mx,my,mz)\=N−1∑i\=0qicmx(mx)cmy(my)cmz(mz)这里的Q其实是一个shape为(Nx,Ny,Nz)的张量，而mx,my,mz对应的是某一个格点的张量索引，每一个索引对应的张量元素都是通过系数函数计算出来的，有了这样的一个概念之后，再重写S(k)的函数：


S(k)\=⌊xmax⌋\+1\.5∑mx\=⌊xmin⌋−1\.5⌊ymax⌋\+1\.5∑my\=⌊ymin⌋−1\.5⌊zmax⌋\+1\.5∑mz\=⌊zmin⌋−1\.5Q(mx,my,mz)ejk⋅m我们会发现，这个插值出来的S(k)函数其实是在计算张量Q在k处的傅里叶变换，那么就可以进一步简写S(k)的形式：


S(k)\=VF∗k(Q)(mx,my,mz)其中F∗表示逆傅里叶变换，V表示逆傅里叶变换归一化常数。按照前面的4\-格点拉格朗日插值法，此时得到的S(k)的值是一个shape为（4，4，4）的张量，这个张量的含义是64个格点分别对于倒格矢k的贡献（插值出来的单个点电荷的作用效果）。那么类似的可以得到：


s(k)\=VF∗k(Q)(mx,my,mz)Fk(Q)(mx,my,mz)\=V\|Fk(Q)(mx,my,mz)\|2代入到Ewald形式的长程相互作用项（可以参考这篇[文章](https://github.com)）中可以得到：


EL\=12kxkykzϵ0∑\|k\|\>0e−σ2k22k2s(k)\=V2kxkykzϵ0∑\|k\|\>0e−σ2k22k2\|Fk(Q)(mx,my,mz)\|2这就是Particle\-Mesh\-Ewald方法计算中计算长程相互作用势能的技巧。既然k空间无法快速收敛，那就减少电荷分布项的计算复杂度，同样也可以起到大量节约计算量的效果。


# 短程相互作用项的截断


在前面Ewald求和的文章中我们介绍过，把静电势能的计算分成长程、短程和自我相互作用项之后，分别有不同的收敛形式。长程相互作用项已经通过上述章节完成了计算量的简化，另外还有一个短程相互作用项ES，我们知道短程相互作用项关于原子实空间的间距是快速收敛的，并且在计算LJ势能的时候我们已经计算过一次给定cutoff截断的近邻表。那么，我们很容易考虑到引入近邻表的概念，直接利用这个近邻表对静电势能的短程相互作用项做一个截断。于是短程相互作用项可以写为：


ES\=∑nN−2∑i\=0N−1∑j\=i\+1qiqj4πϵ0\|rj−ri\+nL\|Erfc(\|rj−ri\+nL\|√2σ)\+∑\|n\|\>0q2i4πϵ0\|nL\|Erfc(\|nL\|√2σ)≈∑i,j∈{Neigh}qiqj4πϵ0\|rj−ri\|Erfc(\|rj−ri\|√2σ)这里有个前提假设是dcutoff\<\<Lpbc，所以略去了周期性盒子中其他盒子内的i电荷对中心盒子的ri处的作用项。


# Particle\-Mesh\-Ewald


根据上面章节中得到的近似的远程相互作用项和短程相互作用项之后，我们可以重写PME(Particle\-Mesh\-Ewald)算法中的总静电势能为：


E\=ES\+EL−Eself\=∑i,j∈{Neigh}qiqj4πϵ0\|rj−ri\|Erfc(\|rj−ri\|√2σ)\+V2kxkykzϵ0∑\|k\|\>0e−σ2k22k2\|Fk(Q)(mx,my,mz)\|2−14πϵ01√2πσN−1∑i\=0q2i# 总结概要


本文介绍了使用基于格点拉格朗日插值法的Particle Mesh Ewald算法，降低分子力场中的静电势能项计算复杂度的基本原理。静电势能的计算在Ewald求和框架下被拆分成了远程相互作用项和短程相互作用项，其中短程相互作用项关于实空间的点电荷间距快速收敛，而远程相互作用项在倒易空间慢速收敛。因此在远程相互作用的计算中，可以使用插值法降低单个倒易格点的计算复杂度，从而使得整体的远程相互作用项计算也能够快速收敛。


# 版权声明


本文首发链接为：[https://github.com/dechinphy/p/pme.html](https://github.com):[楚门加速器](https://shexiangshi.org)


作者ID：DechinPhy


更多原著文章：[https://github.com/dechinphy/](https://github.com)


请博主喝咖啡：[https://github.com/dechinphy/gallery/image/379634\.html](https://github.com)


# 参考链接


1. [https://bohrium.dp.tech/notebooks/62979247598](https://github.com)


