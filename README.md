# zaynr.github.io
首先我们进入到博客系统的根目录，比如blog目录，这里边有个.gitignore文件（如果该文件不存在，自己创建一个），里边默认已经把该忽略的目录文件都写好了，里边内容如下：
```bash
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/%
```
然后在blog目录初始化仓库，切换到source分支，关联远程仓库，并push到远程仓库的source分支
```bash
$ cd blog
$ git init
$ git checkout -b source
$ git add .
$ git commit -m "first commit"
$ git remote add origin git@github.com:change2hao/change2hao.github.io.git
$ git push origin source
```
操作完成后，在另外一台电脑上，先把node环境配好，安装hexo。

```bash
$ brew update //很重要
$ brew install node
$ npm install hexo-cli -g
```
注意不要再执行：

```bash
$ hexo init blog
```
取而代之的是

```bash
$ git clone -b source git@github.com:change2hao:change2hao.github.io.git
$ npm install //根据package.json来下载依赖包
```bash
这样把远程仓库的source分支克隆下来，然后安装依赖包。接下来我们就可以继续写博客了

```bash
$ hexo new "about hexo sync"
$ hexo generate
$ hexo deploy
$ git add .
$ git commit -m "add blog"
$ git push origin source
```
这样就完成了多终端的博客同步。下面说一下第三方主题的同步问题

##### 第三方主题的同步问题

我们一般会选择第三方主题的仓库直接git clone下来。这是一个非常不好的习惯，正确做法是：Fork该第三方主题仓库，这样就会在自己账号下生成一个同名的仓库，并对应一个url，我们应该git clone自己账号下的url。

这样做的原因是：我们很有可能在原来主题基础上做一些自定义的小改动，为了保持多终端的同步，我们需要将这些改动提交到远程仓库。而第三方仓库我们是无法直接push的。

这样就会出现git仓库的嵌套问题，我们通过git submodule来解决这个问题。

```bash
$ git submodule add git@github.com:change2hao/hexo-theme-next.git themes/next
```
我们修改主题后:

```bash
git commit -am "refine UI"
git push origin source
```
在另外一个终端上执行：

```bash
git submodule init // 这句很重要
git submodule update
```
这样就完成了第三方主题的同步。
