```git
本地和源仓库修改了同一文件内容，git pull时报冲突，解决如下：

P@Duk1906 MINGW64 ~/Documents/GitHub/Duk1901 (master)
$ git pull
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From https://github.com/Duk1906/Duk1901
   f5d98d2..0f6f000  master     -> origin/master
error: Your local changes to the following files would be overwritten by merge:
        README.md
Please commit your changes or stash them before you merge.
Aborting
Updating f5d98d2..0f6f000

P@Duk1906 MINGW64 ~/Documents/GitHub/Duk1901 (master)
$ git add -A

P@Duk1906 MINGW64 ~/Documents/GitHub/Duk1901 (master)
$ git commit -m 'update'
[master 13934f2] update
 1 file changed, 3 insertions(+), 3 deletions(-)

P@Duk1906 MINGW64 ~/Documents/GitHub/Duk1901 (master)
$ git pull
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.

P@Duk1906 MINGW64 ~/Documents/GitHub/Duk1901 (master|MERGING)
$ ll
total 12
drwxr-xr-x 1 P 197121    0 3月   6 16:11 DB/
drwxr-xr-x 1 P 197121    0 3月   5 12:48 Linux/
drwxr-xr-x 1 P 197121    0 3月   6 15:10 Python/
-rw-r--r-- 1 P 197121 3823 3月   7 11:52 README.md
drwxr-xr-x 1 P 197121    0 3月   6 19:05 SthMark/
drwxr-xr-x 1 P 197121    0 3月   5 12:49 中间件/

P@Duk1906 MINGW64 ~/Documents/GitHub/Duk1901 (master|MERGING)
$ vim README.md     <<<-----------------修复冲突

P@Duk1906 MINGW64 ~/Documents/GitHub/Duk1901 (master|MERGING)
$ git add -A

P@Duk1906 MINGW64 ~/Documents/GitHub/Duk1901 (master|MERGING)
$ git commit -m 'fix conflict'
[master ded6512] fix conflict

P@Duk1906 MINGW64 ~/Documents/GitHub/Duk1901 (master)
$ git push
Logon failed, use ctrl+c to cancel basic credential prompt.
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 407 bytes | 407.00 KiB/s, done.
Total 4 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3), completed with 2 local objects.
To https://github.com/Duk1906/Duk1901.git
   0f6f000..ded6512  master -> master

P@Duk1906 MINGW64 ~/Documents/GitHub/Duk1901 (master)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean

P@Duk1906 MINGW64 ~/Documents/GitHub/Duk1901 (master)
$ git pull
Already up to date.
```

**git确实做的挺好的，不用刻意记操作，根据提示信息就可以很好地使用git **

