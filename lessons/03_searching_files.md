---
layout: page
title: "Introduction to the Shell"
authors: "Sheldon McKay, Paul Wilson, Milad Fatenejad, Sasha Wood and Radhika Khetani"
comments: true
date: 2015-07-30
---

## Learning objectives

* Practice searching for characters or patterns in a text file using the `grep` command
* Learn about output redirection
* Explore how to use the pipe (`|`) character to chain together commands

## Searching files

We showed a little how to search within a file using `less`. We can also
search within files without even opening them, using `grep`. Grep is a command-line
utility for searching plain-text data sets for lines matching a string or regular expression.
Let's give it a try!

Suppose we want to see how many reads in our file have really bad, with 10 consecutive Ns  
Let's search for the string NNNNNNNNNN in file `SRR098026.fastq` in the `untrimmed_fastq` folder:

```bash
$ cd ~/dc_sample_data/untrimmed_fastq/
$ grep NNNNNNNNNN SRR098026.fastq
```

We get back a lot of lines.  What is we want to see the whole fastq record for each of these read.
We can use the '-B' argument for grep to return the matched line plus one before (-B 1) and two
lines after (-A 2). Since each record is four lines and the last second is the sequence, this should
give the whole record.

```bash
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq
```

for example:

    @SRR098026.177 HWUSI-EAS1599_1:2:1:1:2025 length=35
    CNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
    +SRR098026.177 HWUSI-EAS1599_1:2:1:1:2025 length=35
    #!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

****
**Exercise**

1) Search for the sequence GNATNACCACTTCC in SRR098026.fastq.
In addition to finding the sequence, have your search also return
the name of the sequence.

2) Search for that sequence in both fastq files.
****

## Redirection

We're excited we have all these sequences that we care about that we
just got from the FASTQ files. That is a really important motif
that is going to help us answer our important question. But all those
sequences just went whizzing by with grep. How can we capture them?

We can do that with something called "redirection". The idea is that
we're redirecting the output to the terminal (all the stuff that went
whizzing by) to something else. In this case, we want to print it
to a file, so that we can look at it later.

The redirection command for putting something in a file is `>`

Let's try it out and put all the sequences that contain 'NNNNNNNNNN'
from all the files in to another file called 'bad_reads.txt'

```bash
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq > bad_reads.txt
```

The prompt should sit there a little bit, and then it should look like nothing
happened. But type `ls`. You should have a new file called bad_reads.txt. Take
a look at it and see if it has what you think it should.

If we use '>>', it will append to rather than overwrite a file.  This can be useful for
saving more than one search, for example:

```bash
$ grep -B1 -A2 NNNNNNNNNN SRR097977.fastq >> bad_reads.txt
```

There's one more useful redirection command that we're going to show, and that's
called the pipe command, and it is `|`. It's probably not a key on
your keyboard you use very much. What `|` does is take the output that
scrolling by on the terminal and then can run it through another command.
When it was all whizzing by before, we wished we could just slow it down and
look at it, like we can with `less`. Well it turns out that we can! We pipe
the `grep` command through `less`

```bash
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq | less
```

Now we can use the arrows to scroll up and down and use `q` to get out.

We can also do something tricky and use the command `wc`. `wc` stands for
`word count`. It counts the number of lines or characters. So, we can use
it to count the number of lines we're getting back from our `grep` command.
And that will magically tell us how many sequences we're finding. We're

```bash
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq | wc
```

That tells us the number of lines, words and characters in the file. If we
just want the number of lines, we can use the `-l` flag for `lines`.

```bash
$ grep -B1 -A2 NNNNNNNNNN SRR098026.fastq | wc -l
```

Redirecting is not super intuitive, but it's really powerful for stringing
together these different commands, so you can do whatever you need to do.

The philosophy behind these command line programs is that none of them
really do anything all that impressive. BUT when you start chaining
them together, you can do some really powerful things really
efficiently. If you want to be proficient at using the shell, you must
learn to become proficient with the pipe and redirection operators:
`|`, `>`, `>>`.

## Practicing searching and redirection

Finally, let's use the new tools in our kit and a few new ones to example our SRA metadata file.

```bash
$ cd ../sra_metadata/
$ ls
```

Take a look at the metadata file, `SraRunTable.txt`:

```bash
$ less SraRunTable.txt
```

Let's ask a few questions about the data

1) How many of the read libraries are paired end?

First, what are the column headers?

```bash
$ head -n 1 SraRunTable.txt
```

    BioSample_s	InsertSize_l	LibraryLayout_s	Library_Name_s	LoadDate_s	MBases_l	MBytes_l	ReleaseDate_s Run_s SRA_Sample_s Sample_Name_s Assay_Type_s AssemblyName_s BioProject_s Center_Name_s Consent_s Organism_Platform_s SRA_Study_s g1k_analysis_group_s g1k_pop_code_s source_s strain_s

That's only the first line but it is a lot to take in.  'cut' is a program that will extract columns in tab-delimited
files.  It is a very good command to know.  Lets look at just the first four columns in the header using the '|' redirect
and 'cut'

```bash
$ head -n 1 SraRunTable.txt | cut -f1-4
```

    BioSample_s InsertSize_l      LibraryLayout_s	Library_Name_s    

