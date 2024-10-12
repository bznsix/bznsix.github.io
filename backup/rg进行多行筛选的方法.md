有时候我们不希望仅搜索一个条件，我们希望多个条件结合起来搜索，那么我们就可以巧妙的利用-A选项追加显示，再结合一个rg来进行多重搜索，如下所示：
`rg 'fromBalance = _balances\[from\];' -A 5  | rg 'toBalance = _balances\[to\];'`
-A代表的显示搜查结果及其下五行，之后传给另一个rg。聪明的人已经可以很清楚的知道我在搜什么漏洞了。搜索结果如图所示：
![image](https://github.com/user-attachments/assets/aa106c50-b72c-4c3a-b655-e5a8856eb1ab)

升级小技巧：如何搜索包含某个字符串 但是后面几行不包含某个字符串的呢：
```
rg -F "function buy()" -A 10 | awk '
    BEGIN {found=0; buffer=""; count=0}
    /function buy\(\)/ {found=1}
    found {
        buffer = buffer $0 "\n"
        if ($0 ~ /tx\.origin/) {found=0; buffer=""; next}
        count++
        if (count > 10) {
            if (buffer != "") print buffer
            found=0; buffer=""; count=0
        }
    }
    /^$/ && found {
        if (buffer != "") print buffer
        found=0; buffer=""; count=0
    }
'
```
