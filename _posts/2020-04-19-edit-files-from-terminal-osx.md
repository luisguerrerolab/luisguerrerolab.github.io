---
layout     : post
title      : "Search, replace and delete text from files using the terminal in OS X"
date       : 2020-04-19 19:28:00 -0500
modified_at: 2020-06-06 18:50:00 -0500
---

**How would you replace text included in multiple files, without opening one by one?**

That was a question that an employer once asked me in a job interview. Although I gave an acceptable answer (using the `grep` command), that question has stuck with me because, as programmer, editing files is something I do quite often and until then I had never wondered how I could optimize it.

The technical sophistication[^1] I have gained along these years led me to discover these powerful commands to perform most common file editing tasks.

## Find and replace

To find a specific word or sentence in files with the specified extension and replace it with another, run:

```shell
$ LC_ALL=C find . -name \*.ext -exec sed -i '' 's/text example/another example/g' {} \;
```

This command is particularly helpful when a company, a person or even a database have changed names, but the files where the old name appears are super large, making it complicated for a visual editor to open the file and to apply the change.

## Remove a line

I want to share two options to remove a line containing a specified text from certain files.

Option #1, with a caveat:

```shell
$ LC_ALL=C find . -name \*.ext -execdir sed -i '' '/text example/d' {} \;
```

The command will do the work, but the caveat is that it will add a new line at the end of each file that matches the extension, even if it does not contain the specified text.

Option #2, my choice:

```shell
LC_ALL=C find . -name \*.ext -exec grep -i --color=auto "text example" {} \; -exec sed -n -i '' '/text example/!p' {} \; -print
```

The above command first verifies that files with the specified extension actually contain `text example` before moving to the part that deletes the line from them. This is the work-around I found make rid of the caveat in option #1.

## Search text through files

```shell
$ find . -name \*.ext -exec grep -i --color=auto "text example" {} +
```

Command breakdown:
- `find . -name \*.ext` searches for files that end with `ext` extension.
- `-exec` indicates that a command will be run against the results of the `find` command.
- `grep -i` is the command that will look up for the text inside the files found by `find`.
- `--color=auto` will highlight the text that matches the search argument.
- `"text example"` is the search argument.
- `{}` represents the files found by the `find` command.
- `+` makes the `find` command to print the name of the file along with the line that contains the text.

## Putting it all together

An example is always better. Let's say you want to remove the `using Sytem.IO;` directive from all the C# files containing it. If you are using Git, probably you wouldn't worry too much if something goes wrong editing files, but if anyway you want to be extra careful and want to confirm which files will be affected first, you can run:

```shell
$ find . -name \*.ext -exec grep --color=auto 'using System.IO;' {} \; -print
using System.IO;
./some_folder/SomeClass.cs
...
```

With the visual confirmation ready, now you can be confident and run the command to delete that line from all those files:

```shell
$ LC_ALL=C find . -name \*.ext -exec grep -i --color=auto "using System.IO;" {} \; -exec sed -n -i '' '/using System.IO;/!p' {} \; -print
```

Each command has extra flags that can make it even more powerful. Stay curious and let your technical sophistication guide you.

## Useful Resources

- [What does `LC_ALL=C` do?](https://unix.stackexchange.com/a/87748)
- [Difference between `-exec \;` and `-exec +`](https://unix.stackexchange.com/a/202392)
- [What does `sed -n -i '' '/text/!p/` actually do?](https://stackoverflow.com/questions/5410757/how-to-delete-from-a-text-file-all-lines-that-contain-a-specific-string/5410784#comment34621094_5413132)


---
[^1]: I discovered the term in the course [Learn Enough Ruby](https://www.learnenough.com/ruby) by Michael Hartl. He describes **Technical Sophistication** as *the ability to figure out tech problems on your own, including concrete skills like coding, as well as fuzzy skills like Googling the error message and knowing when to just reboot the darn thing*.

