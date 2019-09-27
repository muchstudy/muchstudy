---
title: Markdown转HTML
date: 2017-01-18 17:28:33
categories:
- FrontEnd
---

&emsp;&emsp;目前，产品中所有前端组件的API文档均由Markdown编写，之前的处理方式为在Atom中写好，然后另存为html的方式放到产品的在线样例库中，最近越来越觉得这种方式太麻烦，不易于管理。考虑砍掉另存为html的步骤，直接在webstorm中写文档，然后在运行时动态把markdown编译成html文档，跟目前的很多在线的Markdown文本编辑器的原理类似。

&emsp;&emsp;经过调研，可以后端转，也可以前端转。最终，采用前端JS来把Markdown转换为HTML。采用<a href="https://github.com/showdownjs/showdown">showdown</a>来实现。

&emsp;&emsp;从showdown的github主页上看到过程挺简单，引入js文件，然后按照下面的语法转换即可：
```javascript
var converter = new showdown.Converter(),
    text      = '#hello, markdown!',
    html      = converter.makeHtml(text);
```

&emsp;&emsp;经过验证，发现默认是不对表格做转换的。查看<a href="http://showdownjs.github.io/demo/">官方DEMO</a>,发现在官方DEMO上一切正常，接着发现原来需要做一些配置项。懒得去一个个看API配置，直接在官方的Demo中加个断点，通过`getOptions`方法拿到DEMO样例的配置项，结果如下：

```javascript
//docurl即xxx.md文件，这里也可以使用jquery来加载指定url的markdown文件
require([ "text!" + docUrl ], function(doc) {
    var converter = new showdown.Converter({
        "omitExtraWLInCodeBlocks":true,
        "noHeaderId":false,
        "prefixHeaderId":"",
        "ghCompatibleHeaderId":true,
        "headerLevelStart":1,
        "parseImgDimensions":true,
        "simplifiedAutoLink":true,
        "excludeTrailingPunctuationFromURLs":false,
        "literalMidWordUnderscores":true,
        "strikethrough":true,
        "tables":true,
        "tablesHeaderId":false,
        "ghCodeBlocks":true,
        "tasklists":true,
        "smoothLivePreview":true,
        "smartIndentationFix":false,
        "disableForced4SpacesIndentedSublists":false,
        "simpleLineBreaks":false,
        "requireSpaceBeforeHeadingText":false,
        "ghMentions":false,"extensions":[],"sanitize":false
    });
    var html = converter.makeHtml(doc);
    //接着把html放到DOM上即可
    //......
    //我这里是弹出一个侧边栏，在侧边栏上显示markdown所写的API内容
    Util.slidebar({
        body : html,
        width : slideWidth || "800px",
        afterLoad : function($elem){
            //容器的跟节点上加上class，解决markdown转换为html后样式不对的问题
            $elem.addClass("markdown-body");
            $elem.find("table").addClass("table table-bordered table-hover");

        }
    });
})
```

&emsp;&emsp;通过上述代码，能够正常把markdown转换为html。但是，发现样式有问题，表格没有边框。从上面也看到了，额外增加了样式。

&emsp;&emsp;最后，把样式文件`markdown.css`也分享出来。当然，如果想结果跟showdown的DEMO样例一样，也可以去把DEMO的样式扒下来。

