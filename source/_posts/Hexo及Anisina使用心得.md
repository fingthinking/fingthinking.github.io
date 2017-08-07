---
title: Hexo及Anisina使用心得
date: 2017-08-05 23:47:16
tags:
	- Anisina
---

## Anisina相关链接
[Anisina使用教程](http://haojen.github.io/2017/05/09/Anisina-%E4%B8%AD%E6%96%87%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/)  
[Anisina Github](https://github.com/Haojen/hexo-theme-Anisina)

## Hexo使用中遇到的问题
Hexo初次使用的教程，就不再赘述，附上我参考的博客：
[手把手教你用Hexo+Github 搭建属于自己的博客](http://blog.csdn.net/gdutxiaoxu/article/details/53576018)

#### 问题1：无法部署项目到github中？  
解决办法：   

a.检查`_config.yml`文件中github的配置

```yml
deploy:
  type: git
  repo: git@github.com:xxxx/xxx.github.io.git
  branch: master
```
b.安装hexo-deployer-git插件  
> npm install hexo-deployer-git --save

#### 问题2: 使用Anisina主题，hexo new page xxx之后，hexo g报错？

```javascript
Unhandled rejection ReferenceError: /Users/liuruteng/blog/fingthinking/themes/Anisina/layout/page.ejs:145
    143|     <div class="row">
    144|         <div class="col-lg-8 col-lg-offset-1 col-md-8 col-md-offset-1 col-sm-12 col-xs-12 post-container">
 >> 145|             <%- body %>
    146|         </div>
    147|         <!-- Sidebar Container -->
    148|         <div class="

body is not defined
    at eval (eval at exports.compile (/Users/liuruteng/blog/fingthinking/node_modules/ejs/lib/ejs.js:242:14), <anonymous>:30:4567)
    at eval (eval at exports.compile (/Users/liuruteng/blog/fingthinking/node_modules/ejs/lib/ejs.js:242:14), <anonymous>:30:12223)
    at /Users/liuruteng/blog/fingthinking/node_modules/ejs/lib/ejs.js:255:15
    at Theme._View.View._compiled (/Users/liuruteng/blog/fingthinking/node_modules/hexo/lib/theme/view.js:127:30)
    at Theme._View.View.View.render (/Users/liuruteng/blog/fingthinking/node_modules/hexo/lib/theme/view.js:29:15)
    at /Users/liuruteng/blog/fingthinking/node_modules/hexo/lib/hexo/index.js:390:25
    at tryCatcher (/Users/liuruteng/blog/fingthinking/node_modules/bluebird/js/release/util.js:16:23)
    at /Users/liuruteng/blog/fingthinking/node_modules/bluebird/js/release/method.js:15:34
    at RouteStream._read (/Users/liuruteng/blog/fingthinking/node_modules/hexo/lib/hexo/router.js:134:3)
    at RouteStream.Readable.read (_stream_readable.js:431:10)
    at resume_ (_stream_readable.js:811:12)
    at _combinedTickCallback (internal/process/next_tick.js:138:11)
    at process._tickCallback (internal/process/next_tick.js:180:9)
```
这种问题主要是由于page.ejs模板文件无法找到`body`变量导致的异常。其中`body`变量是由hexo定义，通过**front-matter**
提供，作为**“子模板的内容嵌入”**

在Anisina主题中，有`index.ejs`作为`page.ejs`的子模板,有如下代码

```yml index.ejs
---
layout: 
---
```
说明，每次调用`index.ejs`的时候，都将会调用`page.ejs`进行渲染，而`index.ejs`将作为`page.ejs`的**body**变量传入其中，于是有了上文中的***145***行的`<%- body%>`的错误。

解决方案给出两种，我使用第二种：  

1. 使用命令`hexo new page xxx`之后，修改*'source/xxx/index.md'*的front-matter部分，添加`layout: TT`,其中的`TT`不能为page，具体的值，请参考***"Anisina/layout/\*.ejs"***中的\*的值；

2. 进入***"Anisina/layout/”***目录，执行如下命令

```shell
# 移动page.ejs为m_page.ejs
mv page.ejs m_page.ejs  
# 将index.ejs的第二行替换，就是将index.ejs作为m_page.ejs的子模板
sed -i="" '2s/^.*/layout: m_page/' index.ejs 
# 创建新的page.ejs模板文件，并作为m_page.ejs的子模板
echo -e "---\nlayout: m_page\n---\n<%- page.content %>" > page.ejs 
```
