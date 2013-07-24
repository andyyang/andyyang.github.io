---
layout: post
title: "面试编程题：一个都不能少"
date: 2013-07-22 01:42
comments: true
categories: 
---
## [原题][weibo]
有k个有序的数组，请找到一个最小的数字范围，使得这k个有序数组中，每个数组都至少有一个数字在该范围中。 例如：

*	数组1：4, 10, 15, 24, 26
*	数组2：0,  9, 12, 20
*	数组3：5, 18, 22, 30

则所得最小范围为[20,24]，其中，20在数组2中，22在数组3中，24在数组1中。

## 代码
{% codeblock lang:ruby %}
	def find_min_range(arrays)
	  k = arrays.length
	  min_range = [0,-1]
	  range = Array.new(k) {|i| {value:arrays[i][0], index:0, pos:i} }
	
	  while true
	    range.sort_by! {|elem| elem[:value] }
	    if range[k-1][:value] - range[0][:value] < min_range[1] - min_range[0] ||
	       min_range[1] - min_range[0] < 0
	      min_range = [range[0][:value], range[k-1][:value]]
	    end
	    min = range.shift
	    index = min[:index]
	    pos = min[:pos]
	    break if index >= arrays[pos].length - 1
	    index += 1
	    range.push({value:arrays[pos][index], index:index, pos:pos})
	  end
	  puts min_range
	end

	find_min_range([[4,10,15,24,26], [0,9,12,20], [5,18,22,30]])
{% endcodeblock %}
[weibo]: http://t.cn/zQGvysV 
