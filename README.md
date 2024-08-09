# PEP503-ZH
An unofficial translation of PEP 503.

- 原文链接： https://peps.python.org/pep-0503/
- 翻译日期：`2024-08-09`
- 免责声明：此翻译版本仅用于个人研究学习用途，不具有 PEP 协议的权威性。本译文不对翻译的正确性负责，也不对相关衍生作品内容的正确性负责。

- Original URL：https://peps.python.org/pep-0503/
- Translation Date：`2024-08-09`
- Disclaimer: This translation is only for personal  study purposes and is not authorized by PEP administration. This translation is not responsible for the incorrectness of the relevant facts, nor is it responsible for the incorrectness of the content of related derivative works.



# PEP 503

## 摘要

当下，Python 包仓库有多种不同实现，针对不同的实现，衍生出了多种不同的工具。其中，规范实现中定义了 PyPI 所使用的 “简单” 存储库 API（Simple Repository API）。本文将定义该 API，并规定任何 “简单” 存储库 API 的具体实现应该具有的正确行为。

## 规格说明

当我们定义一个实现了简单 API 的存储库时，我们需要给出一个根 URL，该存储库中的所有其他 URL 都位于该 URL 目录之下。之所以本 API 名为 “简单（simple）” 代码库 API，是因为在 PyPI 中，该根 URL 为 `https://pypi.org/simple/`。

>注意
>
>下文中提及的所有的 URL，不加说明地，均表示相对于该根 URL 的相对路径。（例如，对 PyPI 的 URL 而言，URL `/foo/` 将指代 `https://pypi.org/simple/foo/`。）

在一个代码仓库内，根 URL （在本 PEP 中， `/` 指的是某个代码仓库的根 URL）**必须（MUST）**是一个有效的 HTML5 页面，并且在该页面中，有指向代码库中各个项目的锚（anchor）。锚的文本信息必须与被指向的项目名一致，且锚的超链接目标（href）必须指向该项目的 URL。下面是一个 HTML 实例：

```html
<!DOCTYPE html>
<html>
  <body>
    <a href="/frob/">frob</a>
    <a href="/spamspamspam/">spamspamspam</a>
  </body>
</html>
```

在根 URL 之下，代码仓库中的每个项目都有一个自己的 URL。该 URL 的形式为 `/<project>/`，其中 `<project>` 应被替换为该项目的正规名（normalized name），因此，名为 "HolyGrail" 的项目在代码仓库中的 URL 将是 `/holygrail/`。这些项目自己的 URL 也必须是有效的 HTML5 页面，该页面中，项目中内的各个文件分别具有一个指向其位置的锚元素。锚元素的超链接目标（href）**必须（MUST）**指向下载该文件时所用的 URL，且该锚元素的文本内容**必须（MUST）**与该文件的文件名一致。指向文件的 URL **应该（SHOULD）** 使用一个形如 `#<hashname>=<hashvalue>` 的 URL 片段附带一个文件的哈希值。其中 `<hashname>` 是采用的哈希算法的名称的小写形式（如 `sha256`），`<hasvalue>` 是十六进制编码的哈希校验和。

除此之外，我们和;要求 API 中的所有 URL 必须符合以下约束：

- 与 HTML5 页面相对应的所有 URL 必须以字符 `/` 为结尾，且代码仓库应该将不以字符 `/` 为结尾的 URL 重定向到以字符 `/` 为结尾的 URL（即在尾部追加一个 `/`）；

- URL 可以使用相对地址，也可以使用绝对地址，只要能指向正确的位置即可；

- 我们并不限制在代码仓库中项目的实际存储位置（只要通过 URL 能访问到指定的资源即可）；

- 上文中提及的 HTML5 页面中可以有其他无关的 HTML 元素，只要必要的锚元素存在即可；

- 代码仓库**可以（MAY）**对非规范化的 URL 进行必要的重定向（例如将 `/Foobar/` 重定向至 `/foobar/`），但客户端**禁止（MUST NOT）**依赖这一重定向特性，客户端必须向规范的 URL 发起请求；

- 代码仓库**应当（SHOULD）**选择一种 Python 标准库中 `hashlib` 模块支持的哈希函数（目前支持的哈希函数包括 `md5`, `sha1`, `sha256`, `sha384`, `sha512`）。目前建议使用 `sha256`；

- 当某个特定分发版本带有 GPG 签名时，在同目录下，须有一个文件名一直且带 `.asc` 拓展名的文件。例如，假设 `/packages/HolyGrail-1.0.tar.gz` 存在且有与之关联的签名，则签名应被放置在 `/packages/HolyGrail-1.0.tar.gz.asc`；

- 代码仓库可以在链接中引入一个 `data-gpg-sig` 属性，并用 `true` 或者 `false` 表明该文件是否有 GPG 签名。若代码仓库采用了这一策略，则必须在所有链接中均引入这一属性；

- 代码仓库可以在链接中引入一个 `data-requires-python` 属性。该属性中可以使用 Requires-Python 元信息域指出相应的版本信息，详见 [PEP 345](https://peps.python.org/pep-0345/)。当该字段出现时，客户端**应当（SHOULD）**忽略那些不符合版本约束的被依赖项。下面给出一个可能的例子：

  ```html
  <a href="..." data-requires-python="&gt;=3">...</a>
  ```

  在 `data-requires-python` 字段中，小于号 `<` 以及大于号 `>` 应当使用其各自 HTML 转义，即 `&lt;` 以及 `&gt;`。

### 规范名称

本 PEP 引用了规范项目名称的概念。如 [PEP 426](https://peps.python.org/pep-0426/) 所定义，文件名中可以出现的所有合法字符包括 ASCII 字母、ASCII 数字，`.`, `-` 以及 `_`。规范名称要求文件名中的所有字母均为小写，且字符 `.`, `_` 出现时均应被替换为一个 `-`。使用 Python 的 `re` 模块可以实现这一功能：

```python
import re

def normalize(name):
    return re.sub(r"[-_.]+", "-", name).lower()
```

### 历史变更

- 2016 年六月加入了 `data-requires-python` 这一可选字段。

## 版权

本文件从属于公有域。

---

源代码：https://github.com/python/peps/blob/main/peps/pep-0503.rst

上次修改时间：`2023-09-09 17:39:29 GMT`

