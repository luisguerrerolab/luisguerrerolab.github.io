---
layout    : post
title     : "Find, replace and delete text from files using the terminal in OS X"
date      : 2020-04-19 19: 28: 00 -0500
categories: [tech guides, terminal]
---

To edit:

- Section Find text in multiple files:
    - Add a more concise description on what is the result of running this command
    - Add extra details on how to exclude multiple paths - `-path -prune -o`
    - How the results change after adding `-type d|f`
    - What happens after removing the `-name` flag

**How would you change a block of text included in multiple files, without opening one by one?**

That was a question my employer asked me in my first job interview. That one has stuck with me because, as programmer, editing files is something I do quite often and until then I had never wondered how I could optimize it.

The technical sophistication[^1] I have gained along these years led me to discover these powerful commands to perform most common file editing tasks.

## Find and replace

To find a specific word or sentence that could appear in multiple files inside a directory and replace it with other one recursively, run:

```shell
$ grep -r ’old string' directory/* | xargs sed -i '' ’s/old string/new string/g'
```

You can run this command from the root directory or inside specific folders, by changing the `directory/*` value.

This command is particularly helpful when the name of a company, a person or even a database has changed, but the files that contain it are super large, making it complicated for a visual editor to apply the change.

## Remove a line

Remove an entire line that may appear in many files with the same extension across a directory:

```shell
$ find . -type f -iname \*.cs -execdir sed -i '' ‘/string to match/d' {} \;
```

## Find text in multiple files

For a given word or sentence, find all the files in a directory that contain it and print the name and the entire line where the text was found:

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

Each command has extra flags that can make it even more powerful. Stay curious and let your technical sophistication guide you.


---


[^1]: I discovered the term in the course [Learn Enough Ruby](https://www.learnenough.com/ruby) by Michael Hartl. He describes *Technical Sophistication* as the ability to figure out tech problems on your own, including concrete skills like coding, as well as fuzzy skills like Googling the error message and knowing when to just reboot the darn thing.

