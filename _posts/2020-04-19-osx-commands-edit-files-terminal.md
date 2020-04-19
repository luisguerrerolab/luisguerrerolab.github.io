---
layout: 		post
title:  		"Find, replace and delete text from files using the terminal in OS X"
date:   		2020-04-19 13:38:25 -0500
categories: [tech guides, terminal]
published:  false
---

**How would you change the company name included in multiple files, without opening one by one?**

That was a question that my employer asked me in my first job interview. That one has stuck with me because, as programmer, editing files is something I do quite often and until then I had never wondered how I could optimize it.

The technical sophistication[^1] I have gained along these years led me to discover a handful of powerful and easy commands to perform most common file editing tasks.

## Find and replace

Find a specific word or sentence in multiple files across a folder and replace it with other one recursively:

```shell
$ grep -r ’old string' folder/* | xargs sed -i '' ’s/old string/new string/g'
```

## Remove a single line

Remove an entire line in files with the same extension across a directory that match the `sed` pattern:

```shell
$ find . -type f -iname \*.cs -execdir sed -i '' ‘/string to match/d' {} \;
```

## Find text in multiple files

Find the line across all the files in a directory that matches the pattern and return the filename:

```bash
$ find . -name "*.csproj" -exec grep --color=auto 'string to match' {} \; -print
```

## An example

If you would like to remove the `using Sytem.IO;` directive from all the C# files that contain it, you could run:

```shell
$ find . -type f -iname \*.cs -execdir sed -i '' ‘/using System.IO/d' {} \;
```

But if you want to make sure first which files will be affected, you could run:
```
$ find . -name "*.cs" -exec grep --color=auto 'using System.IO' {} \; -print
```

• • •

Each command has extra flags that can make it even more powerful. Let your technical sophistication guide you.


---


[^1]: I discovered the term in the course [Learn Enough Ruby](https://www.learnenough.com/ruby) by Michael Hartl. He describes *Technical Sophistication* as the ability to figure out tech problems on your own, including concrete skills like coding, as well as fuzzy skills like Googling the error message and knowing when to just reboot the darn thing.

