---
title: ag-命令行查找工具
tags: [Linux,ag]
categories: [Linux]
date: 2020-09-02 17:30:00
---

# ag-命令行查找工具

>安装：
>
>```shell
>sudo apt-get install silversearcher-ag
>```

```shell
Usage: ag [FILE-TYPE] [OPTIONS] PATTERN [PATH]

  Recursively search for PATTERN in PATH. # 递归查询
  Like grep or ack, but faster. # 比grep ack更快

Example:
  ag -i foo /bar/

Output Options:
     --ackmate            Print results in AckMate-parseable format # 以AckMate可以解析的格式输出
  -A --after [LINES]      Print lines after match (Default: 2) # 匹配到后 打印匹配到的行的后几行  默认2行
  -B --before [LINES]     Print lines before match (Default: 2)# 匹配到后 打印匹配到的行的前几行  默认2行
     --[no]break          Print newlines between matches in different files
                          (Enabled by default) # 不同文件匹配到后是否打印空行 默认启用
  -c --count              Only print the number of matches in each file. # 打印不同文件匹配数 
                          (This often differs from the number of matching lines) # 匹配数 不是 匹配行数
     --[no]color          Print color codes in results (Enabled by default) # 彩色打印匹配结果 默认开启
     --color-line-number  Color codes for line numbers (Default: 1;33) # 行号颜色 默认1;33
     --color-match        Color codes for result match numbers (Default: 30;43) # 匹配结果颜色 默认30;43
     --color-path         Color codes for path names (Default: 1;32) #路径颜色默认 1;32
     --column             Print column numbers in results # 打印列号
     --[no]filename       Print file names (Enabled unless searching a single file)# 打印文件名 默认搜索多个文件时候开启
  -H --[no]heading        Print file names before each file's matches #每个文件搜索前都打印文件名
                          (Enabled by default) # 默认开启
  -C --context [LINES]    Print lines before and after matches (Default: 2)# 打印匹配到的结果行的前后几行 默认为2
     --[no]group          Same as --[no]break --[no]heading # 类似 --[no]break --[no]heading
  -g --filename-pattern PATTERN
                          Print filenames matching PATTERN # 搜索含有PATTERN的文件名
  -l --files-with-matches Only print filenames that contain matches
                          (don't print the matching lines) # 只打印匹配到的文件名 不会打印具体匹配到的行
  -L --files-without-matches
                          Only print filenames that don't contain matches # 只打印未匹配到的文件名
     --print-all-files    Print headings for all files searched, even those that
                          don't contain matches # 匹配结果前打印文件名 没有匹配到的文件也打印
     --[no]numbers        Print line numbers. Default is to omit line numbers
                          when searching streams # 打印行号 默认忽略？？？
  -o --only-matching      Prints only the matching part of the lines # 只打印匹配的部分
     --print-long-lines   Print matches on very long lines (Default: >2k characters)
     --passthrough        When searching a stream, print all lines even if they
                          don't match
     --silent             Suppress all log messages, including errors
     --stats              Print stats (files scanned, time taken, etc.) # 打印状态
     --stats-only         Print stats and nothing else. # 只打印状态
                          (Same as --count when searching a single file) 
     --vimgrep            Print results like vim's :vimgrep /pattern/g would
                          (it reports every match on the line) # 打印格式和vimgrep一样
  -0 --null --print0      Separate filenames with null (for 'xargs -0')

Search Options:
  -a --all-types          Search all files (doesn't include hidden files
                          or patterns from ignore files) # 查询所有文件 不包括隐藏文件和忽略的文件
  -D --debug              Ridiculous debugging (probably not useful) # 会输出debug信息
     --depth NUM          Search up to NUM directories deep (Default: 25)# 递归深度
  -f --follow             Follow symlinks # 遵循符号链接
  -F --fixed-strings      Alias for --literal for compatibility with grep # --literal 别名 与grep兼容
  -G --file-search-regex  PATTERN Limit search to filenames matching PATTERN # 限制搜索范围为 文件名含有PATTERN的文件 
     --hidden             Search hidden files (obeys .*ignore files) # 搜索包含隐藏文件（.*）
  -i --ignore-case        Match case insensitively # 不区分大小写
     --ignore PATTERN     Ignore files/directories matching PATTERN # 忽略含有PATTERN的文件或文件夹
                          (literal file/directory names also allowed)
     --ignore-dir NAME    Alias for --ignore for compatibility with ack.# --ignore别名 与ack兼容
  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
     --one-device         Don't follow links to other devices. # 不遵循链接到其他驱动 只查当前驱动文件
  -p --path-to-ignore STRING
                          Use .ignore file at STRING
  -Q --literal            Don't parse PATTERN as a regular expression # 不再将pattern解析为正则表达式
  -s --case-sensitive     Match case sensitively # 区分大小写
  -S --smart-case         Match case insensitively unless PATTERN contains 
                          uppercase characters (Enabled by default)# 除非PATTERN包含大写字母 否则不区分大小写 默认启用
     --search-binary      Search binary files for matches # 二进制文件中匹配
  -t --all-text           Search all text files (doesn't include hidden files) #搜索所有文本文件（不包括隐藏文件）
  -u --unrestricted       Search all files (ignore .ignore, .gitignore, etc.;
                          searches binary and hidden files as well)# 查询所有文件
  -U --skip-vcs-ignores   Ignore VCS ignore files
                          (.gitignore, .hgignore; still obey .ignore)# 忽略vcs的忽略文件
  -v --invert-match       # 反向匹配
  -w --word-regexp        Only match whole words #整个单词匹配
  -W --width NUM          Truncate match lines after NUM characters # 匹配的行信息在num个字符后截断
  -z --search-zip         Search contents of compressed (e.g., gzip) files #压缩文件中匹配

File Types:
The search can be restricted to certain types of files. Example:
  ag --html needle #指定搜索的文件类型为html
  - Searches for 'needle' in files with suffix .htm, .html, .shtml or .xhtml.

For a list of supported file types run:
  ag --list-file-types # 查看ag支持的文件类型

ag was originally created by Geoff Greer. More information (and the latest release)
can be found at http://geoff.greer.fm/ag
```

