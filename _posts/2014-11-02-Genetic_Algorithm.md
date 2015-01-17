---
layout: blog
title: 实用优化算法（六）Genetic Algorithm
categories: Algorithm
tags: algorithm
---
##前言
**达尔文的“万物起源”学说有以下这些思想：**

1、在没有巨大变故的影响下，物种的总数几乎保持不变；

2、如果没有巨大的变故，食物资源会紧缺但是稳定；

3、个体都会为了紧缺的资源去竞争；

4、一些个体之间的变化将会影响到他们的身体素质，能力得以提升；

5、这些个体的变化是可以被后代所继承的，身体素质最强的个体将会生存下去并且能够繁衍后代；

6、因此一个物种将会缓慢的变化并且去适应周围的新环境。

**上面的一些思想引起了我们的思考：**

1、在一个人口基数内，存在不止一个某物种的个体；

2、父代后面肯定会有一个子代；

3、身体素质最强的个体在生存选择中活下去的可能性要更大一点；

4、活下去的个体又会成为新生代的父代。

##遗传算法（Genetic Algorithm）
遗传算法又叫基因进化算法，属于共同启发式搜索算法的一种。搜索空间G为给定长度的位流（2进制流）

它会经历下面几个过程：

1、第一代（t=1），从人口 **pop** 中挑选出 **ps** 个个体 **p**

2、从基因型 **p.g** 映射转换成表现型 **p.x**

3、计算人口 **pop** 中每个个体的适应值 **f(p.x)**

4、开始选择（Selection），使用选择算法选取 **mps** 个个体放入交配池 **mate** 中

5、根据交叉率 **cr** 使用 交叉（Crossover） 或 变异（Mutation）算法产生新一代个体

6、在完成一次函数评估（适应性检测）后，检查终止条件决定是否停止迭代

![picture1]({{site.blogimgurl}}/2014-11-02-01.png "EA")

	public EA() {
	    super();
	    //变异系数
	    this.cr        = 0.56;
	    //种族的总个体数
	    this.ps        = 382;
	    //交配池能容纳的个体
	    this.mps       = 80;
	    //选取截断算法选择个体进入交配池
	    this.selection = TruncationSelection.INSTANCE;;
	  }
	  @SuppressWarnings("unchecked")
	  @Override
	  public Individual<G, X> solve(final IObjectiveFunction<X> f) {
		Individual<G, X>[] pop, mate;
		Individual<G, X> pbest, pcur, parent;

		//pop中存放种族中所有个体
		pop = new Individual[this.ps];
		//mate中存放即将要交配的个体
		mate = new Individual[this.mps];

		parent = new Individual<>();
		pbest = new Individual<>();
		pbest.v = Double.POSITIVE_INFINITY;

		//生成种族群体
		for(int i = 0; i < pop.length; i++){
			pop[i] = pcur = new Individual<>();
			//随机生成基因，基因为二进制流
			pcur.g = this.nullary.create(this.random);
		}
		for(;;){
			//适应性检测
			for(int j = 0; j < pop.length; j++){
				pcur = pop[j];
				pcur.x = this.gpm.gpm(pcur.g);
				//计算每个个体的适应值，选择当前一代群体中适应性最强的个体
				pcur.v = f.compute(pcur.x);
				if(pcur.v < best.v){
					best.assign(pcur);  
				}
				if(this.termination.shouldTerminate())
					return best;
			}

			//通过选择算法选取个体进入交配池
			this.selection.select(pop, mate, this.random);

			//交配并产生下一代，直到新一代种族个体的数目达到原有水平
			for(int k = 0; k < pop.length; k++){
				pop[k] = pcur = new Individual<>();
				//从交配池中选择一个个体作为父代
				parent = mate[k % mate.length];
				//概率小于cr时，产生一个交配后代
				if(this.random.nextDouble() < this.cr){
					//交叉式交配
					pcur.g = this.binary.recombine(parent.g, mate[this.random.nextInt(mate.length)].g, this.random);
				}else{//概率大于cr时，个体发生变异
					pcur.g = this.unary.mutate(parent.g, this.random);
				}
			}
		}
	  }
	}

我们将在后面的具体介绍**交叉式交配**和**变异**的具体算法。