'-f1-4' means to cut the first four fields (columns).  The LibraryLayout_s column looks promising.  Let's look at some data for just that column.

```bash
$ cut -f3 SraRunTable.txt | head -n 10
```

    LibraryLayout_s
    SINGLE
    SINGLE
    SINGLE
    SINGLE
    SINGLE
    SINGLE
    SINGLE
    SINGLE
    PAIRED
    
We can see that there are (at least) two categories, SINGLE and PAIRED.  We want to search all entries in this column
for just PAIRED and count the number of hits.

```bash
$ cut -f3 SraRunTable.txt | grep PAIRED | wc -l
```

    2

2) How many of each class of library layout are there?

We can use some new tools 'sort' and 'uniq' to extract more information.  For example, cut the third column, remove the
header and sort the values.  The '-v' option for grep means return all lines that DO NOT match.

```bash
$ cut -f3 SraRunTable.txt | grep -v LibraryLayout_s | sort
```

This returns a sorted list (too long to show here) of PAIRED and SINGLE values.  Now we can use 'uniq' with the '-c' flag to
count the different categories.

```bash
$ cut -f3 SraRunTable.txt | grep -v LibraryLayout_s |	sort | uniq -c
```

      2 PAIRED
     35 SINGLE 

3) Sort the metadata file by PAIRED/SINGLE and save to a new file
   We can use if '-k' option for sort to specify which column to sort on.  Note that this does something
   similar to cut's '-f'.

```bash
$ sort -k3 SraRunTable.txt > SraRunTable_sorted_by_layout.txt
```

4) Extract only paired end records into a new file
   Do we know PAIRED only occurs in column 4?  WE know there are only two in the file, so let's check.

```bash
$ grep PAIRED SraRunTable.txt | wc -l
```

    2

OK, we are good to go.

```bash
$ grep PAIRED SraRunTable.txt > SraRunTable_only_paired_end.txt
```    

****
**Final Exercise**

1) How many sample load dates are there?

2) How many samples were loaded on each date

3) Filter subsets into new files based on load date
****

 


## Where can I learn more about the shell?

- Software Carpentry tutorial - [The Unix shell](http://software-carpentry.org/v4/shell/index.html)
- The shell handout - [Command Reference](http://files.fosswire.com/2007/08/fwunixref.pdf)
- [explainshell.com](http://explainshell.com)
- http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO.html
- man bash
- Google - if you don't know how to do something, try Googling it. Other people
have probably had the same question.
- Learn by doing. There's no real other way to learn this than by trying it
out.  Write your next paper in nano (really emacs or vi), open pdfs from
the command line, automate something you don't really need to automate.


## Bonus:

**alias** 

**.bashrc**

**ssh and scp**

**Regular Expressions**

**Permissions**

**Chaining commands together**

**md5sum**

## Useful Links (JAX)

* [Introduction to HPC Workshop Link](https://thejacksonlaboratory.github.io/introduction-to-hpc/)
* [Shell Novice Command List](https://gist.github.com/rizkaz/f87e88d7bf73a2a7fdcd58e0a7d4aafa#file-shell-novice-cheatsheet-md)


SOLUTIONS:

Exercise 1: Searching files
1) Search for the sequence GNATNACCACTTCC in SRR098026.fastq. 
In addition to finding the sequence, have your search also return the name of the sequence.

`grep -B1 GNATNACCACTTCC SRR098026.fastq`

    @SRR098026.245 HWUSI-EAS1599_1:2:1:2:801 length=35
    GNATNACCACTTCCAGTGCTGANNNNNNNGGGATG

2) Search for that sequence in both fastq files.

`grep -B1 GNATNACCACTTCC *.fastq`

    @SRR098026.245 HWUSI-EAS1599_1:2:1:2:801 length=35
    GNATNACCACTTCCAGTGCTGANNNNNNNGGGATG

Exercise 2: Practicing searching and redirection
1) How many sample load dates are there?

`cut -f5 SraRunTable.txt | grep -v LoadDate_s | sort | uniq`

    25-Jul-12
    29-May-14

2) How many samples were loaded on each date.

`cut -f5 SraRunTable.txt | grep -v LoadDate_s | sort | uniq -c`
  
    6 25-Jul-12
    31 29-May-14

3) Filter subsets into new files based on load date.

```
head -n 1 SraRunTable.txt > SraRunTable_25-Jul-12.txt
grep 25-Jul-12 SraRunTable.txt >> SraRunTable_25-Jul-12.txt
head -n 1 SraRunTable.txt > SraRunTable_29-May-14.txt
grep 29-May-14 SraRunTable.txt >> SraRunTable_29-May-14.txt
```

Lookahead : Using Loops - Next Section 

```
# Save the unique days in a variable
dates=`cut -f5 SraRunTable.txt | grep -v LoadDate_s | sort | uniq`
echo $dates

# Loop through dates to create sub samples
for date in $dates
do 
    echo "processing $date"
    head -n 1 SraRunTable.txt > SraRunTable$date.txt
    grep $date SraRunTable.txt >> SraRunTable$date.txt
done
 ```

