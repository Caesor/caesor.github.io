---
layout: blog
title: 实用优化算法（六－2）Crossover & Muation in Genetic Algorithm
categories: Algorithm
tags: algorithm
---
遗传算法包括三个基本操作**选择、交叉、变异**，这一篇说说交叉（Crossover）
交叉有单点交叉（Single－Point）、两点交叉（Two－Point）、多点交叉（Multi－Point）和组合交叉（Uniform）
![picture1]({{site.blogimgurl}}/2014-11-20-01.png "example_pic")

##单点交叉（Single－Point）
`public class BitsBinarySPX implements IBinarySearchOperation<boolean[]> {// start
  /** instantiate */
  public BitsBinarySPX() {
    super();
  }
  @Override
  public boolean[] recombine(final boolean[] p1, final boolean[] p2, final Random r) {
	  int n = p1.length;
	  int crossPoint = r.nextInt(n);
	  final boolean[] p;
	  p = p1.clone();
    //交换某个点后的所有基因
	  for(int i = crossPoint; i < n; i++){
		  p[i] = p2[i];
	  }
	  return p;
  }
}`

##组合交叉（Uniform）
`public class BitsBinaryUX implements IBinarySearchOperation<boolean[]> {
  public BitsBinaryUX() {
    super();
  }
  @Override
  public boolean[] recombine(final boolean[] p1, final boolean[] p2, final Random r) {
	  int n = p1.length;
	  final boolean[] p;
	  p = p1.clone();
    //随机交换每一对基因
	  for(int i = 0; i < n; i++){
		  if(Math.random() > 0.5){
			  p[i] = p2[i];
		  }
	  }
	  return p;
  }
}`