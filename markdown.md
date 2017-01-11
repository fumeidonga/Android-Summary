# 标题1 #
## 标题2 ##
### 标题3 ###
#### 标题4 ####
##### 标题5 #####
###### 标题6 ######

##换行
加两个空格再回车

##段落跟代码##
<p>这是一个段落</p>
<pre>
<code>
public void deleteMenu() {

        WxMpMenuService wxMpMenuService = null;
        try {
        	
        	wxMpMenuService = wxMpService.getMenuService();
            //删除菜单
        	wxMpMenuService.menuDelete();
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
</code>
</pre>


## 区块引用 > ##
> 1、这是区块引用，注意看效果。
>> 2、 ####这里是两次嵌套效果
>>> 3、 <p>这是三层嵌套效果 </p>
>>>> 4、这里是代码 <pre><code>public void deleteMenu() {
 WxMpMenuService wxMpMenuService = null;
        try {
        	wxMpMenuService = wxMpService.getMenuService();
        	 //删除菜单
        	wxMpMenuService.menuDelete();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }</code></pre>
       
##列表##
###无序列表
* red
* blue
+ green
+ white
- yellow
- red  
 
###有序列表
1. red
2. yellow
3. green

##分割线 
***
---
___


##链接
###方式一
[hyperlink syntax](http://markdownpad.com),
###方式二
[hyperlink syntax] [id]

[id]:http://markdownpad.com 'title here'



***
##强调
*一个号的强调*

**两个号的强调**
 
_1111_  
__1111__
___

##图片
###方式一
![图片1](C:\\Users\\david\\Desktop\\1.jpg)
###方式二
![图片2] [id]

[id]:C:\Users\david\Desktop\1.jpg   

## 页内跳转   
* [1.语法示例](#1)   
* [1.1图片](#jump)    
* [1.2换行](#1.2)   
* [1.3强调](#1.3)   

   
##反斜杠 \
\*

\!

...


## 专为Markdown设计 ##

Enjoy first-class Markdown support with easy access to  Markdown syntax and convenient keyboard shortcuts.

Give them a try:

- **Bold** (`Ctrl+B`) and *Italic* (`Ctrl+I`)
- Quotes (`Ctrl+Q`)
- Code blocks (`Ctrl+K`)
- Headings 1, 2, 3 (`Ctrl+1`, `Ctrl+2`, `Ctrl+3`)
- Lists (`Ctrl+U` and `Ctrl+Shift+O`)

### See your changes instantly with LivePreview ###

Don't guess if your [hyperlink syntax](http://markdownpad.com) is correct; LivePreview will show you exactly what your document looks like every time you press a key.

### Make it your own ###

Fonts, color schemes, layouts and stylesheets are all 100% customizable so you can turn MarkdownPad into your perfect editor.

### A robust editor for advanced Markdown users ###

MarkdownPad supports multiple Markdown processing engines, including standard Markdown, Markdown Extra (with Table support) and GitHub Flavored Markdown.

With a tabbed document interface, PDF export, a built-in image uploader, session management, spell check, auto-save, syntax highlighting and a built-in CSS management interface, there's no limit to what you can do with MarkdownPad.

<h2 id="1">1.语法示例</h2>
**效果查看markdown.md文件   

<span id="jump">1.1图片</span>


