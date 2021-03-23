---
title: leetcode-roadmap
date: 2021-03-22 20:15:54
tags: LeetCode
---

推荐站点：

https://qoogle.top/how-to-brush-leetcode/

https://codetop.cc/ 

# https://codetop.cc/ 微软

227. 基本计算器 II https://leetcode-cn.com/problems/basic-calculator-ii/

2012/03/22-23

这道题，用栈是最快速的，可以现场推理思路，完成80%应该不是问题。
但这种表达式，很可能会被考变种，比如加了括号，加了UnaryOp，甚至更难。
我目前知道的万能方法就是，编译器的基本思路，也就是词法-解析-翻译，这种流程。

-[ ] 待补充

总结来说，基本思路需要Lexer，Interpreter。如果token过于复杂，最好加个Parser，Parser解析出AST，Interpreter再遍历AST树，计算结果。

面试时使用这个思路，写代码会慢一点，但是感觉深度比栈思路要深。

