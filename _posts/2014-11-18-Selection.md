---
layout: blog
title: 实用优化算法（六－1）Selection in Genetic Algorithm
categories: Algorithm
tags: algorithm
---
遗传算法包括三个基本操作**选择、交叉、变异**，这一篇说说选择（Selection）

##轮盘赌选择（Roulette Wheel Selection）

`public class RouletteWheelSelection implements ISelectionAlgorithm { // start
  /** the temp */
  private double[] temp;
  /** the roulette wheel selection */
  public RouletteWheelSelection() {
    super();
  }
  /** {@inheritDoc} */
  @Override
  // end
  public void select(final Individual<?, ?>[] pop, final Individual<?, ?>[] mate, final Random r){
  double[] t;
    double max, last;
    int i, j;
    //init the length of t and temp
    t = this.temp;
    if ((t == null) || (t.length < pop.length)) {
      this.temp = t = new double[pop.length];
    }
    max = Double.NEGATIVE_INFINITY;
    //find the max in individual
    for (Individual<?, ?> indi : pop) {
      max = Math.max(indi.v, max);
    }
    max = Math.nextUp(max);
    last = 0d;
    for (i = 0; i < t.length; i++) {
      last += (max - pop[i].v);
      t[i] = last;
    }
    t[t.length - 1] = Double.POSITIVE_INFINITY;
    for (i = 0; i < mate.length; i++) {
      j = Arrays.binarySearch(t, last * r.nextDouble());
      if (j < 0) {
        j = ((-j) - 1);
      }
      mate[i] = pop[j];
    }
  }
}`

##锦标赛选择（Tournament Selection）
`
`

##截断选择（Truncation Selection）
`public class TruncationSelection implements ISelectionAlgorithm { // start
  /** the globally shared instance */
  public static final TruncationSelection INSTANCE = new TruncationSelection();
  /** the truncation selection */
  private TruncationSelection() {
    super();
  }
  @Override
  public void select(final Individual<?, ?>[] pop, final Individual<?, ?>[] mate, final Random r) {
	  Individual<?,?> swap;
	  //根据目标函数f的值作为强度，使用冒泡排序法对人口进行排序
	  for(int i = 0; i < pop.length; i++){
		  for (int j = 1; j < pop.length - i + 1; j++) {
			  if(pop[j-1].v < pop[j].v){
				  swap = pop[j-1];
				  pop[j - 1] = pop[j];
				  pop[j] = swap;
			  }
		  }
	  }
	  //将排在前面的人口纳入交配池
	  for(int i = 0; i < mate.length; i++){
		  mate[i] = pop[i];
	  }
  }
}`
