# 介绍

搭建Sphinx+GitHub+ReadtheDocs的文档写作工具，用Sphinx生成文档，GitHub托管文档，再导入到ReadtheDocs。

## 安装Sphinx
```
pip install sphinx sphinx-autobuild sphinx_rtd_theme
```

## 初始化工程
创建工程目录，在工程目录下执行如下指令：
```
sphinx-quickstart
```
系统会引导生成sphinx工程。
```
> Separate source and build directories (y/n) [n]:y
> Project name: docs-cook
> Author name(s): ted.huang
> Project version []: 0.1
> Project release [1.0]: 0.1
> Project language [en]: zh_CN
```

## 更改主题 sphinx_rtd_theme
修改source/conf.py，添加如下内容：
```
import sphinx_rtd_theme
html_theme = "sphinx_rtd_theme"
html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]
```

## 预览效果
在根目录执行如下指令：
```
make html
```
进入build/html目录下，浏览器打开index.html本地浏览。

## 支持markdown编写
通过recommonmark 来支持markdown，安装指令如下：
```
pip install recommonmark
```

添加修改source/conf.pyn内容如下：
```
from recommonmark.parser import CommonMarkParser
source_parsers = {
    '.md': CommonMarkParser,
}
source_suffix = ['.rst', '.md']
```

## 导入ReadtheDocs系统
将上面创建的工程上传至github，在 ReadtheDocs系统Import github工程，均默认创建即可，创建完后会自动去激活Webhooks，不用再去GitHub设置。一切搞定后，从此只要你往这个仓库push代码，ReadtheDocs上面的文档就会自动更新。

注：在创建read the docs项目时候，语言选择”Simplified Chinese”