* 测试文件

  ```shell
  ~/agtest ❯   ag --help > text   28s 06:38:37 PM
  ~/agtest ❯   man ls > lstest   28s 06:39:37 PM
  ```

## 输出参数

### -A --after [LINES]      

Print lines after match (Default: 2) 

匹配到后 打印匹配到的行的后几行  默认2行

```shell
~ ❯ ag -A after ./agtest                                                                09:18:35 AM
agtest/text #文件路径 文件名
12:  -A --after [LINES]      Print lines after match (Default: 2)
13-  -B --before [LINES]     Print lines before match (Default: 2)
14-     --[no]break          Print newlines between matches in different files
--
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
27-     --[no]group          Same as --[no]break --[no]heading
28-  -g --filename-pattern PATTERN
--
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
64-     --one-device         Don't follow links to other devices.
65-  -p --path-to-ignore STRING
--
79:  -W --width NUM          Truncate match lines after NUM characters
80-  -z --search-zip         Search contents of compressed (e.g., gzip) files
81-
```

```shell
~ ❯ ag -A 3 after ./agtest                                                              09:18:48 AM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
13-  -B --before [LINES]     Print lines before match (Default: 2)
14-     --[no]break          Print newlines between matches in different files
15-                          (Enabled by default)
--
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
27-     --[no]group          Same as --[no]break --[no]heading
28-  -g --filename-pattern PATTERN
29-                          Print filenames matching PATTERN
--
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
64-     --one-device         Don't follow links to other devices.
65-  -p --path-to-ignore STRING
66-                          Use .ignore file at STRING
--
79:  -W --width NUM          Truncate match lines after NUM characters
80-  -z --search-zip         Search contents of compressed (e.g., gzip) files
81-
82-File Types:
```

###  -B --before [LINES] 

Print lines before match (Default: 2)

匹配到后 打印匹配到的行的前几行  默认2行

```shell
~ ❯ ag -B after ./agtest                                                                09:38:41 AM
agtest/text
10-Output Options:
11-     --ackmate            Print results in AckMate-parseable format
12:  -A --after [LINES]      Print lines after match (Default: 2)
--
24-  -H --[no]heading        Print file names before each file's matches
25-                          (Enabled by default)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
--
61-                          (literal file/directory names also allowed)
62-     --ignore-dir NAME    Alias for --ignore for compatibility with ack.
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
--
77-  -v --invert-match
78-  -w --word-regexp        Only match whole words
79:  -W --width NUM          Truncate match lines after NUM characters
```

```shell
~ ❯ ag -B 3 after ./agtest                                                              09:38:56 AM
agtest/text
9-
10-Output Options:
11-     --ackmate            Print results in AckMate-parseable format
12:  -A --after [LINES]      Print lines after match (Default: 2)
--
23-     --[no]filename       Print file names (Enabled unless searching a single file)
24-  -H --[no]heading        Print file names before each file's matches
25-                          (Enabled by default)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
--
60-     --ignore PATTERN     Ignore files/directories matching PATTERN
61-                          (literal file/directory names also allowed)
62-     --ignore-dir NAME    Alias for --ignore for compatibility with ack.
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
--
76-                          (.gitignore, .hgignore; still obey .ignore)
77-  -v --invert-match
78-  -w --word-regexp        Only match whole words
79:  -W --width NUM          Truncate match lines after NUM characters
```

### --[no]break         

 Print newlines between matches in different files  (Enabled by default)

不同文件匹配到结果之间是否打印空行 默认开启

```shell
~ ❯ ag --break do ./agtest                                                              09:48:35 AM
agtest/lstest
16:              do not ignore entries starting with .
19:              do not list implied . and ..
32:              do not list implied entries ending with ~
50:       -f     do not sort, enable -aU, disable -ls --color
56:              likewise, except do not append '*'
65:       -g     like -l, but do not list owner
74:              in a long listing, don't print group names
90:              do not list implied entries matching shell PATTERN (overridden by -a or -A)
103:              do not list implied entries matching shell PATTERN
123:       -o     like -l, but do not list group information
136:              enclose entry names in double quotes
174:       -U     do not sort; list entries in directory order
230:       Full documentation at: <https://www.gnu.org/software/coreutils/ls>
# 空行
agtest/text
31:                          (don't print the matching lines)
33:                          Only print filenames that don't contain matches
35:                          don't contain matches
41:                          don't match
51:  -a --all-types          Search all files (doesn't include hidden files
64:     --one-device         Don't follow links to other devices.
67:  -Q --literal            Don't parse PATTERN as a regular expression
72:  -t --all-text           Search all text files (doesn't include hidden files)
```

```shell
~ ❯  ag --nobreak do ./agtest                                                           09:48:46 AM
agtest/text
31:                          (don't print the matching lines)
33:                          Only print filenames that don't contain matches
35:                          don't contain matches
41:                          don't match
51:  -a --all-types          Search all files (doesn't include hidden files
64:     --one-device         Don't follow links to other devices.
67:  -Q --literal            Don't parse PATTERN as a regular expression
72:  -t --all-text           Search all text files (doesn't include hidden files)
agtest/lstest
16:              do not ignore entries starting with .
19:              do not list implied . and ..
32:              do not list implied entries ending with ~
50:       -f     do not sort, enable -aU, disable -ls --color
56:              likewise, except do not append '*'
65:       -g     like -l, but do not list owner
74:              in a long listing, don't print group names
90:              do not list implied entries matching shell PATTERN (overridden by -a or -A)
103:              do not list implied entries matching shell PATTERN
123:       -o     like -l, but do not list group information
136:              enclose entry names in double quotes
174:       -U     do not sort; list entries in directory order
230:       Full documentation at: <https://www.gnu.org/software/coreutils/ls>
```

