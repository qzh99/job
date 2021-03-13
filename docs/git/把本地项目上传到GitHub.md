### 1、创建项目

### 2、初始化项目

`git init`

### 3、添加文件到暂存区

+ 单个文件`git add file1` 
+ 多个文件`git add file1 file2 file3` 

+ 全部`git add .`或`git add -A`

### 4、commit到本地

`git commit -m'message'`

### 5、在GitHub上创建远程项目

在Github上创建和本地工程同名的项目

### 6、在本地设置远程仓库地址

`git remote add origin https://github.com/qzh99/JavaCore.git `

### 7、先拉取远程代码

`git pull origin master --allow-unrelated-histories`

### 8、推送本地代码到远程

+ 首次推送时使用`-u` 参数创建upStream 上传流，后续就不用 `git push -u origin master`
+ 后续推送直接`git push`