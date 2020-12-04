+++
title = "Latex - IEEEtran模板踩坑日志"
summary = "最近有篇paper需要投稿到TCSVT，所以前几天下载了IEEEtran模板把之前的word版转换成latex版。因为编辑过程中遇到不少困难，故特地记录下来，方便自己以后查阅。"
authors = ["duyang"]
 tags = ["科研","Latex"]
categories = ["技术"]
series = ["Latex"]
date = "2019-03-15"
lastmod = "2019-03-15"
draft = false
+++

## "section{}"中单词大小写问题

在IEEEtran这个模板中，section{}部分中的内容默认全部格式化成大写，并且每个单词首字母字号更大一些，排查了一下，这个效果是与下面这段代码有关的，如果不想要大写，这段代码可以注释掉。

```latex
\hyphenation{op-tical net-works semi-conduc-tor}
```

## 图例标注位置问题

图例标注默认是左对齐的，我参考了TCSVT中的几篇文章，发现三篇文章对图例的样式设置都不一样

。一般来说当然是居中好看一些，由于不知道怎么直接修改位置，于是引用了"caption"包，可以设定图例的格式以及位置，比如可以设置成"Figure."也可以设置成"Fig."。但是发现修改之后表格的标注会自动修改。原本表格的caption内容是全部大写的，跟section{}中的内容一样，使用了caption包之后就会失去全部大写的效果。我目前放弃使用了caption包，等到下次编辑时看是否能找到好的解决方案。

## 花括号下多行公式左对齐解决办法

```latex
\begin{equation}\nonumber
\centering
\left\{
\begin{array}{lll}
N\leqslant N_u & \\
\sum_{i=1}^{N}p_i\leqslant N_n  & \\
p_i=2^{j}-1,i\in 1,2,\cdots,N,j\in N^* &
\end{array}
\right.
\end{equation}
```

![花括号下多行公式左对齐](http://poex2tesj.bkt.clouddn.com/Snipaste_2019-03-15_22-50-56.png)

## 通栏公式(跨双栏)

当论文中公式过长的时候，一栏很难展示的很好，也许可以通过\\!调整位置或者通过\begin{small}来缩小公式字体，但过长的公式无论如何微调表现都不尽人意，这时候就需要利用通栏公式了。下图就是我想要得到的通栏公式的效果：

![通栏公式](http://poex2tesj.bkt.clouddn.com/Snipaste_2019-03-15_22-55-48.png)

为此，查阅了不少博客，最终找到了一种有效的处理办法。附上博客地址：

[跨双栏长公式的置顶/置底](https://agnesedong.wordpress.com/2010/01/13/%E5%88%86%E4%BA%AB%E4%B8%80%E5%93%88%EF%BC%8C%E8%B7%A8%E5%8F%8C%E6%A0%8F%E9%95%BF%E5%85%AC%E5%BC%8F%E7%9A%84%E7%BD%AE%E9%A1%B6%E7%BD%AE%E5%BA%95/)

```latex
\begin{figure*}
\begin{equation}
    M_{opt}=
    \begin{Bmatrix}
        \begin{aligned}
            & \left \{ HC_1\leftrightarrow \left \{ HC_2,\cdots ,HC_{1+p_1^*} \right \} \right \},\\ 
            & \left \{ HC_{2+p_1^*}\leftrightarrow \left \{ HC_{3+p_1^*},\cdots ,HC_{p_1^*+p_2^*+2} \right \} \right \},\cdots,\\ 
            & \left \{ HC_{\sum_{i=1}^{N-1}p_i^*+N}\leftrightarrow \left \{ HC_{\sum_{i=1}^{N-1}p_i^*+N+1},\cdots ,HC_{\sum_{i=1}^{N}p_i^*+N} \right \} \right \}  
        \end{aligned}
    \end{Bmatrix}
    \label{eq6} 
\end{equation} 
\end{figure*}
```

上面这段代码就是图中双栏公式的实现代码，效果还算可以。

## 参考文献的引用

还记得我第二篇paper在编辑参考文献bib文件的时候，一篇一篇地去Google scholar上查，然后一个属性一个属性的复试粘贴QAQ，想想都是泪，真的蠢。这次从Google scholar上发现可以直接点击BibTex就可以直接复制整段引用信息了，然后粘贴到bib文件里，效率比当时是快不少了。但是这样只能一篇一篇的复制，我不知道怎么批量导入，这个还需要查查资料才行。

## 表格内文字自动换行问题

在写算法步骤时，很多时候是采用表格形式展示的，这时候就要使得文字过长的一行可以自动换行了，查询资料之后，找到一种办法效果挺好，就是限定每列固定长度：

```latex
\begin{table}[!ht]
\renewcommand\arraystretch{1.3}
\caption{Embedding steps of the proposed scheme.}
\label{tab2}
\begin{tabular}{p{1.1cm}p{6.9cm}}
\toprule
\multicolumn{2}{l}{\textbf{Input:} Original JPEG bitstream $J$ and secret data.}  \\
\multicolumn{2}{l}{\textbf{Output:}  Marked JPEG bitstream $J_M$. }  \\ \midrule
\textbf{Step 1.} & Parse the entropy coded data and the DHT segment from $J$.                                                        \\
\textbf{Step 2.} & Count the frequency of RSVs according to the corresponding Huffman codes and sort the RSVs in descending order. \\
\textbf{Step 3.} & Construct the optimal mapping relationship $M_{opt}$ according to the given payload.                                 \\
\textbf{Step 4.} & Replace the original Huffman codes in $J$ with the Huffman codes in the mapping set to embed data.                \\
\textbf{Step 5.} & Modify the RSVs in the DHT segment according to $M_{opt}$, then $J_M$ is generated.                                     \\ \bottomrule
\end{tabular}
\end{table}
```

得到的效果图如下：

![表格文字自动换行](http://poex2tesj.bkt.clouddn.com/Snipaste_2019-03-15_23-09-54.png)

我觉得效果还是不错的，这段代码核心就是

```latex
\begin{tabular}{p{1.1cm}p{6.9cm}}
```

通过指定每列固定长度来使得超过长度文字能够自动换行。

## 表格行间距设置问题

就一行代码，比较简单，在\begin{table}下加上

```latex
\renewcommand\arraystretch{1.5}
```

即可，很实用。