### -c --count              

Only print the number of matches in each file. (This often differs from the number of matching lines)

打印不同文件中匹配数  匹配数不是匹配行数

```shell
~ ❯ ag do ./agtest                                                                      09:51:54 AM
agtest/lstest
16:              do not ignore entries starting with .
19:              do not list implied . and ..
32:              do not list implied entries ending with ~
50:       -f     do not sort, enable -aU, disable -ls --color
56:              likewise, except do not append '*'
65:       -g     like -l, but do not list owner
74:              in a long listing, don't print group names
90:              do not list implied entries matching shell PATTERN (overridden by -a or -A)
103:              do not list implied entries matching shell PATTERN
123:       -o     like -l, but do not list group information
136:              enclose entry names in double quotes
174:       -U     do not sort; list entries in directory order
230:       Full documentation at: <https://www.gnu.org/software/coreutils/ls>

agtest/text
31:                          (don't print the matching lines)
33:                          Only print filenames that don't contain matches
35:                          don't contain matches
41:                          don't match
51:  -a --all-types          Search all files (doesn't include hidden files
64:     --one-device         Don't follow links to other devices.
67:  -Q --literal            Don't parse PATTERN as a regular expression
72:  -t --all-text           Search all text files (doesn't include hidden files)
~ ❯ ag -c do ./agtest                                                                   09:55:54 AM
agtest/text:8
agtest/lstest:13
```

### --[no]color         

 Print color codes in results (Enabled by default) 

彩色打印匹配结果 默认开启

![image-20200903100033965](/img/linux/ag/image-20200903100033965.png)

* `--color-line-number`  

  Color codes for line numbers (Default: 1;33) 

  行号颜色 默认1;33

* `--color-match`    

  Color codes for result match numbers (Default: 30;43) 

  匹配结果颜色 默认30;43

* `--color-path`  

  Color codes for path names (Default: 1;32) 

  路径颜色默认 1;32

![image-20200903103836432](/img/linux/ag/image-20200903103836432.png)

![image-20200903104026933](/img/linux/ag/image-20200903104026933.png)

### --column             

Print column numbers in results

打印列号

```shell
~ ❯ ag --column after ./agtest                                                          10:40:00 AM
agtest/text
12:8:  -A --after [LINES]      Print lines after match (Default: 2)
26:50:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:51:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:48:  -W --width NUM          Truncate match lines after NUM characters
```

### --[no]filename       

Print file names (Enabled unless searching a single file)

打印文件名 默认多个文件时候开启

```shell
~ ❯ ag after ./agtest                                                                   10:46:56 AM
agtest/text # 文件夹搜索 默认打印文件名
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag after ./agtest/text    #单个文件搜索默认不打印文件名                                  10:47:02 AM
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters                                     
```

```shell
~ ❯ ag --filename after  ./agtest/text                                                  10:50:11 AM
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag --filename after  ./agtest                                                       10:50:21 AM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag --nofilename after  ./agtest                                                     10:50:33 AM
  -A --after [LINES]      Print lines after match (Default: 2)
  -C --context [LINES]    Print lines before and after matches (Default: 2)
  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
  -W --width NUM          Truncate match lines after NUM characters
```

> --filename 无法改变默认的单个文件搜索规则 使其显示文件名 如果要想显示单个文件文件名 则查看 -H 参数
>
> --nofilename可以使多个文件搜索时候 不再显示文件名

### -H --[no]heading        

Print file names before each file's matches  (Enabled by default)
每个文件搜索前都打印文件名  默认开启

```shell
~ ❯ ag -H after  ./agtest/text                                                          10:50:38 AM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag -H after  ./agtest                                                               10:52:00 AM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag --noheading after  ./agtest                                                      10:56:47 AM
agtest/text:12:  -A --after [LINES]      Print lines after match (Default: 2)
agtest/text:26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
agtest/text:63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
agtest/text:79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag --noheading after  ./agtest/text                                                 10:56:59 AM
agtest/text:12:  -A --after [LINES]      Print lines after match (Default: 2)
agtest/text:26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
agtest/text:63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
agtest/text:79:  -W --width NUM          Truncate match lines after NUM characters
```

### -C --context [LINES]    

Print lines before and after matches (Default: 2)

打印匹配到的结果行的前后几行 默认为2

```shell
~ ❯ ag -C after ./agtest                                                                11:03:15 AM
agtest/text
10-Output Options:
11-     --ackmate            Print results in AckMate-parseable format
12:  -A --after [LINES]      Print lines after match (Default: 2)
13-  -B --before [LINES]     Print lines before match (Default: 2)
14-     --[no]break          Print newlines between matches in different files
--
24-  -H --[no]heading        Print file names before each file's matches
25-                          (Enabled by default)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
27-     --[no]group          Same as --[no]break --[no]heading
28-  -g --filename-pattern PATTERN
--
61-                          (literal file/directory names also allowed)
62-     --ignore-dir NAME    Alias for --ignore for compatibility with ack.
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
64-     --one-device         Don't follow links to other devices.
65-  -p --path-to-ignore STRING
--
77-  -v --invert-match
78-  -w --word-regexp        Only match whole words
79:  -W --width NUM          Truncate match lines after NUM characters
80-  -z --search-zip         Search contents of compressed (e.g., gzip) files
81-
~ ❯ ag --context=1 after ./agtest                                                       11:03:25 AM
agtest/text
11-     --ackmate            Print results in AckMate-parseable format
12:  -A --after [LINES]      Print lines after match (Default: 2)
13-  -B --before [LINES]     Print lines before match (Default: 2)
--
25-                          (Enabled by default)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
27-     --[no]group          Same as --[no]break --[no]heading
--
62-     --ignore-dir NAME    Alias for --ignore for compatibility with ack.
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
64-     --one-device         Don't follow links to other devices.
--
78-  -w --word-regexp        Only match whole words
79:  -W --width NUM          Truncate match lines after NUM characters
80-  -z --search-zip         Search contents of compressed (e.g., gzip) files
```

