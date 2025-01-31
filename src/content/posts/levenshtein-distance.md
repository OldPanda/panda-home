---
title: 如何实现 git 命令行的联想功能
pubDate: 2020-05-22 16:06 PST
categories: ["聊聊技术"]
tags: Python, cli, Git, Levenshtein distance, 莱文斯坦距离, 算法
heroImage: /images/blog/git-ui-scaled.jpg
heroImageDescription: Photo by Yancy Min on Unsplash
---

## 问题背景

码农生涯离不开 git ，无论是编码开发，版本控制，还是持续集成，代码审查， git 无疑是有效跟踪项目进展的利器，而 git 命令行更是必不可少的工具。我之前也尝试过一些带界面的 git 工具，然而都没有命令行来的顺手，按钮太多，界面太复杂，反而容易搞不清楚一些简单的操作应该如何下手。

命令行用的多了，难免有手滑输错命令的时候，这个时候屏幕上就会提示

```shell
🐼 ~ » git stats
git: 'stats' is not a git command. See 'git --help'.
The most similar command is
	status
```

当年第一次用 git 的时候还是小惊喜了一下，因为之前接触的其他工具不是直接提示命令找不到，就是直接输出一大段 help 界面，对用户很不友好，所以 git 能直接把最接近的子命令输出到屏幕上还是很方便的。

## 算法简介

