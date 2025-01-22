# LaTeX

## 说明

LaTeX 是一个文字排版系统，用于处理篇幅较长，结构严谨的文档  

与 Markdown 相比，它提供了更多更细致的排版选项  

## LateX 引擎、格式、发行版

### LaTeX 引擎

引擎用于将 Tex 文档的内容转换为特定格式的语言  
> 编译器  
>
最初的 TeX 引擎为 TeX 作者 Donald Knuth 开发的 Knuth TeX，但如今已不再使用  
e-TeX 是 Knuth TeX 的后继，也是目前事实上的标准 TeX 引擎，后续的 TeX 编译器都基于此  
> e for extended  
>
pdfTeX 为西文世界最常用的 TeX 编译器  
LuaTeX 为 pdfeX 的后继者，加入了 Lua 脚本及 Unicode 的支持  
XeTeX 为基于 e-TeX 的另一个分支，且同样原生支持 Unicode  
pTeX 及其后继，统称为 pTeX 系引擎
> p for publish  
>
### LaTeX 格式

TBD

### LaTeX 发行版

TeX Live  
TBD

## 基础语法

Latex 的命令是大小写敏感的  

### 文档结构

一个基本的 LaTeX 文档如下

```LaTeX
\documentclass[a4paper, 12pt]{article}
\begin{document}
    Basic LaTeX document
\end{document}
```

必须以 `\documentclass` 开始  
其后的花括号中指明该文档的类型

### 文档标题

```LaTeX
\title{Title Name}
\author{Author Name}
\date{\today}
\maketitle
```

以上内容需包含于 `{document}` 之间  
其中 `\title` 必须，`\author` 非必须，`\date` 非必须
当不显式使用 `\date` 时默认为当前时间，当花括号内为空时 `\date{}` 不显示时间  

对于 article 类型的文档，标题之后紧随正文  
对于 report 类型的文档，标题单独为一页  

### 章节

分节命令遵循如下格式  

```LaTeX
\section{Section Title}
something
```

花括号内容为该章节标题，另起一行为正文内容  

对于 article 类型的文档，适用如下分节命令  

- \section{...}
- \subsection{...}
- \subsubsection{...}
- \paragraph{...}
- \subparagraph{...}

#### 标签  

```LaTeX
\section{Section Title}
\label{sec1}something

% ...
\section{Other Section}
\ref{sec1}  \pageref{sec1}
```

可以对章节命令创建标签并在文档的其他部分引用  
`\ref` 将被编译为所引用章节的编号，`\pageref` 将编译为所引用章节的起始页码  

#### 目录

```LaTeX
\tableofcontents
```

#### 章节缩进  

默认每个章节第一段首行顶格，之后的段落首行缩进  
对于需要顶格的段落，可使用 `\noindent`  
需要所有段落顶格，使用 `\setlength{\parindent}{0pt}`  

### 列表  

LaTeX 支持有序列表与无序列表，且两者可互相嵌套  

#### 有序列表  

```LaTeX
\begin{itemize}
  \item[-] Item Content
\end{itemize}
```

其中，方括号内为无序列表每项前的标志  

```LaTeX
\begin{enumerate}
  \item Item Concent
\end{enumerate}
```

#### 无序列表  

## 进阶语法

### 字体与字号

#### 字体效果

部分字体效果命令如下  

|命令|说明|
|-|-|
|\textit|斜体|
|\textbf|粗体|
|\underline|下划线|

#### 字号  

||
|-|
|\tiny|
|\scriptsize|
|\footnotesize|
|\small|
|\large|
|\Large|
|\LARGE|
|\huge|

## 扩展

### LaTex 中文支持  

在文档的导言部分引入 CTeX 宏包

```LaTeX
\usepackage[UTF8]{ctex}
```

### LaTeX 中使用 Markdown

TBD
