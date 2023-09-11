# Data Wrangling on the Command Line
*Make sure that you `cd` to the `hackers-dwotcl` directory that you cloned from GitHub before running these commands*

*Prior to each section, there will be an instruction indicating whether these commands are to be run from `hackers-dwotcl` or from `hackers-dwotcl/logs`*

## Table of Contents

**[Objectives](#objectives)**

**[Source](#source)**

  * [Standard Stream](#stdstr)
  * [Redirection](#redirection)

**[Filter](#filter)**

  * [grep](#grep)
  * [egrep (or grep -E)](#egrep)
  * [find](#find)
  * [head & tail](#head&tail)

**[Transform](#transform)**

  * [sed](#sed)
  * [awk](#awk)
  * [wc](#wc)
  * [sort](#sort)
  * [uniq](#uniq)
  * [paste](#paste)
  * [tr & cut](#tr&cut)

**[Output](#output)**

<div id="objectives"/>

## Objectives
We have two sample logs, one from a typical Linux server, and one from the OpenSSH program. At the end of our data wrangling session, we should be able to get a `wrangled-auth-fail.csv` file, all without leaving the command line!

This file will contain records of all network activities associated with an IP address from both the Linux server and the OpenSSH client. 
- `user`: the user associated with the authentication failure
- `remote_host`: the IP address or name of the remote host
- `system`: whether it came from the Linux Server or the OpenSSH client
- `count`: the number of times the authentication failure happened for this particular IP address

The file will also be sorted in descending order based on the `count` field.

<div id="source"/>

## Source
*cd to `hackers-dwotcl/logs`*

<div id="stdstr"/>

### Standard Stream
-   Standard input (`0`)
	-   A data stream going into a program - your keyboard, or output from another program
-   Standard output (`1`)
	-   Normal output displayed on command line
-   Standard error (`2`)
	-   Error output displayed on the command line

<div id="redirection"/>

### Redirection
Pipe:

    cat sample_linux.log | head -10

Redirect operators (< and >):

    tail -10 < sample_linux.log > sample_last10.log

Append instead of overwriting (>>):

    tail -10 sample_linux.log >> sample_last10.log

Redirecting stderr to stdout:

    ls stderr | grep "hello"
    ls stderr 2>&1 | grep "hello"

Interpolating a command's output:

    grep "authentication failure" $(find ./logs/sample* -type f -mtime -7) | tail -10
    grep "authentication failure" `find ./logs/sample* -type f -mtime -7` | tail -10

<div id="filter"/>

## Filter

<div id="grep"/>

### grep
*`grep` is one of the most useful Unix commands available!*

Used to search for contents from stdin or a file.

Basic usage:

    cat sample_linux.log | grep "sshd"
    grep "sshd" sample_linux.log

Useful flags:

 - `-i`: ignore case
 - `-v`: omit search string from results (inverse of regular grep)
 - `-r`: recursively search current directory & all its subdirectories
 - `-o`: print only the parts that match your search string
 - `-n`: print line number for input file
 - `-h`: omit filename from output (default when there's only one file to search)
 - `-H`: include filename from output (default when there is more than one file to search)
 - `-Bn`: print matched line and `n` number of lines before
 - `-An`: print matched line and `n` number of lines after

```
grep -i "alert" sample_linux.log
grep -r "sshd" ../
grep "authentication failure" sample_linux.log | grep -vn "root"
grep -B1 "session closed" sample_linux.log
grep -A1 "session opened" sample_linux.log
```

<div id="egrep"/>

### egrep (or grep -E)
`grep` but with extended regex.

Basic usage:

    egrep "logrotate|su\(.*\)\[.*\]" *.log

<div id="find"/>

### find
Searches for files or directories.

*cd to `hackers-dwotcl`*

Basic usage:

    find ./logs -name "*.log"

Useful flags:

 - `-name` or `-iname`: search for files and directories based on their name (i for case insensitive)
 - `-type`: filter results based on type; `f` for regular files, `d` for directories, `l` for symbolic links
 - `-mtime` or `-mmin`: find files based on modification time in days (`mtime`) or minutes (`mmin`)
	 - `-` for less than, `+` for more than, no symbol for exactly the specified time
 - `-size`: search for files based on size
	 -  default in bytes, with optional suffixes like `c` (bytes), `k` (kilobytes), `M` (megabytes), `G` (gigabytes)
	 - `+` for more than, `-` for less than, no symbol for exactly the specified size

```
find ./logs -type f -name "*.log" -mtime -7
find ./logs -type f -name "*.log" -size +200k
```

<div id="head&tail"/>

### head & tail
Gets the first or last `n` lines of a file.

*cd to `hackers-dwotcl/logs`*

Basic usage

    head -10 sample_linux.log
    tail -10 sample_linux.log

<div id="transform"/>

## Transform
*cd to `hackers-dwotcl/logs`*

Here is where the fun begins!

<div id="sed"/>

### sed
(Text) Stream editor. Most commonly used for its substitution (`s`) feature.

Basic usage:

    grep "authentication failure" sample_linux.log | sed 's/authentication failure/auth fail/g' > auth_fail.log

Using regex in sed commands:

    sed 's/combo [a-z]+(.*)\[.*\]//g' auth_fail.log

Note that if we leave the new_text field blank, we can effectively delete chunks of text! In this case, we want to any text resembling this format: sshd(pam_unix)[28886].

In-place replacing (editing the original file directly):

    sed -i 's/combo [a-z]+(.*)\[.*\]//g' auth_fail.log

In-place replacing and create a backup of the original file with extension .bkp (note that for OS X, we *need* to specify the backup extension, otherwise the command won't work).

    sed -i.bkp 's/rhost/rhost_ip/g' auth_fail.log

Other features of sed beside substitution (read up on your own!):

 - Add lines
	 - Add before matched lines (i)
	 - Add after matched lines (a)
 - Replace lines \(c\)
 - Delete lines (d)
 - Specify a series of commands (e)

<div id="awk"/>

### awk

awk is a programming language that happens to be really good at processing text. It's also extremely powerful!

awk's default behaviour is to take its input contents (either a file or stdin), and take each line as record. awk then automatically splits each record into fields, using whitespace as the default field separator.

awk also has certain built-in field variables & operators that can be accessed:

 - `NR`: keeps track of the current count of input records (basically think of it as a line number)
 - `NF`: keeps a count of the number of fields within the current record
	 - Consequently, doing `$NF` gives you the last record in the file
 - `FS`: input field separator (default is whitespace)
 - `RS`: input record separator (default is newline)
 - `OFS`: output field separator
 - `ORS`: output record separator
 - `BEGIN` and `END`: allows us to run awk code before and after processing lines from stdin

Basic usage:

    grep "authentication failure;" sample*.log | awk '{print $(NF-1), $NF}'

Useful flags:

- `-F`: defines the field separator; default is whitespace
- `-v VARNAME=VALUE`: Defines a variable (`VARNAME`) with a specific value (`VALUE`) that can be used within the `awk` script.
	- `-v RS=RECORD_SEPARATOR`: Sets the input record separator. The default is a newline, but you can specify a different character or string.
	- `-v ORS=OUTPUT_RECORD_SEPARATOR`: Sets the output record separator. Similar to `RS`, but for output formatting.
	- `-v OFS=OUTPUT_FIELD_SEPARATOR`: Sets the output field separator. Similar to `-F`, but for output formatting.

Since awk is a programming language, we can do more complicated things with it, such as conditional printing:

    grep -i "authentication failure;" sample*.log | awk '$NF ~ /user=/ {print $1 "; ", $(NF-1), $NF}' | sed 's/;/:/g' > auth_fail_users_ip.log

Here, we're telling awk to only print the 1st, 2nd last (NF - 1) and last (NF) records that contain 'user=' in their last field.

- `$NF`: last row
- `~`: matches a pattern
- `/user=/`: pattern to match within the slashes

Since awk is a programming language, it also supports the use of the ternary operator to print output conditionally:

    grep -i "authentication failure;" sample*.log | awk '$NF ~ /user=/ {print "system="($1 ~ /openssh/ ? "openssh" : "linux")" " $(NF-1), $NF}' | sed 's/;/:/g' > auth_fail_users_ip.log

Note this line `"system="($1 ~ /openssh/ ? "openssh" : "linux")" "`

- Basically we are saying, if $1 contains "openssh" then output `system=openssh `, else output `system=linux `
    
Now we're starting to build bigger data wrangling pipelines using sed and awk! Can you see how we're transforming our raw data to pick out features that we want? Compare these:

    cat sample*
    cat auth_fail_users_ip.log

<div id="wc"/>

### wc
Basic usage:

    grep "sshd" sample_openssh.log | wc

The default behaviour of wc is to display, in the following order:
\<line count `-l`\> \<word count `-w`\> \<byte count `-c`\>

Useful flags:

- `-l`: counts the number of lines in the file
- `-w`: counts the number of words in the file (words are defined as sequences of characters separated by whitespace)
- `-m`: counts the number of characters in the file
- `-c`: counts the number of bytes in the file (useful to determine file size in bytes)

<div id="sort"/>

### sort
Basic usage:

    cat sshd_auth_fail.log | sed 's/.*combo //g' | sort

Note that by default, `sort` sorts using lexicographical order.

Useful flags:

- `-r`: reverse sorting order
- `-n`: numeric sort
- `-u`: unique; remove duplicate from output and leave only unique lines
- `-kn` and `-t`: `kn` allows you sort using the `n`th field, using whitespace as the default separator. `-t` allows you to specify a custom separator

Try running the same command with the following flags and notice the difference:

    cat sshd_auth_fail.log | sed 's/.*combo //g' | sort -n -t'[' -k2

<div id="uniq"/>

### uniq
Basic usage:

    cat sshd_auth_fail.log | uniq

The basic behaviour of uniq is that it removes consecutive duplicate lines from the input and displays only the unique lines. So meaning that if you have two of the same line but they are not consecutive, both of these lines will still get printed in the final output.

Useful flags:

- `-c`: prefix lines with consecutive duplicate lines (note that you typically should run `sort` before this, since only consecutive duplicate lines are counted)
- `-u`: prints only unique lines in the output, regardless of whether they are consecutive or not, i.e. print every unique line exactly once.
- `-i`: ignore case; i.e. abc and ABC will be treated as duplicates

<div id="paste"/>

### paste
Basic usage:

    paste sample_linux.log sample_openssh.log > pasted.log
    cat auth_fail_users_ip.log | awk '{print $NF}' | sort -u | paste -s -

paste combines lines horizontally. It can do this for n number of files (combine all the lines each file horizontally), or can combine lines from stdin horizontally if used with `-s` flag.

Useful flags:

- `-s`: serial; combine lines from all input files into a single line, rather than side by side
- `-d`: delimiter: specify custom delimiter (default is tab)

<div id="tr&cut"/>

### tr & cut
 - `tr`: used to translate or delete characters in a text stream
 - `cut`: used to extract specific fields from lines of text based on a delimiter character

We won't be covering these today, because `awk` can basically do everything that these two can do and more.

- Then why bother with these commands, you might ask? These commands can be more lightweight than awk, and may be useful in situations when you need a faster and easier solution.

<div id="output"/>

## Output
Back to our initial objective! Remember, the output destination of our wrangling pipeline can either be stdout, or a file. So let's create our data wrangling pipeline to get our `wrangled-auth-fail.csv` file. To recap, this is what we want in our file:

> - `user`: the user associated with the authentication failure
> - `remote_host`: the IP address or name of the remote host
> - `system`: whether it came from the Linux Server or the OpenSSH client
> - `count`: the number of times the authentication failure happened for this particular IP address
> 
> The file will also be sorted in descending order based on the `count`
> field.

Before we start, make sure that you cd to `hackers-dwotcl`.

If you already have the file `./logs/auth_fail_users_ip.log` then great! If not run the following command pipeline: 

    grep -i "authentication failure;" ./logs/sample*.log | awk '$NF ~ /user=/ {print "system="($1 ~ /openssh/ ? "openssh" : "linux")" " $(NF-1), $NF}' | sed 's/;/:/g' > ./logs/auth_fail_users_ip.log

Next, let's create our pipeline:

    cat ./logs/auth_fail_users_ip.log | sed 's/rhost/remote_host/g' | sort | uniq -c | sort -rn | awk -v OFS="," 'BEGIN {print "count,system,remote_host,user"} {$1=$1; print}' | sed 's/,[^=]*=/,/g' > wrangled-auth-fail.csv

Let's break down this pipeline:

1. `sed 's/rhost/remote_host/g'`: rename rhost to remote_host
2. `sort | uniq -c | sort -rn`: sort and get the count of consecutive duplicate lines, then sort the counts numerically in descending order
3. `awk -v OFS="," 'BEGIN {print "count,system,remote_host,user"} {$1=$1; print}'`
	
	- Define output field separator as `,`
	- `BEGIN`: add our csv headers `count,system,remote_host,user` to the start of the file
	- `$1=$1`: remove the whitespaces around the count field
4. `sed 's/,[^=]*=/,/g'`: remove everything that is sandwiched between `,` and `=`, and replace it with `,` (effectively removing the field names)
5. `> wrangled-auth-fail.csv`: redirect stdout into our `wrangled-auth-fail.csv` file

In fact, you can combine everything into a single massive pipeline as such:

    grep -i "authentication failure;" ./logs/sample*.log | awk '$NF ~ /user=/ {print "system="($1 ~ /openssh/ ? "openssh" : "linux")" " $(NF-1), $NF}' | sed 's/;/:/g' | sed 's/rhost/remote_host/g' | sort | uniq -c | sort -rn | awk -v OFS="," 'BEGIN {print "count,system,remote_host,user"} {$1=$1; print}' | sed 's/,[^=]*=/,/g' > wrangled-auth-fail.csv

And there you have it! We have just created our own data wrangling pipeline! 🎉

*Copyright &copy; 2023 Daniel Ong (@danielonges on GitHub). All rights reserved.*

> Written with [StackEdit](https://stackedit.io/).

