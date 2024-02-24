---
title: powershell安装配置内容
date: 2023-06-22 21:03:32
tags: PowerShell
category: Shell
---


## PowerShell 配置脚本

```ps
Import-Module DirColors
Set-Alias bb "busybox" 
Set-Alias vim "nvim"
Set-Alias vi "nvim"
Set-Alias open "explorer.exe"
Set-Alias which "gcm"
Set-Alias vscode "code"
Set-Alias ll "ls"
Set-Alias android "D:\Install\IDE\Android Studio\bin\studio64.exe"
Set-Alias yd "C:\Users\heping\Documents\WindowsPowerShell\yd.ps1"
function locate{
	if ($args[0].StartsWith("-")) {
		D:\Install\Manager\Everything\Everything.exe $args
	} else {
		D:\Install\Manager\Everything\Everything.exe -s $args[0]
	}
}

```

## 安装的软件

```shell
scoop install busybox
scoop install sudo
```


## 配置内容

+ busybox 软链接配置
  ```bat
   for %%i in (ar, arch, ash, awk, base64, basename, bash, bunzip2, bzcat, bzip2, cal, cat, chmod, cksum, ^
             clear, cmp, comm, cp, cpio, cut, date, dc, dd, df, diff, dirname, dos2unix, dpkg, dpkg-deb, du, echo, ed,^
             egrep, env, expand, expr, factor, false, fgrep, find, fold, fsync, ftpget, ftpput, getopt, grep, groups,^
             gunzip, gzip, hd, head, hexdump, iconv, id, ipcalc, kill, killall, less, link, ln, logname, ls, lzcat, lzma,^
             lzop, lzopcat, man, md5sum, mkdir, mktemp, mv, nc, nl, od, paste, patch, pgrep, pidof, pipe_progress, pkill,^
             printenv, printf, ps, pwd, readlink, realpath, reset, rev, rm, rmdir, rpm, rpm2cpio, sed, seq, sh, sha1sum,^
             sha256sum, sha3sum, sha512sum, shred, shuf, sleep, sort, split, ssl_client, stat, strings, su, sum, tac, tail,^
             tar, tee, test, timeout, touch, tr, true, truncate, ts, ttysize, uname, uncompress, unexpand, uniq, unix2dos,^
             unlink, unlzma, unlzop, unxz, unzip, usleep, uudecode, uuencode, vi, watch, wc, wget, which, whoami, whois,^
             xargs, xxd, xz, xzcat, yes, zcat)^
     do mklink C:\Windows\System32\%%i.exe C:\Windows\System32\busybox.exe
   for %%i in (ar, arch, ash, awk, base64, basename, bash, bunzip2, bzcat, bzip2, cal, cat, chmod, cksum, ^
                clear, cmp, comm, cp, cpio, cut, date, dc, dd, df, diff, dirname, dos2unix, dpkg, dpkg-deb, du, echo, ed,^
                egrep, env, expand, expr, factor, false, fgrep, find, fold, fsync, ftpget, ftpput, getopt, grep, groups,^
                gunzip, gzip, hd, head, hexdump, iconv, id, ipcalc, kill, killall, less, link, ln, logname, ls, lzcat, lzma,^
                lzop, lzopcat, man, md5sum, mkdir, mktemp, mv, nc, nl, od, paste, patch, pgrep, pidof, pipe_progress, pkill,^
                printenv, printf, ps, pwd, readlink, realpath, reset, rev, rm, rmdir, rpm, rpm2cpio, sed, seq, sh, sha1sum,^
                sha256sum, sha3sum, sha512sum, shred, shuf, sleep, sort, split, ssl_client, stat, strings, su, sum, tac, tail,^
                tar, tee, test, timeout, touch, tr, true, truncate, ts, ttysize, uname, uncompress, unexpand, uniq, unix2dos,^
                unlink, unlzma, unlzop, unxz, unzip, usleep, uudecode, uuencode, vi, watch, wc, wget, which, whoami, whois,^
                xargs, xxd, xz, xzcat, yes, zcat)^
   rm C:\Windows\System32\b%%i.exe
  ```
