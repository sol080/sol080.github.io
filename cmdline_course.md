---
layout: default
---

## Introduction

Command Line Tools for Linguists is a course offered at the University of Helsinki for linguistic students to take during their undergraduate, graduate or post-graduate study. The course covers basic commands for Windows or Linux/MacOS system essential for research project in linguistics. During seven weeks, student learns to use terminal to deal with files and directories in general, to search/manipulate corpus (text files), to build automation programs, and finally to use Version Control for their projects. The learning process is solidly organised into  weekly online materials, quizzes, and in-person tutorials.

Here are some of my personal notes for each week of the course on things to remember and reflections on the quizzes (Week titles are taken from the course material)
 
## Week 1: Introduction to Command Line Environments

There are many ways to work with directories, with some important commands are:
- Print  the current directory with **pwd** and set it to another with **cd**
- Make a directory with **mkdir** and list its contents (including hidden files) with **ls -a**

Command line tools offer basic commands to deal with files in general, which can be complimented with an editor in the case of text files.
- Copy file(s) with **cp** (beware of overwriting when the new file name exists!), remove file(s) with **rm**, move or rename a file with **mv**
These commands can be used for directories by adding option **-r** to the command.
- Display a text file with **cat**, or **less**
These commands will open an application view, which can be quit using **q**, **ESC**, **Ctrl-c** or **Ctrl-d**)
- Usage of emacs for text files (Control key and Meta key are heavily used for shortcut operations). One can initialise a new text file or open existing one by emacs with this code below:
```
emacs <file-name.txt>
```


## Week 2: Navigating a UNIX System

In this week, we learnt about root directory and directory hierarchy in UNIX computers. One important directory is **_bin_**, which containing commands in binary files. bin can be located under root (for OS-required programs) or usr (for programs installed to help users). One can also check where the command they run is located using which command.

Another important topic is how to monitor, initial, move from foreground to background and vice-versa, and kill processes.
- List processes using **top**
- Move a process to background by adding **&** at the end of the command, or using **Ctrl+z**
- Move a process to forground again using **fg**
- Get pid of all running processes using **ps aux**, or search for specific ones using **grep** next to ps aux in a pipe line
- Kill a process using **kill -9 <PID>**

All basic commands in Week 1 and Week 2 can also be used in remote servers using ssh (secure shell) and file transfer between remote and local server can be done with scp.
```
scp <user_1>@<server_1>:<path_1> <user_2>@<server_2>:<path_2> .
```
**.** refers to current directory, which can also be altered to path to a directory if necessary. Without further settings, it is recommended to use scp when one's working on their local server.

## Week 3: Basic Corpus Processing

Getting into corpus processing, one needs to be aware of, and extract the corpus' metadata including  different encoding systems and number of lines, words, characters.
- The former can be tracked with  **file**, and converted with **iconv**
One example for iconv syntax
```
iconv -f <original-encoding> -t <target-encoding> <file-name> > <target-file-name>
```
- Windows and Unix system also differ to each other in usage of carriage returns, which can be shown with **less -U** and converted by **dos2unix**
- List of words in a corpus can be built by using **tr** for tokenisation, **sort** for alphabetical ordering, and **uniq** for removing duplicates
- **grep** or **egrep** are often used to find linguistically relevant information from a text file, which requires usage of **_regular expressions_**  

## Week 4: Advanced Corpus Processing

To manipulate a corpus, **sed** can be used a long with **egrep** and **tr** to form a **_pipeline_** (with **|** and **>**).
One examplary pipeline can be
```
cat life_of_bee.sent | sed -E 's/.* //' | tr -cd "A-Za-z0-9\n'" | sort | uniq -c | sort -nr > life_of_bee.sentend
```
Where a list of sentences in Life of Bee is converted into list of sentence-ending words. **sed** with extended regular expressions (flagged by **-E**) removes every word with a space after in a line, hence, leaves sentence-ending words with punctuations. **tr -d** means to delete and **-c "A-Za-z0-9\n'"**  means anything except the characters in "", which also points to deleting punctuations. **uniq -c** is used to count occurences of unique entries and returns lines in format <number-of-occurences> <word>, while **"sort -nr"** sort lines into decending order. The result is sent to a target file called life_of_bee.sentend with ** > **.

