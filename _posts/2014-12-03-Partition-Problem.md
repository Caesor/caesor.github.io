---
layout: blog
title: 实用优化算法（八）分割问题（Partitioning Problem）
categories : algorithm
tags : algorithm
---

##问题的定义

设想有5个包裹，5个包裹完成分装的人工时间分别是 ｛5，7，3，8，1｝，如何分工将会是公平的。

分割问题被定义为一个实数组 V = (v1,v2,...,vn)，目标是将 V 分割为两个数组，原来数组的每个元素只会在两个数组中的一个中。

要求是：两个数组中实数的和应该是相等的，或者至少是尽可能相近的。

例如： V = （5，7，3，8，1） => V1=(5,7) 和 v2(1,3,8)

**问题空间：** X = （5，7，3，8，1）

**目标函数：** Minimize f(x) = abs( Σ(x[i] in v1) - Σ(y[i] in v2) )

我们将问题空间转化为**搜索空间 G = (0,1,0,1,0)**，可以用 "0" 表示将该元素放入数组 v1, 用 "1" 表示将该元素放入数组 v2

##搜索操作（Search Operation）

**Nullary:** 产生一个随机的二进制流数组

	public class BitsNullaryUniform implements INullarySearchOperation<boolean[]> {
	  public BitsNullaryUniform(final int m) {
	    super();
	    this.n = m;
	  }
	  @Override
	  public boolean[] create(final Random r) {
	    final boolean[] g;
	    int i;

	    g = new boolean[this.n];         // allocate integer array
	    for (i = this.n; (--i) >= 0;) { // initialize each element to its index
	      //随机构建二进制数组
	      if (r.nextBoolean()) {
	        g[i] = true;
	      }
	    }
	    return g;
	  }
	}

**Unary:** 对候选对象以一定概率进行变异

	public class BitsUnaryFlip implements IUnarySearchOperation<boolean[]> {// start
	  /** instantiate */
	  public BitsUnaryFlip() {
	    super();
	  }
	  @Override
	  public boolean[] mutate(final boolean[] p, final Random r) {
	    final boolean[] g;
	    
	    g = p.clone();                    // copy parent string
	    do {                              // at least once, but maybe more often
	      g[r.nextInt(g.length)] ^= true; // 对该位取反
	    } while (r.nextBoolean());        // 可能会发生多次变异（但概率比较小）
	    return g;                         // 返回新的位流
	  }
	}

##使用模拟退火算法（SA）解决分割问题

	public class SAOnPartition {// start
	  public static void main(final String[] args) {
	    final SA<boolean[], boolean[]> sa      = new SA<>();
	    //初始化导入真实数据
	    final PartitionProblem         problem = new PartitionProblem("instance_2.txt");
	    
	    //定义 Partition Problem 的 Search Operation，这里使用的是 二进制数组
	    sa.nullary = new BitsNullaryUniform(problem.getSetSize());//定义初始化二进制流方法
	    sa.unary   = new BitsUnaryFlip();//定义变异方法
	    //定义此次SA所使用的温度变化函数，这里选择的是 温度 指数型降低
	    sa.temperature = new Exponential(10000d, 0.025d);

	    for(int i = 1; i <= 100; i++)  {
	      sa.termination = new MaxSteps(500000);
	      System.out.println(sa.solve(problem).v);
	    }
	  }
	}