### --[no]group          

Same as --[no]break --[no]heading 

类似 --[no]break --[no]heading

```shell
~ ❯ ag --group after ./agtest                                                           11:03:28 AM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag --nogroup after ./agtest                                                         11:04:43 AM
agtest/text:12:  -A --after [LINES]      Print lines after match (Default: 2)
agtest/text:26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
agtest/text:63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
agtest/text:79:  -W --width NUM          Truncate match lines after NUM characters
```

### -g --filename-pattern PATTERN
Print filenames matching PATTERN

搜索含有PATTERN的文件名

~~~shell
~ ❯ ag -g test ./agtest                                                                 11:11:44 AM
agtest/text
agtest/lstest
~ ❯ ag -g text ./agtest                                                                 11:11:58 AM
agtest/text
~~~

> 匹配的是路径中是否含有PATTERN

### -l --files-with-matches 

Only print filenames that contain matches (don't print the matching lines) 

只打印匹配到的文件名 不会打印具体匹配到的行

~~~shell
~ ❯ ag -l after ./agtest                                                                11:12:08 AM
agtest/text
~~~

### -L --files-without-matches
Only print filenames that don't contain matches 

只打印未匹配到的文件名

~~~shell
~ ❯ ag -L after ./agtest                                                                11:16:29 AM
agtest/lstest
~~~

###  --print-all-files    

Print headings for all files searched, even those that don't contain matches 

 匹配结果前打印文件名 没有匹配到的文件也打印

```shell
~ ❯ ag --print-all-files after ./agtest                                                 11:17:32 AM
agtest/lstest

agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
```

### --[no]numbers        

Print line numbers. Default is to omit line numbers when searching streams

打印行号 默认忽略？？？

```shell
~ ❯ ag after ./agtest                                                                   11:30:39 AM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag --number after ./agtest                                                          11:31:27 AM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag --nonumber after ./agtest                                                        11:31:33 AM
agtest/text
  -A --after [LINES]      Print lines after match (Default: 2)
  -C --context [LINES]    Print lines before and after matches (Default: 2)
  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
  -W --width NUM          Truncate match lines after NUM characters
```

### -o --only-matching      

Prints only the matching part of the lines 

只打印匹配的部分

```shell
~ ❯ ag after ./agtest                                                                   11:31:40 AM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag -o after ./agtest                                                                11:42:03 AM
agtest/text
12:after
12:after
26:after
63:after
79:after
```

### --print-long-lines   

Print matches on very long lines (Default: >2k characters)

```shell
~ ❯ ag --print-long-lines af ./agtest                                                   11:47:15 AM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
```

### --stats              

Print stats (files scanned, time taken, etc.)

打印状态

```shell
~ ❯ ag --stats after ./agtest                                                           01:57:52 PM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
5 matches
1 files contained matches
2 files searched
13025 bytes searched
0.000971 seconds
```

### --stats-only         

Print stats and nothing else. 

只打印状态

```shell
~ ❯ ag --stats-only after ./agtest                                                      01:59:22 PM
5 matches
1 files contained matches
2 files searched
13025 bytes searched
0.001526 seconds
```

### --vimgrep            

Print results like vim's :vimgrep /pattern/g would  (it reports every match on the line) 

打印格式和vimgrep一样

~~~shell
~ ❯ ag --vimgrep after ./agtest                                                         01:59:57 PM
agtest/text:12:8:  -A --after [LINES]      Print lines after match (Default: 2)
agtest/text:12:39:  -A --after [LINES]      Print lines after match (Default: 2)
agtest/text:26:50:  -C --context [LINES]    Print lines before and after matches (Default: 2)
agtest/text:63:51:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
agtest/text:79:48:  -W --width NUM          Truncate match lines after NUM characters
~~~

## 查询参数

###  -a --all-types          

Search all files (doesn't include hidden files or patterns from ignore files) 

查询所有文件 不包括隐藏文件或 忽略文件模式

```shell
~/agtest ❯ ls -la                                                                       01:37:55 PM
total 28
drwxr-xr-x  2 chen chen 4096 Sep  4 09:40 .
drwxr-xr-x 19 chen chen 4096 Sep  4 13:37 ..
-rw-r--r--  1 chen chen 2990 Sep  4 09:39 .mvtest
-rw-r--r--  1 chen chen 8075 Sep  3 09:47 lstest
-rw-r--r--  1 chen chen 4950 Sep  2 18:42 text
~ ❯ ag -a name ./agtest                                                                 01:38:20 PM
agtest/text
21:     --color-path         Color codes for path names (Default: 1;32)
23:     --[no]filename       Print file names (Enabled unless searching a single file)
24:  -H --[no]heading        Print file names before each file's matches
28:  -g --filename-pattern PATTERN
29:                          Print filenames matching PATTERN
30:  -l --files-with-matches Only print filenames that contain matches
33:                          Only print filenames that don't contain matches
48:  -0 --null --print0      Separate filenames with null (for 'xargs -0')
57:  -G --file-search-regex  PATTERN Limit search to filenames matching PATTERN
61:                          (literal file/directory names also allowed)
62:     --ignore-dir NAME    Alias for --ignore for compatibility with ack.

agtest/lstest
3:NAME
35:              mation); with -l: show ctime and sort by name; otherwise:  sort  by  ctime,  newest
74:              in a long listing, don't print group names
93:              hyperlink file names; WHEN can be 'always' (default if omitted), 'auto', or 'never'
96:              append indicator with style WORD  to  entry  names:  none  (default),  slash  (-p),
121:              print entry names without quoting
135:       -Q, --quote-name
136:              enclose entry names in double quotes
139:              use quoting style WORD for  entry  names:  literal,  locale,  shell,  shell-always,
155:              sort by WORD instead of name: none (-U), size (-S), time (-t), version (-v), exten‐
172:              name; otherwise: sort by access time, newest first
~ ❯ ag name ./agtest                                                                    01:38:28 PM
agtest/text
21:     --color-path         Color codes for path names (Default: 1;32)
23:     --[no]filename       Print file names (Enabled unless searching a single file)
24:  -H --[no]heading        Print file names before each file's matches
28:  -g --filename-pattern PATTERN
29:                          Print filenames matching PATTERN
30:  -l --files-with-matches Only print filenames that contain matches
33:                          Only print filenames that don't contain matches
48:  -0 --null --print0      Separate filenames with null (for 'xargs -0')
57:  -G --file-search-regex  PATTERN Limit search to filenames matching PATTERN
61:                          (literal file/directory names also allowed)
62:     --ignore-dir NAME    Alias for --ignore for compatibility with ack.

agtest/lstest
3:NAME
35:              mation); with -l: show ctime and sort by name; otherwise:  sort  by  ctime,  newest
74:              in a long listing, don't print group names
93:              hyperlink file names; WHEN can be 'always' (default if omitted), 'auto', or 'never'
96:              append indicator with style WORD  to  entry  names:  none  (default),  slash  (-p),
121:              print entry names without quoting
135:       -Q, --quote-name
136:              enclose entry names in double quotes
139:              use quoting style WORD for  entry  names:  literal,  locale,  shell,  shell-always,
155:              sort by WORD instead of name: none (-U), size (-S), time (-t), version (-v), exten‐
172:              name; otherwise: sort by access time, newest first
```

### -D --debug              

Ridiculous debugging (probably not useful) 

会输出debug信息

~~~shell
~ ❯ ag -D after ./agtest                                                                01:40:33 PM
DEBUG: Found user's home dir: /home/chen
DEBUG: Skipping ignore file /home/chen/.agignore: not readable
DEBUG: global core.excludesfile: /home/chen/.config/git/ignore
DEBUG: Skipping ignore file /home/chen/.config/git/ignore: not readable
DEBUG: Query is after
DEBUG: PCRE Version: 8.39 2016-06-14
DEBUG: Using 7 workers
DEBUG: Worker 0 started
DEBUG: Thread 0 set to CPU 0
DEBUG: Thread 1 set to CPU 1
DEBUG: Thread 2 set to CPU 2
DEBUG: Worker 2 started
DEBUG: Worker 1 started
DEBUG: Thread 3 set to CPU 3
DEBUG: Worker 3 started
DEBUG: Thread 4 set to CPU 4
DEBUG: Worker 4 started
DEBUG: Thread 5 set to CPU 5
DEBUG: Worker 5 started
DEBUG: Thread 6 set to CPU 6
DEBUG: searching path ./agtest for after
DEBUG: Skipping ignore file ./agtest/.ignore: not readable
DEBUG: Skipping ignore file ./agtest/.gitignore: not readable
DEBUG: Skipping ignore file ./agtest/.git/info/exclude: not readable
DEBUG: Skipping ignore file ./agtest/.hgignore: not readable
DEBUG: search_dir: path is './agtest', base_path is '/home/chen/agtest/', path_start is './agtest'
DEBUG: text not ignored
DEBUG: lstest not ignored
DEBUG: ./agtest/text added to work queue
DEBUG: ./agtest/lstest added to work queue
DEBUG: Match found. File ./agtest/text, offset 243 bytes.
DEBUG: Worker 6 started
DEBUG: Worker 6 finished.
DEBUG: Match found. File ./agtest/text, offset 274 bytes.
DEBUG: Worker 5 finished.
DEBUG: Match found. File ./agtest/text, offset 1244 bytes.
DEBUG: No match in ./agtest/lstest
DEBUG: Worker 4 finished.
DEBUG: Worker 1 finished.
DEBUG: Match found. File ./agtest/text, offset 3527 bytes.
DEBUG: Match found. File ./agtest/text, offset 4489 bytes.
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
DEBUG: Worker 0 finished.
DEBUG: Worker 3 finished.
DEBUG: Worker 2 finished.
~~~

### --depth NUM          

Search up to NUM directories deep (Default: 25)

递归深度 默认25

~~~shell
~ ❯ ag --depth=2 after                                                                  02:03:01 PM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters

agtest/depth/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag --depth=1 after                                                                  02:03:12 PM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~~~

### -f --follow             

Follow symlinks  

遵循符号链接

> **符号链接**（**软链接、Symbolic link**）是一类特殊的[文件](https://zh.wikipedia.org/wiki/计算机文件)， 其包含有一条以绝对[路径](https://zh.wikipedia.org/wiki/路径_(计算机科学))或者相对路径的形式指向其它文件或者目录的引用。[[1\]](https://zh.wikipedia.org/wiki/符号链接#cite_note-1) 符号链接最早在4.2[BSD](https://zh.wikipedia.org/wiki/BSD)版本中出现（1983年）。今天[POSIX](https://zh.wikipedia.org/wiki/POSIX)操作系统标准、大多数[类Unix系统](https://zh.wikipedia.org/wiki/类Unix系统)、[Windows Vista](https://zh.wikipedia.org/wiki/Windows_Vista)、[Windows 7](https://zh.wikipedia.org/wiki/Windows_7)都支持符号链接。[Windows 2000](https://zh.wikipedia.org/wiki/Windows_2000)与[Windows XP](https://zh.wikipedia.org/wiki/Windows_XP)在某种程度上也支持符号链接。
>
> 符号链接的操作是透明的：对符号链接文件进行读写的程序会表现得像直接对目标文件进行操作。某些需要特别处理符号链接的程序（如备份程序）可能会识别并直接对其进行操作。
>
> 一个符号链接文件仅包含有一个文本字符串，其被操作系统解释为一条指向另一个文件或者目录的路径。它是一个独立文件，其存在并不依赖于目标文件。如果删除一个符号链接，它指向的目标文件不受影响。如果目标文件被移动、重命名或者删除，任何指向它的符号链接仍然存在，但是它们将会指向一个不复存在的文件。这种情况被有时被称为*被遗弃*。

### -G --file-search-regex  PATTERN 

Limit search to filenames matching PATTERN

限制搜素范围为 文件名含有PATTERN的文件

```shell
~ ❯ ag -G 'test' name ./agtest                                                          02:45:10 PM
agtest/text
21:     --color-path         Color codes for path names (Default: 1;32)
23:     --[no]filename       Print file names (Enabled unless searching a single file)
24:  -H --[no]heading        Print file names before each file's matches
28:  -g --filename-pattern PATTERN
29:                          Print filenames matching PATTERN
30:  -l --files-with-matches Only print filenames that contain matches
33:                          Only print filenames that don't contain matches
48:  -0 --null --print0      Separate filenames with null (for 'xargs -0')
57:  -G --file-search-regex  PATTERN Limit search to filenames matching PATTERN
61:                          (literal file/directory names also allowed)
62:     --ignore-dir NAME    Alias for --ignore for compatibility with ack.

agtest/depth/text
21:     --color-path         Color codes for path names (Default: 1;32)
23:     --[no]filename       Print file names (Enabled unless searching a single file)
24:  -H --[no]heading        Print file names before each file's matches
28:  -g --filename-pattern PATTERN
29:                          Print filenames matching PATTERN
30:  -l --files-with-matches Only print filenames that contain matches
33:                          Only print filenames that don't contain matches
48:  -0 --null --print0      Separate filenames with null (for 'xargs -0')
57:  -G --file-search-regex  PATTERN Limit search to filenames matching PATTERN
61:                          (literal file/directory names also allowed)
62:     --ignore-dir NAME    Alias for --ignore for compatibility with ack.

agtest/lstest
3:NAME
35:              mation); with -l: show ctime and sort by name; otherwise:  sort  by  ctime,  newest
74:              in a long listing, don't print group names
93:              hyperlink file names; WHEN can be 'always' (default if omitted), 'auto', or 'never'
96:              append indicator with style WORD  to  entry  names:  none  (default),  slash  (-p),
121:              print entry names without quoting
135:       -Q, --quote-name
136:              enclose entry names in double quotes
139:              use quoting style WORD for  entry  names:  literal,  locale,  shell,  shell-always,
155:              sort by WORD instead of name: none (-U), size (-S), time (-t), version (-v), exten‐
172:              name; otherwise: sort by access time, newest first
```

```shell
~ ❯ ag -G 'lstest' name ./agtest                                                        02:45:23 PM
agtest/lstest
3:NAME
35:              mation); with -l: show ctime and sort by name; otherwise:  sort  by  ctime,  newest
74:              in a long listing, don't print group names
93:              hyperlink file names; WHEN can be 'always' (default if omitted), 'auto', or 'never'
96:              append indicator with style WORD  to  entry  names:  none  (default),  slash  (-p),
121:              print entry names without quoting
135:       -Q, --quote-name
136:              enclose entry names in double quotes
139:              use quoting style WORD for  entry  names:  literal,  locale,  shell,  shell-always,
155:              sort by WORD instead of name: none (-U), size (-S), time (-t), version (-v), exten‐
172:              name; otherwise: sort by access time, newest first
```

### --hidden             

Search hidden files (obeys .*ignore files) 

搜索包含隐藏文件（.*）

```shell
~ ❯ ag --hidden name agtest                                                             02:49:29 PM
agtest/.mvtest # 隐藏文件
3:NAME
4:       mv - move (rename) files
12:       Rename SOURCE to DEST, or move SOURCE(s) to DIRECTORY.
89:       rename(2)

agtest/text
21:     --color-path         Color codes for path names (Default: 1;32)
23:     --[no]filename       Print file names (Enabled unless searching a single file)
24:  -H --[no]heading        Print file names before each file's matches
28:  -g --filename-pattern PATTERN
29:                          Print filenames matching PATTERN
30:  -l --files-with-matches Only print filenames that contain matches
33:                          Only print filenames that don't contain matches
48:  -0 --null --print0      Separate filenames with null (for 'xargs -0')
57:  -G --file-search-regex  PATTERN Limit search to filenames matching PATTERN
61:                          (literal file/directory names also allowed)
62:     --ignore-dir NAME    Alias for --ignore for compatibility with ack.

agtest/lstest
3:NAME
35:              mation); with -l: show ctime and sort by name; otherwise:  sort  by  ctime,  newest
74:              in a long listing, don't print group names
93:              hyperlink file names; WHEN can be 'always' (default if omitted), 'auto', or 'never'
96:              append indicator with style WORD  to  entry  names:  none  (default),  slash  (-p),
121:              print entry names without quoting
135:       -Q, --quote-name
136:              enclose entry names in double quotes
139:              use quoting style WORD for  entry  names:  literal,  locale,  shell,  shell-always,
155:              sort by WORD instead of name: none (-U), size (-S), time (-t), version (-v), exten‐
172:              name; otherwise: sort by access time, newest first

agtest/depth/text
21:     --color-path         Color codes for path names (Default: 1;32)
23:     --[no]filename       Print file names (Enabled unless searching a single file)
24:  -H --[no]heading        Print file names before each file's matches
28:  -g --filename-pattern PATTERN
29:                          Print filenames matching PATTERN
30:  -l --files-with-matches Only print filenames that contain matches
33:                          Only print filenames that don't contain matches
48:  -0 --null --print0      Separate filenames with null (for 'xargs -0')
57:  -G --file-search-regex  PATTERN Limit search to filenames matching PATTERN
61:                          (literal file/directory names also allowed)
62:     --ignore-dir NAME    Alias for --ignore for compatibility with ack.
```

### -i --ignore-case        

Match case insensitively 

 不区分大小写

~~~shell
~ ❯ ag AFTER agtest                                                                     02:53:06 PM
~ ❯ ag -i AFTER agtest                                                                  02:53:13 PM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters

agtest/depth/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~~~

### --ignore PATTERN     

Ignore files/directories matching PATTERN 

忽略含有PATTERN的文件或文件夹

```shell
~ ❯ ag after agtest                                                                     02:53:18 PM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters

agtest/depth/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag --ignore 'depth' after agtest       #忽略文件夹                                     02:55:49 PM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯                                                                          
```

```shell
~ ❯  ag  name agtest                                                                    03:03:15 PM
agtest/text
21:     --color-path         Color codes for path names (Default: 1;32)
23:     --[no]filename       Print file names (Enabled unless searching a single file)
24:  -H --[no]heading        Print file names before each file's matches
28:  -g --filename-pattern PATTERN
29:                          Print filenames matching PATTERN
30:  -l --files-with-matches Only print filenames that contain matches
33:                          Only print filenames that don't contain matches
48:  -0 --null --print0      Separate filenames with null (for 'xargs -0')
57:  -G --file-search-regex  PATTERN Limit search to filenames matching PATTERN
61:                          (literal file/directory names also allowed)
62:     --ignore-dir NAME    Alias for --ignore for compatibility with ack.

agtest/depth/text
21:     --color-path         Color codes for path names (Default: 1;32)
23:     --[no]filename       Print file names (Enabled unless searching a single file)
24:  -H --[no]heading        Print file names before each file's matches
28:  -g --filename-pattern PATTERN
29:                          Print filenames matching PATTERN
30:  -l --files-with-matches Only print filenames that contain matches
33:                          Only print filenames that don't contain matches
48:  -0 --null --print0      Separate filenames with null (for 'xargs -0')
57:  -G --file-search-regex  PATTERN Limit search to filenames matching PATTERN
61:                          (literal file/directory names also allowed)
62:     --ignore-dir NAME    Alias for --ignore for compatibility with ack.

agtest/lstest
3:NAME
35:              mation); with -l: show ctime and sort by name; otherwise:  sort  by  ctime,  newest
74:              in a long listing, don't print group names
93:              hyperlink file names; WHEN can be 'always' (default if omitted), 'auto', or 'never'
96:              append indicator with style WORD  to  entry  names:  none  (default),  slash  (-p),
121:              print entry names without quoting
135:       -Q, --quote-name
136:              enclose entry names in double quotes
139:              use quoting style WORD for  entry  names:  literal,  locale,  shell,  shell-always,
155:              sort by WORD instead of name: none (-U), size (-S), time (-t), version (-v), exten‐
172:              name; otherwise: sort by access time, newest first
~ ❯ ag --ignore 'lstest' name agtest            #忽略文件                                        03:02:27 PM
agtest/text
21:     --color-path         Color codes for path names (Default: 1;32)
23:     --[no]filename       Print file names (Enabled unless searching a single file)
24:  -H --[no]heading        Print file names before each file's matches
28:  -g --filename-pattern PATTERN
29:                          Print filenames matching PATTERN
30:  -l --files-with-matches Only print filenames that contain matches
33:                          Only print filenames that don't contain matches
48:  -0 --null --print0      Separate filenames with null (for 'xargs -0')
57:  -G --file-search-regex  PATTERN Limit search to filenames matching PATTERN
61:                          (literal file/directory names also allowed)
62:     --ignore-dir NAME    Alias for --ignore for compatibility with ack.

agtest/depth/text
21:     --color-path         Color codes for path names (Default: 1;32)
23:     --[no]filename       Print file names (Enabled unless searching a single file)
24:  -H --[no]heading        Print file names before each file's matches
28:  -g --filename-pattern PATTERN
29:                          Print filenames matching PATTERN
30:  -l --files-with-matches Only print filenames that contain matches
33:                          Only print filenames that don't contain matches
48:  -0 --null --print0      Separate filenames with null (for 'xargs -0')
57:  -G --file-search-regex  PATTERN Limit search to filenames matching PATTERN
61:                          (literal file/directory names also allowed)
62:     --ignore-dir NAME    Alias for --ignore for compatibility with ack.
```

  ### -p --path-to-ignore STRING
  Use .ignore file at STRING

### -Q --literal            

Don't parse PATTERN as a regular expression 

不再将pattern解析为正则表达式

```shell
~ ❯ ag  '[a-z]fter' agtest                                                              03:27:42 PM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters

agtest/depth/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag  -Q '[a-z]fter' agtest
```

###  -s --case-sensitive     

Match case sensitively # 区分大小写

~~~shell
~ ❯ ag  -s 'after' agtest                                                               03:28:28 PM
agtest/depth/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters

agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag  -s 'After' agtest
~~~

 ### -S --smart-case         

Match case insensitively unless PATTERN contains uppercase characters (Enabled by default)

除非PATTERN包含大写字母 否则不区分大小写 (默认启用)

~~~shell
~ ❯ ag after agtest                                                                     03:35:15 PM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters

agtest/depth/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag After agtest                                                                     03:35:19 PM
~ ❯ ag -S After agtest                                                                  03:35:23 PM
~ ❯ ag -S after agtest                                                                  03:35:28 PM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters

agtest/depth/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~~~

 ### -t --all-text           

Search all text files (doesn't include hidden files) 

搜索所有文本文件（不包括隐藏文件）

```shell
~ ❯ ag --hidden mv agtest                                                               03:41:26 PM
agtest/.mvtest
1:MV(1)                                     User Commands                                     MV(1)
4:       mv - move (rename) files
7:       mv [OPTION]... [-T] SOURCE DEST
8:       mv [OPTION]... SOURCE... DIRECTORY
9:       mv [OPTION]... -t DIRECTORY SOURCE...
80:       Report mv translation bugs to <https://translationproject.org/team/>
91:       Full documentation at: <https://www.gnu.org/software/coreutils/mv>
92:       or available locally via: info '(coreutils) mv invocation'
94:GNU coreutils 8.30                        September 2019                                    MV(1)

agtest/mvtest
1:MV(1)                                     User Commands                                     MV(1)
4:       mv - move (rename) files
7:       mv [OPTION]... [-T] SOURCE DEST
8:       mv [OPTION]... SOURCE... DIRECTORY
9:       mv [OPTION]... -t DIRECTORY SOURCE...
80:       Report mv translation bugs to <https://translationproject.org/team/>
91:       Full documentation at: <https://www.gnu.org/software/coreutils/mv>
92:       or available locally via: info '(coreutils) mv invocation'
94:GNU coreutils 8.30                        September 2019                                    MV(1)
~ ❯ ag -t mv agtest                                                                     03:41:35 PM
agtest/mvtest
1:MV(1)                                     User Commands                                     MV(1)
4:       mv - move (rename) files
7:       mv [OPTION]... [-T] SOURCE DEST
8:       mv [OPTION]... SOURCE... DIRECTORY
9:       mv [OPTION]... -t DIRECTORY SOURCE...
80:       Report mv translation bugs to <https://translationproject.org/team/>
91:       Full documentation at: <https://www.gnu.org/software/coreutils/mv>
92:       or available locally via: info '(coreutils) mv invocation'
94:GNU coreutils 8.30                        September 2019
```

### -u --unrestricted       

Search all files (ignore .ignore, .gitignore, etc.;searches binary and hidden files as well)

查询所有文件

~~~shell
~ ❯ ag mv agtest                                                                        03:43:27 PM
agtest/mvtest
1:MV(1)                                     User Commands                                     MV(1)
4:       mv - move (rename) files
7:       mv [OPTION]... [-T] SOURCE DEST
8:       mv [OPTION]... SOURCE... DIRECTORY
9:       mv [OPTION]... -t DIRECTORY SOURCE...
80:       Report mv translation bugs to <https://translationproject.org/team/>
91:       Full documentation at: <https://www.gnu.org/software/coreutils/mv>
92:       or available locally via: info '(coreutils) mv invocation'
94:GNU coreutils 8.30                        September 2019                                    MV(1)
~ ❯ ag -u mv agtest                                                                     03:43:42 PM
agtest/.mvtest
1:MV(1)                                     User Commands                                     MV(1)
4:       mv - move (rename) files
7:       mv [OPTION]... [-T] SOURCE DEST
8:       mv [OPTION]... SOURCE... DIRECTORY
9:       mv [OPTION]... -t DIRECTORY SOURCE...
80:       Report mv translation bugs to <https://translationproject.org/team/>
91:       Full documentation at: <https://www.gnu.org/software/coreutils/mv>
92:       or available locally via: info '(coreutils) mv invocation'
94:GNU coreutils 8.30                        September 2019                                    MV(1)

agtest/mvtest
1:MV(1)                                     User Commands                                     MV(1)
4:       mv - move (rename) files
7:       mv [OPTION]... [-T] SOURCE DEST
8:       mv [OPTION]... SOURCE... DIRECTORY
9:       mv [OPTION]... -t DIRECTORY SOURCE...
80:       Report mv translation bugs to <https://translationproject.org/team/>
91:       Full documentation at: <https://www.gnu.org/software/coreutils/mv>
92:       or available locally via: info '(coreutils) mv invocation'
94:GNU coreutils 8.30                        September 2019
~~~

### -v --invert-match       

反向匹配

```shell
~/agtest ❯ cat vtest                                                                    03:46:56 PM
1
2
3
4
5
```

```shell
~/agtest ❯ ag 1 vtest                                                                9s 03:46:36 PM
1:1
~/agtest ❯ ag -v 1 vtest                                                                03:46:52 PM
2:2
3:3
4:4
5:5
```

### -w --word-regexp       

 Only match whole words 

整个单词匹配

~~~shell
~ ❯ ag -w fter agtest                                                                   03:48:59 PM
~ ❯ ag -w after agtest                                                                  03:49:08 PM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters

agtest/depth/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~~~

###  -W --width NUM          

Truncate match lines after NUM characters 

匹配的行信息在num个字符后截断

~~~shell
~ ❯ ag -w after agtest                                                                  03:50:20 PM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters

agtest/depth/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag -W 15 after agtest                                                               03:50:28 PM
agtest/text
12:  -A --after [L [...]
26:  -C --context  [...]
63:  -m --max-coun [...]
79:  -W --width NU [...]

agtest/depth/text
12:  -A --after [L [...]
26:  -C --context  [...]
63:  -m --max-coun [...]
79:  -W --width NU [...]
~~~

### -z --search-zip 

Search contents of compressed (e.g., gzip) files 

压缩文件中匹配

~~~shell
~ ❯ ag after agtest                                                                     04:01:55 PM
agtest/depth/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters

agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters
~ ❯ ag -z after agtest                                                                  04:02:01 PM
agtest/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters

agtest/depth/text
12:  -A --after [LINES]      Print lines after match (Default: 2)
26:  -C --context [LINES]    Print lines before and after matches (Default: 2)
63:  -m --max-count NUM      Skip the rest of a file after NUM matches (Default: 10,000)
79:  -W --width NUM          Truncate match lines after NUM characters

agtest/agtest.tar.gz
205:  -A --after [LINES]      Print lines after match (Default: 2)

agtest/agtest.tar.gz
220:(Default: 2)

agtest/agtest.tar.gz
258:: 10,000)

agtest/agtest.tar.gz
275:
agtest/agtest.tar.gz
ag: truncated file: Success
533:%
~ ❯
~~~



