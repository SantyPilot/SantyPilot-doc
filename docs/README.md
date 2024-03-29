## SantyPilot 是什么

[SantyPilot](https://github.com/SantyPilot)是一个开源飞控项目，前身是[LibrePilot](https://github.com/librepilot/)  
SantyPilot聚焦于教学与科研领域，专注于固定翼与多旋翼性能调优与算法验证  
通过轻量模块化的框架，和配套的开发规范，方便新手快速上手  

## SantyPilot官方文档
[官方文档链接](https://santypilot.github.io/SantyPilot-doc/)  
本项目为SantyPilot官方文档，您可以自由更新其中内容  
如果您在环境配置，开发，使用过程中遇到任何问题    
可以到代码仓[issue页面](https://github.com/SantyPilot/SantyPilot-doc/issues)将问题描述清楚，附上报错截图，提交给我们   
搭建本地SantyPilot-doc开发环境  
```bash
node -v
v14.17.0
npm -v
6.14.13
npm install gitbook
```
您可以按照如下方法构建与更新SantyPilot文档  
```bash
git clone git@github.com:SantyPilot/SantyPilot-doc.git
cd SantyPilot-doc
# do some modification
gitbook build # build .md files
gitbook serve # preview on localhost:4000

git add ./
git commit -m "ADD NEW FEATURE"
git push origin $YOUR_BRANCH # make a push

mkdir ../temp # make a temp dir
cp -rf ./_book/* ../temp # cp page files into temp
git checkout $YOUR_PAGE_BRANCH # check into your develop branch
cp -rf ../temp/* .

git add ./
git commit -m "UPDATE PAGE FILES"
git push origin $YOUR_PAGE_BRANCH # make another push

# then make two PR: one $YOUR_BRANCH->dev
# the other $YOUR_PAGE_BRANCH->gh-pages
```
如果遇到如下报错
```
Installing GitBook 3.2.2
/usr/local/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287
      if (cb) cb.apply(this, arguments)
```
您可以编辑报错文件把报错跳过去
```
vim /usr/local/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js
```
注释掉
```js
62   //fs.stat = statFix(fs.stat)                                              
63   //fs.fstat = statFix(fs.stat)                                              
64   //fs.lstat = statFix(fs.stat)                                              
```
重新构建即可通过

## SantyPilot 生态推广
欢迎关注我们的公众号，了解前沿咨询，构建无人机全栈开发能力
<table class="santynotebook">
  <tr>
    <td>
      <img align="center" src="./SantyNotebook.jpg" width="250px">
    </td>
  </tr>
</table>
<style>
.santynotebook {
    text-align:center;
}
</style>
如果您有任何科研教学上的合作或需求，可以发邮件联系我（小新）   
 
<table>
<tr>
    <td>邮箱</td>
    <td>xiaosanti213@gmail.com</td>
</tr>
<tr>
    <td>邮箱</td>
    <td>963786615@qq.com</td>
</tr>
<tr>
    <td>QQ</td>
    <td>963786615</td>
</tr>
</table>