## Week 5: Scripting and Configuration Files

We might want to, manipulate several text files simutanously in the same manner we did in week 3 and week 4. Here is where scripts (.sh files) coming in, with concepts such as parameters(i.e input and output files) and variables (marked by **$<number>** in scripts). Other important syntaxes are **if** and **exit**. Script files can be moderated by **chmod** and run with **bash** or **./**.

Example: A script to turn adjectives into their comparative forms

```
#! /bin/bash
# script: comparative.sh
# Author: Duong Nguyen (Sept 2022)
#

FINAL_LETTER=`echo $1 | sed -E 's/.*(.)/\1/'`
STEM=`echo $1 | sed -E 's/(.*)./\1/'`

if [ "$FINAL_LETTER" == "y" ] 
then
    echo "${STEM}ier"
elif [ "$FINAL_LETTER" == "e" ]
then
    echo "${STEM}er"
else
    echo "$1er"
fi
```

The PC local system also contains several environment variables, which are used in shell/bash configuration files for specific settings.

## Week 6: Installing and Running Programs

Multiple scripts can be turned into a program for specific purposes. One can either install existing programs or softwares or write their own program. For linguists, **_Python_** with its **_nltk_** library (and many others!) are powerful toolsfor handling corpus, which can be installed using **homebrew** in Mac OS. It is recommended to create and maintain a database of installed softwares, and use **locate** to monitor which files are added and where in our system. For Python, it is recommended to have one **_virtual environment_** per project, where libraries and programs settings can be customised according to user's needs. Finally, one can handily write a **_Makefile_** compiling multiple scripts and implement it as a program.

Example: a Makefile to process noels into one corpus, count word frequency and list all sentences

```
BOOKS=alice christmas_carol dracula frankenstein heart_of_darkness life_of_bee moby_dick modest_propsal pride_and_prejudice tale_of_two_cities ulysses

NOMD=$(BOOKS:%=data/%.no_md.txt)
FREQLISTS=$(BOOKS:%=results/%.freq.txt)
SENTEDBOOKS=$(BOOKS:%=results/%.sent.txt)

all: data/all.no_md.txt data/all.freq.txt data/all.sent.txt

%.no_md.txt: %.txt
	python3 src/remove_gutenberg_metadata.py $< $@

results/%.freq.txt: data/%.no_md.txt 
	src/freqlist.sh $< $@

results/%.sent.txt: data/%.no_md.txt
	src/sent_per_line.sh $< $@

data/all.freq.txt: $(FREQLISTS)
	cat $^ > $@

data/all.sent.txt: $(SENTEDBOOKS)
	cat $^ > $@

data/all.no_md.txt: $(NOMD)
	cat $^ > $@
```

## Week 7: Version Control

Version control is a common practice in workflows of development projects, to monitor changes in a file (i.e. a program file), to let we undo modifications where needed. It includes having branches (as drafts) to work on until it can be merged into the original file, which can also be tracked and reversed. **_git_** is a technology for version control and **_Github_** is popular with teams of many members developing programs.

Some useful git commands are:
| Command              	   | Purpose				           |
| ------------------------ |:---------------------------------------------:|
| git init		   | initialise a repository from existing code    |
| git status    	   | see staging area and untracked files	   |
| git add -A    	   | add all files in staging area	 	   |
| git reset		   | remove all files from staging area		   |
| git commit -m"<message>" | commit the files in staging area with message |
| git log    		   | see the last commit			   |
| git clone		   | clone a remote repository			   |
| git remote -v		   | view information of a remote repository	   |
| git branch -a 	   | view all remote and local branches of a repo  |
| git diff   		   | view changes in the code  		     	   |
| git push <remote> <local>| push commit(s) from local to a remote repo	   |
| git pull <remote> <local>| pull changes of remote repo to local repo 	   |
| git branch <branch-name> | create a branch				   |
| git checkout <branch>	   | start working on a branch			   |
| git merge <branch>	   | merging branch to the main/master branch	   |
| git branch --merged	   | view merged branch	   	       		   |
| git branch -d <branch>   | delete a local branch			   |

