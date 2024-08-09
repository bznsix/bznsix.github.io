有时候我们不希望仅搜索一个条件，我们希望多个条件结合起来搜索，那么我们就可以巧妙的利用-A选项追加显示，再结合一个rg来进行多重搜索，如下所示：
`rg 'fromBalance = _balances\[from\];' -A 5  | rg 'toBalance = _balances\[to\];'`
-A代表的显示搜查结果及其下五行，之后传给另一个rg。聪明的人已经可以很清楚的知道我在搜什么漏洞了。搜索结果如图所示：
![image](https://github.com/user-attachments/assets/aa106c50-b72c-4c3a-b655-e5a8856eb1ab)
