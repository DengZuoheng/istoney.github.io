---
title: Linux命令之grep
category: tools
tags: grep
---

grep是Linux中的文本搜索工具，可以使用正则表达式搜索文本，并将匹配的文本打印。grep的全称是Global Regular Expression Print，即全局正则表达式打印。

在Linux中输入命令`grep --help`, 查看grep的命令使用帮助，包括命令格式、正则表达式选项、其他选项、输出控制和上下文控制五个部分。

### # 命令格式

```sh
Usage: grep [OPTION]... PATTERN [FILE]...
Search for PATTERN in each FILE or standard input.
PATTERN is, by default, a basic regular expression (BRE).
Example: grep -i 'hello world' menu.h main.c
```
grep的命令格式是grep [选项...] PATTERN [文件...]，即在多个文件中查找PATTERN，其中PATTERN是一个基本的正则表达式。查找中可以有多个选项。

### # 正则表达式选项

```sh
Regexp selection and interpretation:
  -E, --extended-regexp     PATTERN is an extended regular expression (ERE)
                            PATTERN是一个扩展正则式

  -F, --fixed-strings       PATTERN is a set of newline-separated fixed strings
                            PATTERN是一些用换行分割的固定字符串

  -G, --basic-regexp        PATTERN is a basic regular expression (BRE)
                            PATTERN是一个基本正则式

  -P, --perl-regexp         PATTERN is a Perl regular expression
                            PATTERN是一个Perl正则式

  -e, --regexp=PATTERN      use PATTERN for matching
                            指定使用字符串PATTERN进行匹配

  -f, --file=FILE           obtain PATTERN from FILE
                            从文件中读取进行匹配的PATTERN, 每行一个

  -i, --ignore-case         ignore case distinctions
                            忽略字符的大小写

  -w, --word-regexp         force PATTERN to match only whole words
                            只显示全字匹配的行

  -x, --line-regexp         force PATTERN to match only whole lines
                            只显示全行匹配的行

  -z, --null-data           a data line ends in 0 byte, not newline
                            在打印数据后输出0个字符，而不是换行
```

### # 其他选项

```sh
Miscellaneous:
  -s, --no-messages         suppress error messages
                            隐藏错误信息

  -v, --invert-match        select non-matching lines
                            对匹配结果取反，选取不匹配的行

  -V, --version             print version information and exit
                            打印版本信息并退出

      --help                display this help and exit
                            打印帮助信息并退出

      --mmap                deprecated no-op; evokes a warning
```

### # 输出控制

```sh
Output control:
  -m, --max-count=NUM       stop after NUM matches
                            只显示匹配的前NUM行

  -b, --byte-offset         print the byte offset with output lines
                            将匹配行的位移和匹配的内容一起显示

  -n, --line-number         print line number with output lines
                            将匹配行的行号和内容一起打印

      --line-buffered       flush output on every line
                            在每一行都清空输出缓存

  -H, --with-filename       print the file name for each match
                            将匹配行的文件名和内容一起打印

  -h, --no-filename         suppress the file name prefix on output
                            显示匹配行的内容时隐藏文件名

      --label=LABEL         use LABEL as the standard input file name prefix
                            使用LABEL作为输入文件名前缀

  -o, --only-matching       show only the part of a line matching PATTERN
                            只显示匹配行中匹配的那一部分

  -q, --quiet, --silent     suppress all normal output
                            隐藏所有的正常输出，即不显示任何输出

      --binary-files=TYPE   assume that binary files are TYPE;
                            TYPE is 'binary', 'text', or 'without-match'
                            设二进制文件的类型是TYPE

  -a, --text                equivalent to --binary-files=text
                            不要忽略二进制文件

  -I                        equivalent to --binary-files=without-match
                            忽略二进制文件

  -d, --directories=ACTION  how to handle directories;
                            ACTION is 'read', 'recurse', or 'skip'
                            按照ACTION处理目录

  -D, --devices=ACTION      how to handle devices, FIFOs and sockets;
                            ACTION is 'read' or 'skip'
                            按照ACTION处理设备、FIFO队列和socket连接

  -r, --recursive           like --directories=recurse
                            递归打印

  -R, --dereference-recursive  likewise, but follow all symlinks
                            废除递归，但是照常处理符号连接

      --include=FILE_PATTERN  search only files that match FILE_PATTERN
                            只对文件名匹配FILE_PATTERN的文件进行搜索

      --exclude=FILE_PATTERN  skip files and directories matching FILE_PATTERN
                            跳过与FILE_PATTERN匹配的文件和目录

      --exclude-from=FILE   skip files matching any file pattern from FILE
                            跳过与FILE中的文件名匹配的所有文件

      --exclude-dir=PATTERN  directories that match PATTERN will be skipped.
                            跳过名字与PATTERN匹配的目录

  -L, --files-without-match  print only names of FILEs containing no match
                            打印不包含匹配的文件的文件名

  -l, --files-with-matches  print only names of FILEs containing matches
                            打印包含匹配的文件的文件名

  -c, --count               print only a count of matching lines per FILE
                            对于每个文件打印最多count个匹配的行

  -T, --initial-tab         make tabs line up (if needed)
                            在打印的行内容之前添加tap，这个选项在和 -H -n -b 一起使用时尤其有用

  -Z, --null                print 0 byte after FILE name
                            在打印内容之后是输出0个字节，用于去除换行等额外的字符
```

### # 上下文控制

```sh
Context control:
  -B, --before-context=NUM  print NUM lines of leading context
                            除了打印匹配的行，还打印匹配行之前的NUM行

  -A, --after-context=NUM   print NUM lines of trailing context
                            除了匹配的行，还打印匹配行之后的NUM行

  -C, --context=NUM         print NUM lines of output context
                            除了匹配的行，打印匹配行的上下各NUM行

  -NUM                      same as --context=NUM
                            同 -C

      --color[=WHEN],
      --colour[=WHEN]       use markers to highlight the matching strings;
                            WHEN is 'always', 'never', or 'auto'
                            选择条件，对匹配的字符串进行高亮

  -U, --binary              do not strip CR characters at EOL (MSDOS/Windows)
  
  -u, --unix-byte-offsets   report offsets as if CRs were not there
                            (MSDOS/Windows)
```