那么这个贴心的小功能是如何实现的呢？很自然地想到，要想办法定义两个字符串之间的“相似程度”，换句话说，要找到一个度量衡，来量化两个字符串之间的不同程度为多大，然后遍历一遍 git 的子命令，找出不同程度最小的那个返回即可。这个度量衡早在 1965 年前苏联数学家 [Vladimir Levenshtein](https://en.wikipedia.org/wiki/Vladimir_Levenshtein) 就已经给出了定义，因此命名为 [Levenshtein Distance](https://en.wikipedia.org/wiki/Levenshtein_distance) （莱文斯坦距离）。内容很简单，相似程度可以用两个字符串的**编辑距离**来量化，即给出字符串甲和字符串乙，只通过**添加**一位字母，**删除**一位字母，或者**替换**一位字母的操作，最少经过多少次操作能把字符串甲变成字符串乙。

举个例子，字符串 `kitten` 和 `sitten` 的编辑距离是多少？显然是 1 ，因为把首位字母**替换**为 s 即可成为后者。再来一个，字符串 `abd` 和 `abcd` 的编辑距离也是 1 ，因为只需要**添加**一个字母 c 就得到了后者。但是字符串有很多，复杂的情况无穷无尽，比如说 `September` 和 `October` 的编辑距离就不容易一眼看出来，所以需要一个算法来帮我们计算这个结果。

先考虑最简单的情况，当一个字符串为空的时候，编辑距离自然是另一个字符串的长度，因为只需要通过简单的添加操作，就能把一个空字符串变成另一个字符串，或者通过简单的删除操作，就能把一个非空字符串变成空字符串。

然后是比较复杂的情况，当两个字符串都不为空的时候，应该如何得到它们的编辑距离？我们先假设两个字符串的子串的编辑距离已知，这样我们就可以很容易地得出这两个字符串的编辑距离。具体说来，假如两个字符串分别命名为 `str1` 和 `str2` ，已知下列子串的编辑距离（为说明方便，我这里的子串用 Python 的语法来表示）

- `str1[:-1]` -> `str2[:-1]`

- `str1[:-1]` -> `str2`

- `str1` -> `str2[:-1]`

已知第一种情况下的编辑距离，我们只需要比较 `str1` 和 `str2` 的最后一位字母，如果相同的话， `str1` 和 `str2` 的编辑距离其实与 `str1[:-1]` 和 `str2[:-1]` 的编辑距离是一样的，因为我们不需要对最后一位字母做任何操作。如果最后一位字母不相同的话，我们只需要在第一种情况的基础上做一次**替换**，就能得到 `str2` 了，所以这时 str1 和 str2 的编辑距离为 `str1[:-1]` 和 `str2[:-1]` 的编辑距离加一。

已知 `str1[:-1]` -> `str2` 的编辑距离， `str1` 到 `str1[:-1]` 的距离显然是 1 ，只要**删除**最后一位字母就行了，所以 str1 到 str2 的编辑距离为 `str1[:-1]` -> `str2` 加一，具体操作为 `str1` -> `str1[:-1]` -> `str2` 。数学家嘛，总是喜欢把一个未知的问题转化成一个已知的问题来解决。

同样地，已知 `str1` -> `str2[:-1]` 的距离，只需要给 `str2[:-1]` **添加**最后一位字母，就得到了 `str1` -> `str2` 的编辑距离。

最后，从上述三种情况得到的编辑距离中取最小，就是我们想要的答案。

但问题还没完，上述三种情况的编辑距离我们是“假设”它们已知了，而实际情况中我们是无法直接得到它们的编辑距离的。考虑一下一开始提过的“最简单的情况”，基于它们，一步步向复杂的情况推进，我们很容易地就得到了所有情况的编辑距离。

于是，这个算法就有了形式化的定义。记字符串 a 与 b 的编辑距离为 \[katex display=true\]lev\_{a,b}(|a|, |b|)\[/katex\]其中 |a| 和 |b| 分别表示 a 与 b 的长度，可以得到推导公式

$$
lev_{a,b}(i, j)=\begin{cases}
    \begin{array}{ll}
        max(i, j) & if min(i, j)=0, \\
        min
        \begin{cases}
            \begin{array}{ll}
                lev_{a,b}(i-1,j) + 1 \\
                lev_{a,b}(i,j-1) + 1 \\
                lev_{a,b}(i-1,j-1) + 1_{a_i \ne b_j}
            \end{array}
        \end{cases} & otherwise.
    \end{array}
\end{cases}
$$

这个公式是用递归的形式定义的，但工程实现的时候我一般会倾向于避免这种方式，防止在输入过长的时候爆栈，这里可以用矩阵来存储 `lev(i,j)` 的值。以[维基百科](https://en.wikipedia.org/wiki/Levenshtein_distance)上的 `sitting` 和 `kitten` 的编辑距离为例，可以得到下面一个矩阵，右下角的 3 即为它们的编辑距离。

<table class="wikitable"><tbody><tr><td></td><td></td><th>k</th><th>i</th><th>t</th><th>t</th><th>e</th><th>n</th></tr><tr><td></td><td>0</td><td>1</td><td>2</td><td>3</td><td>4</td><td>5</td><td>6</td></tr><tr><th>s</th><td>1</td><td><span title="substitution of 'k' for 's'" class="rt-commentedText" style="border-bottom:1px dotted">1</span></td><td>2</td><td>3</td><td>4</td><td>5</td><td>6</td></tr><tr><th>i</th><td>2</td><td>2</td><td><span title="'i' equals 'i'" class="rt-commentedText" style="border-bottom:1px dotted">1</span></td><td>2</td><td>3</td><td>4</td><td>5</td></tr><tr><th>t</th><td>3</td><td>3</td><td>2</td><td><span title="'t' equals 't'" class="rt-commentedText" style="border-bottom:1px dotted">1</span></td><td>2</td><td>3</td><td>4</td></tr><tr><th>t</th><td>4</td><td>4</td><td>3</td><td>2</td><td><span title="'t' equals 't'" class="rt-commentedText" style="border-bottom:1px dotted">1</span></td><td>2</td><td>3</td></tr><tr><th>i</th><td>5</td><td>5</td><td>4</td><td>3</td><td>2</td><td><span title="substitution of 'e' for 'i'" class="rt-commentedText" style="border-bottom:1px dotted">2</span></td><td>3</td></tr><tr><th>n</th><td>6</td><td>6</td><td>5</td><td>4</td><td>3</td><td>3</td><td><span title="'n' equals 'n'" class="rt-commentedText" style="border-bottom:1px dotted">2</span></td></tr><tr><th>g</th><td>7</td><td>7</td><td>6</td><td>5</td><td>4</td><td>4</td><td><span title="delete'g'" class="rt-commentedText" style="border-bottom:1px dotted">3</span></td></tr></tbody></table>

Python 编码实现如下，

```python
def levenshtein_distance(str1, str2):
    rows = len(str1)
    cols = len(str2)
    lev = [[0 for _ in range(cols+1)] for _ in range(rows+1)]
    for i in range(rows+1):
        lev[i][0] = i
    for j in range(cols+1):
        lev[0][j] = j
    for i in range(1, rows+1):
        for j in range(1, cols+1):
            lev[i][j] = min(
                lev[i-1][j] + 1,
                lev[i][j-1] + 1,
                lev[i-1][j-1] + (0 if str1[i-1] == str2[j-1] else 1)
            )
    return lev[rows][cols]
```

这个算法的时间复杂度和空间复杂度都为 O(M\*N) ， M 和 N 分别代表两个字符串的长度。

回到 git 命令行， git 的子命令不过只有二十几个，最长的也不过 8 个字母，所以当遇到无法识别的命令时，一一计算输入的子命令和已有的子命令之间的编辑距离，就能快速找到最相近的一个并返回到屏幕上。当然了， git 也不会胡乱推荐，如果输入的子命令太过于奇葩，令编辑距离过大， git 是什么话也不会说的。

```shell
🐼 ~ » git abcdefg
git: 'abcdefg' is not a git command. See 'git --help'.
```

## 2022/02/19 更新

最近在[学习了解 Go 语言的命令行工具生成器 Cobra](https://old-panda.com/posts/write-a-cli-using-golang-cobra/) ，简单看了下它的源码，发现也有子命令联想功能，实现的就是字符串之间的[莱温斯坦距离](https://github.com/spf13/cobra/blob/v1.3.0/cobra.go#L166)，而在一众子命令中寻找最为相似的[逻辑](https://github.com/spf13/cobra/blob/v1.3.0/command.go#L727)，与本文写作时推测的有些不同， Cobra 会设定一个距离的阈值，将莱温斯坦距离小于等于这个阈值的子命令加入数组等待返回，同时考虑了用户的输入是否为某个有效子命令的前缀。

## 参考资料

- [https://en.wikipedia.org/wiki/Levenshtein\_distance](https://en.wikipedia.org/wiki/Levenshtein_distance)

- [https://leetcode-cn.com/problems/edit-distance/](https://leetcode-cn.com/problems/edit-distance/)

- [Cobra v1.3.0 源码](https://github.com/spf13/cobra/tree/v1.3.0)
