---
layout: blog
title: 实用优化算法（六－2）Crossover & Muation in Genetic Algorithm
categories: Algorithm
tags: algorithm
---
遗传算法包括三个基本操作**选择、交叉、变异**，这一篇说说交叉（Crossover）

交叉有单点交叉（Single－Point）、两点交叉（Two－Point）、多点交叉（Multi－Point）和组合交叉（Uniform）4种方式

![picture1]({{site.blogimgurl}}/2014-11-20-01.png "example_pic")

##单点交叉（Single－Point）

	public class BitsBinarySPX implements IBinarySearchOperation<boolean[]> {// start
	  /** instantiate */
	  public BitsBinarySPX() {
	    super();
	  }
	  @Override
	  public boolean[] recombine(final boolean[] p1, final boolean[] p2, final Random r) {
		  final boolean[] g;
		  final int x;
		  //初始化空位流
		  g = new boolean[p1.length];
		  //选择交叉点
		  x = (1 + r.nextInt(g.length - 1));
		  //复制 p1 中前半部分到 g
		  System.arraycopy(p1, 0, g, 0, x);
		  //复制 p2 中后半部分到 g
		  System.arraycopy(p2, x, g, x, g.length - x);

		  return g;
	  }
	}

##组合交叉（Uniform）

	public class BitsBinaryUX implements IBinarySearchOperation<boolean[]> {
	  public BitsBinaryUX() {
	    super();
	  }
	  @Override
	  public boolean[] recombine(final boolean[] p1, final boolean[] p2, final Random r) {
	  	final boolean[] g;
	  	g = p1.clone();
	  	for(int i = 0; i < g.length; i++){
	  	  //每个基因都有有一半的概率来自某一个父代
	  	  if(r.nextBoolean()){
	  		g[i] = p2[i];
	  	  }
	  	}
	  	return g;
	  }
	}