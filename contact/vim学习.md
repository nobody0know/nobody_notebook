# Vim学习

## 好用的操作符加动作符

### 1.删除整个括号内所有字符，不包括括号并进入编辑模式

```vim
ci(        //change inner （
```

或

```vim
cib        //b是括号的缩写
```

### 2.复制整个括号内所有字符然后使用p复制

```vim
yi(
```

### 3.修改到“，”复制到

```vim
cfxxx
```

### 4.快速跳转到定义

```vim
gd
```

### 5.看函数签名

```vim
gh
```

### 6.快速切换标签页

```vim
alt n
gt、gnt(n为要跳的标签页数) 
```

### 7.快速切换到某(n)一行

```vim
n 回车
ngg （n为要跳到的行数）
```

### 8.从系统剪切板粘贴到vim

```vim
“+p （全部粘贴）
```

### 9.删除一个单词

```vim
daw
```

### 10.替换

```vim
1. 基本的替换

:s/vivian/sky/ 替换当前行第一个 vivian 为 sky

:s/vivian/sky/g 替换当前行所有 vivian 为 sky

:n,$s/vivian/sky/ 替换第 n 行开始到最后一行中每一行的第一个 vivian 为 sky

:n,$s/vivian/sky/g 替换第 n 行开始到最后一行中每一行所有 vivian 为 sky

（n 为数字，若 n 为 .，表示从当前行开始到最后一行）

:%s/vivian/sky/（等同于 :g/vivian/s//sky/）替换每一行的第一个 vivian 为 sky

:%s/vivian/sky/g（等同于 :g/vivian/s//sky/g） 替换每一行中所有 vivian 为 sky

2. 可以使用 # 作为分隔符，此时中间出现的 / 不会作为分隔符

:s#vivian/#sky/# 替换当前行第一个 vivian/ 为 sky/

:%s+/oradata/apras/+/user01/apras1+ （使用+ 来替换 / ）： /oradata/apras/替换成/user01/apras1/
```
