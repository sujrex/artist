#windows下使用git
1. 安装[git](https://git-scm.com/download/win)选择非"thumbdrive edition(绿色版)"版本
2. 打开git bash
3. 创建工作目录
```shell
   cd e:/
   mkdir gitworkspace & cd gitworkspace
   git init //初始化git repository
```
3. 自报家门 --global表示这台机器上所有的repository都会使用这个配置
```shell
   git config --global user.email '1584066465@qq.com'
   git config --global user.name 'xyg'
 ```
5. 新增文件
```shell
   git add test.txt [test1.txt [test2.txt]]
   git commit -m '文件变动描述'
```
6. 修改并提交
```shell
   git status //查看改动状态
   git diff test.txt //查看test.txt有哪些变动，类似beyondcompare
   git add test.txt
   git commit -m "文件改动描述"
   git status
```
7. 回退版本
```shell
   git log [--pretty=oneline] //第一列是sha1计算的版本号
   git reset -hard HEAD^ //一个^表示上一个版本，2个依次类推。多个的话可以使用 HEAD~100 表示回退到100个之前的版本
```
8. 恢复版本，一般来说是无法恢复的，除非还能看到回退之前的 git log
```shell
   git reset -hard xxx //这里的xxx就是指 git log得到的第一列 也就是版本号
   如果不巧，git log记录已经被刷掉，或者窗口被关掉，怎么办？
   git reflog //后悔药，可以看到过往的操作历史记录，当然也就有commit id了
```
9. 撤销修改
```shell
   git checkout -- test.txt // -- 很重要，如果没有，则命令变为“切换到另一个分支”
```
10. 退回暂存区修改
```shell
   git reset HEAD test.txt //如果修改已经被 git add到暂存区，又不想提及了，可以用此命令回退
```
11. 删除文件
```shell
   git rm test.txt
   git rm files -fr //删除文件夹以及其下面的文件
```
12. 恢复被误删的文件
```shell
   git checkout -- test.txt //如果你是在文件管理器中不小心删除了test.txt文件，想要恢复，就用此命令
```
13. 连接github
```shell
   ssh-keygen -t rsa -C '1584066465@qq.com' 
```
> 生成秘钥文件，一路回车，无需输入密码，因为不太重要
> 完毕之后可以在 c:/user/sujrex/.ssh/
> 注册github, Settings->SSH and GPG keys->New SSH Key->输入上述文件夹中的id_rsa.pub中的内容，点击保存
> 公司一个，家里一个，可以重复执行上面的步骤，建立多个只属于自己的通道

14. 创建在线仓库
> +->New repository->填写"Repository Name" and "description" and chose "public" and tick "Initialize ..." after "Create Repository"
15. 添加远程库
> 如果本地已经又git库了，现在要同步到github上,在github上建立和本地库一样的名字，使用
```shell
   git remote add origin git@github.com:sujrex/artist.git
   git push -u origin master //第一次提交
   git push origin master //后续提交
```
16. 从github克隆
```shell
   git clone git@github.com:sujrex/artist.git
```
17. 创建分支
```shell
   git checkout -b dev //创建dev分支，并切换到dev分支 -b就是一下两条命令的合集 git branch dev + git checkout dev 
   git branch //查看分支 带*号的表示当前再用分支
   git checkout master //切换回master分支
```
18. 标签新增
```shell
   git branch master //切换到要打tag的分支上
   git tag v1.0 //这样就默认打到了最新的commit上
   git tag v1.0 00fsas //打到固定的那个commit上，后面的commit id可以通过 git log --pretty=oneline --abbrev-commit查看
   git tag -a v1.0 -m 'release v1.0 about miyou' //给打的tag带上说明
   git tag //查看所有tag
   git show v1.0 //查看标签信息
```
19. 操作标签
```shell
   git tag -d v1.0 //删除某个标签
   git push origin v1.0 //将某个标签推送到github上
   git push origin --tags //一次性将本地的tag全部推送到github上
```
> 删除远程tag
```shell
   git tag -d v1.0 //先删除本地tag
   git push origin :refs/tags/v1.0 //删除远程tag
```
20. 参与github上开源项目
> 在自己账号上fork将要参与的项目
```shell
   git clone git@github.com:sujrex/think.git
```
> 修改代码，增加特性
```shell
   git push think master
```
> 如果想要将自己的修改推送到开源项目上，在github上 new pull request.对方接受你的pull之后，就算合成了
21. 同步使用码云
```shell
   git remote rm origin //删除已关联的远程库
   git remote add github git@github.com:sujrex/artist.git
   git remote add gitee git@gitee.com:sujrex/artist.git
   git push github master //推送修改到github
   git push gitee master //推送修改到gitee
```
22. 忽略需要提交的文件
> 编辑.gitignore文件,内容如下：
> #windows Thumbs.db
> #secret config file
> config.php
> 编辑完成后，将文件提交到git，那么后面相关的文件就不会被提交，git status时就不会列出 Untracked files ...解救强迫症的你
```shell
   git add -f config.php //可以忽略.gitignore的配置强制提交
   git check-ignore -v config.php //查看config.php为甚被忽略，找到此规则
```
23. 一个有用的别名配置
```shell
   git config --global alias.lg "log --color --graph--pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
   git lg //就可以调用上面的一长串命令了
```
> 配置文件是放在 .gitconfig中[C:\Users\sujrex]，也可以直接编辑该文件修改相关配置
24. 配置diff和merge 使用beyondcompare
```shell
   #编辑.gitconfig加入
   tool = bc4
   [difftool]
   prompt = true
   [difftool "bc4"]
   cmd = \"D:/Program Files/Beyond Compare 4/BCompare.exe\" "$(cygpath -w $LOCAL)" "$REMOTE"
   [merge]
   tool = bc4
   [mergetool]
   prompt = true
   [mergetool "bc4"]
   #trustExitCode = true
   cmd = \"D:/Program Files/Beyond Compare 4/BCompare.exe\" "$LOCAL" "$REMOTE" "$BASE" "$MERGED"
   #保存
   git difftool test.txt //查看test.txt文件变动
   git mergetool test.txt //合并test.txt文件冲突
   #合并冲突文件时，git会自动创建xx.orig文件，如果不需要刻意通过修改配置禁止该文件自动产生
   git --global mergetool.keepBackup false
```
25. 重命名
```shell
   git mv oldname newname
```
26. 中文乱码
```shell
   git config --global core.quotepath false
```
> 在git bash的界面中右击空白处，弹出菜单，选择选项->文本->本地Locale，设置为zh_CN，而旁边的字符集选框选为UTF-8