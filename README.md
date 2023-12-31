## SantyPilot 是什么

SantyPilot是一个开源飞控项目，前身是[LibrePilot](https://github.com/librepilot/)  
SantyPilot聚焦于教学与科研领域，专注于固定翼与多旋翼性能调优与算法验证  
通过轻量模块化的框架，和配套的开发规范，方便新手快速上手  

## SantyPilot官方文档
[官方文档链接](https://santypilot.github.io/SantyPilot-doc/GLOSSARY.html)  
本项目为SantyPilot官方文档代码仓，您可以自由更新其中内容  
假定您已经搭建完成本地gitbook开发环境  
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
git checkout $YOUR_PAGE_BRANCH # check into temp dirs
cp -rf ../temp/* .

git add ./
git commit -m "UPDATE PAGE FILES"
git push origin $YOUR_PAGE_BRANCH # make another push

# then make two PR: one $YOUR_BRANCH->dev
# the other $YOUR_PAGE_BRANCH->gh-pages
```

## SantyPilot生态推广
欢迎关注我们的公众号
<table align="center">
  <tr>
    <td>
      <img src="https://github.com/SantyPilot/SantyPilot-doc/blob/master/SantyNotebook.jpg" width="250px">
    </td>
  </tr>
</table>
如果您有任何科研教学上的合作或需求，可以发邮件联系我（小新）   
 
邮箱：Xiaosanti@gmail.com  963786615@qq.com   
 
QQ:963786615   