```css
.markdown-body {
    overflow: hidden;
    font-family: "Helvetica Neue", Helvetica, "Segoe UI", Arial, freesans, sans-serif;
    font-size: 14px;
    line-height: 1.6;
    word-wrap: break-word
}

ol {
    list-style: decimal
}

ul {
    list-style: square
}

.markdown-body > *:first-child {
    margin-top: 0 !important
}

.markdown-body > *:last-child {
    margin-bottom: 0 !important
}

.markdown-body a:not([href]) {
    color: inherit;
    text-decoration: none
}

.markdown-body .absent {
    color: #c00
}

.markdown-body .anchor {
    position: absolute;
    top: 0;
    left: 0;
    display: block;
    padding-right: 6px;
    padding-left: 30px;
    margin-left: -30px
}

.markdown-body .anchor:focus {
    outline: none
}

.markdown-body h1, .markdown-body h2, .markdown-body h3, .markdown-body h4, .markdown-body h5, .markdown-body h6 {
    position: relative;
    margin-top: 1em;
    margin-bottom: 16px;
    font-weight: bold;
    line-height: 1.4
}

.markdown-body h1 .octicon-link, .markdown-body h2 .octicon-link, .markdown-body h3 .octicon-link, .markdown-body h4 .octicon-link, .markdown-body h5 .octicon-link, .markdown-body h6 .octicon-link {
    display: none;
    color: #000;
    vertical-align: middle
}

.markdown-body h1:hover .anchor, .markdown-body h2:hover .anchor, .markdown-body h3:hover .anchor, .markdown-body h4:hover .anchor, .markdown-body h5:hover .anchor, .markdown-body h6:hover .anchor {
    padding-left: 8px;
    margin-left: -30px;
    text-decoration: none
}

.markdown-body h1:hover .anchor .octicon-link, .markdown-body h2:hover .anchor .octicon-link, .markdown-body h3:hover .anchor .octicon-link, .markdown-body h4:hover .anchor .octicon-link, .markdown-body h5:hover .anchor .octicon-link, .markdown-body h6:hover .anchor .octicon-link {
    display: inline-block
}

.markdown-body h1 tt, .markdown-body h1 code, .markdown-body h2 tt, .markdown-body h2 code, .markdown-body h3 tt, .markdown-body h3 code, .markdown-body h4 tt, .markdown-body h4 code, .markdown-body h5 tt, .markdown-body h5 code, .markdown-body h6 tt, .markdown-body h6 code {
    font-size: inherit
}

.markdown-body h1 {
    padding-bottom: 0.3em;
    font-size: 2.25em;
    line-height: 1.2;
    border-bottom: 1px solid #eee
}

.markdown-body h1 .anchor {
    line-height: 1
}

.markdown-body h2 {
    padding-bottom: 0.3em;
    font-size: 1.75em;
    line-height: 1.225;
    border-bottom: 1px solid #eee
}

.markdown-body h2 .anchor {
    line-height: 1
}

.markdown-body h3 {
    font-size: 1.5em;
    line-height: 1.43
}

.markdown-body h3 .anchor {
    line-height: 1.2
}

.markdown-body h4 {
    font-size: 1.25em
}

.markdown-body h4 .anchor {
    line-height: 1.2
}

.markdown-body h5 {
    font-size: 1em
}

.markdown-body h5 .anchor {
    line-height: 1.1
}

.markdown-body h6 {
    font-size: 1em;
    color: #777
}

.markdown-body h6 .anchor {
    line-height: 1.1
}

.markdown-body p, .markdown-body blockquote, .markdown-body ul, .markdown-body ol, .markdown-body dl, .markdown-body pre {
    margin-top: 0;
    margin-bottom: 16px
}

.markdown-body hr {
    height: 4px;
    padding: 0;
    margin: 16px 0;
    background-color: #e7e7e7;
    border: 0 none
}

.markdown-body ul, .markdown-body ol {
    padding-left: 2em
}

.markdown-body ul.no-list, .markdown-body ol.no-list {
    padding: 0;
    list-style-type: none
}

.markdown-body ul ul, .markdown-body ul ol, .markdown-body ol ol, .markdown-body ol ul {
    margin-top: 0;
    margin-bottom: 0
}

.markdown-body li > p {
    margin-top: 16px
}

.markdown-body dl {
    padding: 0
}

.markdown-body dl dt {
    padding: 0;
    margin-top: 16px;
    font-size: 1em;
    font-style: italic;
    font-weight: bold
}

.markdown-body dl dd {
    padding: 0 16px;
    margin-bottom: 16px
}

.markdown-body blockquote {
    padding: 0 15px;
    color: #777;
    border-left: 4px solid #ddd;
    font-size: 14px;
}

.markdown-body blockquote > :first-child {
    margin-top: 0
}

.markdown-body blockquote > :last-child {
    margin-bottom: 0
}

.markdown-body .table-bordered>thead>tr>th{
    border-bottom-width: 0;
}
/*.markdown-body table {
display: block;
width: 100%;
overflow: auto;
word-break: normal;
word-break: keep-all
}

.markdown-body table th {
font-weight: bold
}

.markdown-body table th, .markdown-body table td {
padding: 6px 13px;
border: 1px solid #ddd
}

.markdown-body table tr {
background-color: #fff;
border-top: 1px solid #ccc
}

.markdown-body table tr:nth-child(2n) {
background-color: #f8f8f8
}*/

.markdown-body img {
    max-width: 100%;
    box-sizing: border-box
}

.markdown-body .emoji {
    max-width: none
}

.markdown-body span.frame {
    display: block;
    overflow: hidden
}

.markdown-body span.frame > span {
    display: block;
    float: left;
    width: auto;
    padding: 7px;
    margin: 13px 0 0;
    overflow: hidden;
    border: 1px solid #ddd
}

.markdown-body span.frame span img {
    display: block;
    float: left
}

.markdown-body span.frame span span {
    display: block;
    padding: 5px 0 0;
    clear: both;
    color: #333
}

.markdown-body span.align-center {
    display: block;
    overflow: hidden;
    clear: both
}

.markdown-body span.align-center > span {
    display: block;
    margin: 13px auto 0;
    overflow: hidden;
    text-align: center
}

.markdown-body span.align-center span img {
    margin: 0 auto;
    text-align: center
}

.markdown-body span.align-right {
    display: block;
    overflow: hidden;
    clear: both
}

.markdown-body span.align-right > span {
    display: block;
    margin: 13px 0 0;
    overflow: hidden;
    text-align: right
}

.markdown-body span.align-right span img {
    margin: 0;
    text-align: right
}

.markdown-body span.float-left {
    display: block;
    float: left;
    margin-right: 13px;
    overflow: hidden
}

.markdown-body span.float-left span {
    margin: 13px 0 0
}

.markdown-body span.float-right {
    display: block;
    float: right;
    margin-left: 13px;
    overflow: hidden
}

.markdown-body span.float-right > span {
    display: block;
    margin: 13px auto 0;
    overflow: hidden;
    text-align: right
}

.markdown-body code, .markdown-body tt {
    padding: 0;
    padding-top: 0.2em;
    padding-bottom: 0.2em;
    margin: 0;
    font-size: 85%;
    background-color: rgba(0, 0, 0, 0.04);
    border-radius: 3px
}

.markdown-body code:before, .markdown-body code:after, .markdown-body tt:before, .markdown-body tt:after {
    letter-spacing: -0.2em;
    content: "\00a0"
}

.markdown-body code br, .markdown-body tt br {
    display: none
}

.markdown-body del code {
    text-decoration: inherit
}

.markdown-body pre > code {
    padding: 0;
    margin: 0;
    font-size: 100%;
    word-break: normal;
    white-space: pre;
    background: transparent;
    border: 0
}

.markdown-body .highlight {
    margin-bottom: 16px
}

.markdown-body .highlight pre, .markdown-body pre {
    padding: 16px;
    overflow: auto;
    font-size: 85%;
    line-height: 1.45;
    background-color: #f7f7f7;
    border-radius: 3px
}

.markdown-body .highlight pre {
    margin-bottom: 0;
    word-break: normal
}

.markdown-body pre {
    word-wrap: normal
}

.markdown-body pre code, .markdown-body pre tt {
    display: inline;
    max-width: initial;
    padding: 0;
    margin: 0;
    overflow: initial;
    line-height: inherit;
    word-wrap: normal;
    background-color: transparent;
    border: 0
}

.markdown-body pre code:before, .markdown-body pre code:after, .markdown-body pre tt:before, .markdown-body pre tt:after {
    content: normal
}

.markdown-body kbd {
    display: inline-block;
    padding: 3px 5px;
    font-size: 11px;
    line-height: 10px;
    color: #555;
    vertical-align: middle;
    background-color: #fcfcfc;
    border: solid 1px #ccc;
    border-bottom-color: #bbb;
    border-radius: 3px;
    box-shadow: inset 0 -1px 0 #bbb
}

.markdown-body ul {
    list-style: disc
}

.markdown-body ul ul, markdown-body ol ul {
    list-style: circle
}
```
