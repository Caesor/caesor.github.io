---
layout: blog
title : 实用优化算法（九）Hill Climbing vs Greedy Construction
categories : algorithm
tags : algorithm
---

##贪心算法（Greedy Construction）

贪心算法又称贪婪算法，是指在对问题求解时，总是做出对当时看来是最好的选择。也就是说，不从整体最优上加以考虑，他所作出的仅是在某种意义上的局部最优解。贪心算法不是对所有问题都能得到整体最优解，但对范围相当广泛的许多问题他能产生整体最优解或者是整体最优解的近似解。

Does it sounds familiar? Yep!好像登山算法也是这样说的，今天这节我们将说说登山算法和贪心算法的区别。

##贪心算法 vs 登山算法
<table>
	<thead>
		<tr>
			<td>Greedy Construction</td>
			<td>Hill Climbing</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>从一个空值的解决方案开始 e.g. x=(1,?,?,?,?)</td>
			<td>从一个完全假设的候选结果开始 e.g. x=(1,2,3,4,5)</td>
		</tr>
		<tr>
			<td>一步步去构造一个完整的结果</td>
			<td>不断修改候选结果并修改候选结果</td>
		</tr>
		<tr>
			<td>每一步确定一个结果中的要素 e.g x=(1,?,?,?,?) -> x=(1,2,?,?,?) -> x=(1,2,3,?,?) -> x=(1,2,3,4,?) -> x=(1,2,3,4,5)</td>
			<td>每一步评估新的候选结果是否优秀 e.g x=(1,2,3,4,5) -> x=(1,4,3,5,2)</td>
		</tr>
		<tr>
			<td>总是根据启发性函数选择一个新元素</td>
			<td>保留评估值优秀的结果</td>
		</tr>
		<tr>
			<td>当候选结果构建完成后结束</td>
			<td>等确定的终止条件到达时结束 e.g. 1000000步之后</td>
		</tr>
	</tbody>
</table>

##总结
贪婪算法结构只会构建一个候选结果，而登山算法会创建多个候选结果最后保留一个最好的结果。贪婪算法的构建过程可以用作登山算法的 Nullary 搜索操作。




