---
date: 2015-10-06 01:49:12+00:00
layout: post
title: svn源码树快照生成工具
categories: 工具
tags: svn
---

项目的源码管理工具使用的是SVN，由于涉及的项目组众多，源码库功能模块分散在众多小的源码库中，最终的项目源码发布需要从各个源码库中抓取分支，然后版本发布。

这就给版本管理员带来了麻烦，发布版本的时候，需要从各个源码库抓取源码，然后进行编译，调试过程中出现问题，就在需要更新出问题的源码分支，甚至有可能会临时的修改源码中的某一些文件。

开发人员如果想要使用某一个版本的源码，一般把版本管理员发布的源码包下载下来，然后在进行Bug定位和跟踪，由于源码量比较大，所以造成了很大的不方便。

所以编写了下边的源码快照工具，有以下功能：


- 遍历所有的源码库，获取并记录所有的源码库当前版本号
- 记录当前源码库中，新修改提交的文件版本号。（svn 只更新提交了的个别文件）
- 记录当前临时修改，但没有提交的代码，并生成patch文件放到patches/目录。
- 对于无法生成patch的二进制文件，则拷贝到patches/目录。
- 生成snapshot.sh脚本，这个脚本包含了当前svn源码树的快照信息。


在任意的项目SVN源码树，把前边输出的snapshot.sh和patches/目录放到这个源码树的根目录，执行snapshot.就可以恢复到相应的快照状态。


```
#/bin/sh

### exec this scripts in root directory.
### the snapshot of the work copy will be saved to $OUT.

OUT=$(pwd)/snapshot.sh
PATCH_DIR=$(pwd)/patches
EAT=$(pwd)/eat
patch_idx=0
mime_file_idx=0

echo "### V3.02.20 svn snapshot scripts ###" >$OUT
echo "PATCH_DIR=\$(pwd)/patches" >>$OUT

echo -e "\ncheck_svn_tree()
{
f_temp=\$1
f_temp=\${f_temp%/*}
if ! test -d \$f_temp
then
	while( ! test -d  \${f_temp%/*})
	do
		f_last=\$f_temp
		f_temp=\${f_temp%/*}
		if [ \"\$f_last\" = \"\$f_temp\" ]
		then
			break
		fi
	done
	echo \"will update \$f_temp\"
	svn update \$f_temp
fi
}\n" >>$OUT

#Prepare eat program for svn diff.
touch $EAT
chmod a+x $EAT

rm $PATCH_DIR -fr
mkdir -p $PATCH_DIR

#Find svn repositories.
#svn_repos=$(find ./ -name '*.svn' -type d -print)
svn_repos="./ "
svn_repos+=$(svn status | grep ^Perf | cut -d\' -f 2)
for svn_repo in $svn_repos
do
	#svn_dir=${svn_repo%/*}
	svn_dir=$svn_repo
	cd $svn_dir
	#Get svn local copy's revision.
	REV=$(svn info . | sed -n '/Revision:/p' | awk '{print $2}')
	echo "cd $svn_dir"
	echo "cd $svn_dir" >>$OUT
	echo "echo Cleaning: $svn_dir" >>$OUT
	echo "svn st | grep '^?' | awk '{print $2}' | xargs -I{} rm -rf '{}'" >>$OUT
	echo "svn st --no-ignore | grep '^I' | awk '{print $2}' | xargs -I{} rm -rf '{}'" >>$OUT
	echo "echo Reverting: $svn_dir" >>$OUT
	echo "svn cleanup" >>$OUT
	echo "svn revert . -R" >>$OUT
	echo "svn update . -r $REV" >>$OUT
	echo "svn revert . -R" >>$OUT
	
	#Get svn local copy's changed files.
	M_LIST=$(svn diff -r BASE:COMMITTED --diff-cmd $EAT | sed -n '/Index:/p' | awk '{print $2}')
	for svn_file in $M_LIST
	do
		echo "  Changed File: "$svn_file
		#Get changed file's revision.
		file_rev=$(svn info $svn_file | sed -n '/Last Changed Rev:/p' | awk '{print $4}')
		echo "#----------" >>$OUT
		echo "svn revert $svn_file >/dev/null 2>&1" >>$OUT
		echo "check_svn_tree $svn_file" >>$OUT
		echo "svn update -r $file_rev $svn_file" >>$OUT
		echo "svn revert $svn_file" >>$OUT
	done
	
	#Clean again after revert.
	echo "svn st | grep '^?' | awk '{print $2}' | xargs -I{} rm -rf '{}'" >>$OUT
	echo "svn st --no-ignore | grep '^I' | awk '{print $2}' | xargs -I{} rm -rf '{}'" >>$OUT
	
	#Get temporary patches
	PATCH_FILES=$(svn diff . --diff-cmd $EAT | sed -n '/Index:/p' | awk '{print $2}')
	if [ ""x != "$PATCH_FILES"x ];then
		let patch_idx+=1
		echo "#==========" >>$OUT
		svn diff . >$PATCH_DIR/$patch_idx.patch
		echo "patch -p0 <\$PATCH_DIR/$patch_idx.patch" >>$OUT
	fi
	
	for mime_file in $PATCH_FILES
	do
		mime_type=$(svn pl $mime_file | sed -n '/svn:mime-type/p')
		if [ ""x != "$mime_type"x ];then
			let mime_file_idx+=1
			cp $mime_file $PATCH_DIR/${mime_file_idx}_${mime_file##*/} -fr
			echo "#==========" >>$OUT
			echo "cp \$PATCH_DIR/${mime_file_idx}_${mime_file##*/} $mime_file -f" >>$OUT
		fi
	done
	
	echo "cd ->/dev/null" >> $OUT
	echo "" >>$OUT
	echo "###############">> $OUT
	echo "" >>$OUT
	cd - > /dev/null
done
```
