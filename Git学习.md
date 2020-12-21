# Git学习

# 1、常用命令

### 1.1、获得版本库

- git	init

将当前文件夹初始化为一个git仓库

- git	clone

从远程仓库克隆项目到本地



### 1.2、版本管理

- git	add

将修改的文件添加到暂存区



- git commit

将缓存中文件提交到资源库中

- git	rm

移除



### 1.3、查看信息

- git	help

查看帮助



- git	log

查看提交记录



- git	diff

查看区别



### 1.4、远程协作

- git	pull



- git	push

### 1.5、配置

```linux
/etc/gitconfig     --system

~/.gitconfig        --global

.git/config          --local
--例子
git	config	--global	user.name	"gaokang"
```

