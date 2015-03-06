## "UNIX philosophy"[wikipedia](https://en.wikipedia.org/wiki/Unix_philosophy)

* small, sharp tools
* that can be chained together 
* to solve complex problems
* goes back to the introductino of the pipe and grep in 1973

where simple means: providing well thought out functionality,
implemented to perfection (see [grep algorigthm and implementation](http://lists.freebsd.org/pipermail/freebsd-current/2010-August/019310.html))

## before we do anything

* use the up-arrow (and down-arrow) to go back to previouse commands
* use the tabulator key for file name completion:  type just the first letter of a filename, then try tab
* use CTRL-A to move your cursor to the beginning of the line
* use CTRL-E to move your cursor to the end of the line
* use CTRL-K to delete from the cursor to the end of the line

## What kind of file is that?

first, let's look at the size:

    $ ls -l parole 
    -rw-r----- 1 bjelline admin 2286089 Feb 25 12:13 parole
    $ ls -lh parole 
    -rw-r----- 1 bjelline admin 2.2M Feb 25 12:13 parole


you don't know what might be in the file?
guess the file type:

    $ file parole 
    parole: ASCII text

if it's text, we can take a peek with less.

    $ less parole

use key commands in less:

  * q to quit
  * SPACE to page one down
  * CTRL-B to go back one page
  * CTRL-F to go back one page
  * G to jump to the end
  * CTRL-G to find out where you a (line number + percent)
  * 50% jump to middle of file (works for any number 0-100)
  * /cookie to search for cookie
    * n to jump to the next occurance
    * b to jump back to the previous occurance

less is used in many other command line tools, e.g. in man.
it pays to learn these key commands!

How many lines are there?

    $ wc -l parole
    24291 parole

See also man wc

## rename and move stuff around

now we know that 'parole' is a csv file
let's rename it:

    $ mv parole parole.csv

See also man mv

## find certain lines, get rid of certain files

grep will look for a string in each line, and print
the line if the string is contained.  

   $ grep TRAMA parole.csv

If there is a space in your search string you must quote it:

   $ grep 'JAMES, MARK' parole.csv

grep is case sensitive, use -i to make it case insensitive:

   $ grep -i trama parole.csv

the option v turns grep around: all lines are printed, except those
matching the pattern.  this way you can get rid of certain files

   $ grep -v ....

to build more comlex search, with search patterns, use egrep

   $ egrep 'TRAMA|TENNEY' parole.csv

See also man grep

## sorting

    $ sort names.txt


## count, top-n

    $ sort names.txt | uniq -c | sort -r


## redirection 

Here we use the greater than sign for output-redirection to a file
This will overwrite the file if it already exists, so be careful when to use it!

    $ grep DENIED parole.csv > parole-denied.csv

if you want to add to an existing file use double greater than signs:

    $ grep GRANTED parole.csv  > parole-granted.csv
    $ grep PAROLED parole.csv >> parole-granted.csv

This way you can get any program to writ to a file.

How does this work?  The UNIX command line tools send their output
to a "channel" called "standard output" (or STDOUT, stdout in different programming languages).
the redirection is the general way of sending this channel to a file.

See also man bash, search for REDIRECTION

## pipeline

How many people were denied parole?  we just
need to count the lines:

    $ grep DENIED parole.csv > parole-denied.csv
    $ wc -l parole-denied.csv

But there is no need to creat the file at all: we can creat a pipeline
with the vertical bar symbol. The pipeline connects the output
channel of grep to the input channel of wc:

    $ grep DENIED parole.csv | wc -l 

The vertical bar is often called "pipe", so you would read
this command as: grep denied parole.csv pipe wordcount minus L

Using the pipe you can create complex programs without writing any permanent code.

When using longer pipes, always put "| less" at the end to take a peek.

## cutting columns from files

cut, paste


## handling csv files

There are the commands 'cut' and 'paste' that you might want to
use to handle comma-separated-values files.  But beware: as soon as
you have commas inside one of the columns cut will not work for you!

Here the csv-tools come in: csvcut knows about strings and escaping
in csv and will cout out the correct columns:

    $ csvcut -c1,2 parole.csv

Now we can use the power of pipelines to learn more about the single factors:



## compressed files

search inside *.zip or *.gz files with zgrep, zegrep


## using csvcut and other stuff

    $  csvcut -c3 parole.csv |cut -c1-2| 

    $ csvcut -c3 parole.csv |cut -c1-2| awk '{ if (int($0) > 20) print "19$0"; else print "20$0";'

    $ echo "YEAR INCARCERATION" > year_of_incarceration.txt
    $ csvcut -c3 parole.csv |cut -c1-2| grep -v DI|  perl -n -e 'if($_>16) { print "19$_" } else { print "20$_" }' >> year_of_incarceration.txt

    csvcut -c5 parole.csv | cut -d\/ -f 3 | sed -e 's/DATE/YEAR/' |head -10 > birth_year.txt

    csvcut -c11,12 parole-3.csv | perl -n -e '($in,$birth) = split /,/; if($.==1){print "AGE AT INCARCERATION\n"} else { print $in-$birth, "\n"}' > age_at_incarceration.txt

file:///Users/bjelline/Dropbox/ScrapBook/data/20130124145714/index.html

    tr -cs A-Za-z '\n' |
    tr A-Z a-z |
    sort |
    uniq -c |
    sort -rn |
    sed ${1}q

McIlroy’s short, impossible-to-misunderstand explanation:

If you are not a UNIX adept, you may need a little explanation, but not much, to understand this pipeline of processes. The plan is easy:

* Make one-word lines by transliterating the complement (-c) of the alphabet into newlines (note the quoted newline), and squeezing out (-s) multiple newlines.
* Transliterate upper case to lower case.
* Sort to bring identical words together.
* Replace each run of duplicate words with a single representative and include a count (-c).
* Sort in reverse (-r) numeric (-n) order.
* Pass through a stream editor; quit (q) after printing the number of lines designated by the script’s first parameter (${1}).