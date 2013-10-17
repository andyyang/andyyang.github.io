---
layout: post
title: "每周编程题：线程协同打印数字序列"
date: 2013-10-17 00:55
comments: true
categories: 
---
## 题目
Write a program to print a number sequence from 1 to m with n threads. Each thread print one number at a time. The threads run in a round robin fashion.

Input: n = 10, m = 100

sample output:  
Thread-1: 1  
Thread-2: 2  
....  
....  
Thread-10: 10  
Thread-1: 11  
Thread-2: 12  
....  
....  
Thread-10: 20  
Thread-1: 21  
Thread-2: 22  
...  
...  
Thread-10: 100

## 分析
这里需要多个线程轮流打印一个数字序列，因此这些线程需要协同工作，即线程1取数字k打印后暂停打印，线程2取下一个数字k+1继续打印，以此类推，线程10取数字k+9打印后，线程1又开始取数字k+10打印，直到序列中的所有数字都打印完。

本来准备在每个工作线程开始后在工作线程内部调用Thread.stop来暂停工作线程，然后在主线程中通过Thread#run唤醒第一个工作线程，并在之后通过Thread#join依次等待所有工作线程结束。但是由于Ruby线程模型的限制，这样的调用会产生错误信息"in `join': No live threads left. Deadlock?"，即Thread＃join导致主线程被挂起，同时调度运行其他的工作线程，如果此时所有的工作线程由于Thread.stop的调用而处于“睡眠”状态时，就会出现没有线程可以调度运行即死锁的问题。

解决办法是通过一个所有线程共享的变量来指示当前应该是哪个工作线程来运行，如果当前线程不是应该运行的工作线程，则该线程调用Thread.pass把执行权让给其他线程；如果当前线程是应该运行的工作线程，则在打印数字后将指示变量设置为下一个线程的序号。由于任何时候只有一个线程会设置该指示变量，因此不存在变量同步访问的问题。

## 代码
{% codeblock lang:ruby %}
if ARGV.size != 2
  puts "usage: threads n m"
  exit
end

n = ARGV[0].to_i
m = ARGV[1].to_i

if n <= 0 || m <= 0
  puts "n and m must be lager than 0"
  exit
end

threads = []
current_thread = 1

(1..n).each do |i|
  threads << Thread.new do
    count = i
    while count <= m
      if i != current_thread
        Thread.pass
        redo
      end
      puts "Thread#{i}: #{count}"
      count += n
      current_thread = (i + 1) <= n ? (i + 1) : 1
    end
  end
end

threads.each do |t|
  t.join
end
{% endcodeblock %}

