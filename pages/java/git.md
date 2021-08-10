# git

入门推荐 [廖雪峰git教程](https://www.liaoxuefeng.com/wiki/896043488029600)

## 规范

```
// 一、从 GitLab 上 clone 某项目
git clone <projectName>
// 二、创建你的本地分支
git checkout -b <yourName>
// 关联你的本地分支与GitLab上你的远程分支
git branch --set-upstream-to=origin/<yourName> yourName
// 三、进行开发，开发完成，提交
git add *
git commit -m "message"
// 四、推送到你的远程分支上，准备在staging环境中编译部署
git push origin
// 五、生成diff文件发起Coe Review
git fetch -a
git rebase origin/master
git diff -U1000 origin/master > tmp.diff

// 六、Code Review 通过后提交 Request merge
// 1. 合并你的多个 commit 为一个
git checkout <yourName>
git log // 查看你进行了多少次commit，次数假如为<num>
git rebase -i HEAD~<num>
// 然后在输出的日志里，第一个commit选择pick，后面的commit选择squash
// 合并所有的commit comments为一个总的comment
git log //确认合并commit是否成功
// 如果你的分支不是基于最新的master分支，那么还需要rebase到master分支上
// git fetch -a
// git rebase origin/master
// 有冲突的话修复冲突，然后 
// git add .
// git rebase --continue
// 然后整合提交的 Message 
mvn clean package 编译试试能否SUCCESS
git push（-f) origin <yourName> // 如果之前 Push 过，就force push 下
// 2. 在 GitLab 里，Create merge request，等待Owner同意merge请求
```


