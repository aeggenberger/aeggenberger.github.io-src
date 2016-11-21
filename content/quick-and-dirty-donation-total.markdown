Title: Quick and dirty data extraction with grep and friends 
Date: 2016-11-21 15:31
Category: Command-line
Tags: filthy, nasty, command line, awk, gawk, tr, grep
Slug: data-cleanup-on-one-line

Going over an agenda before an upcoming school board meeting I noticed there are
a lot of donations totalling a lot of money. But how many are there and for how
much money, really? Before learning about unix command line utilities I would
have said it's too many to tell. I mean, just look at this mess:

<pre>
E.    Approve Schafer Family Foundation $50,000 Donation for repair to
Football Field and / or Equipment (Approve)    


Motion___________________                 Second___________________
    
F.      Approve Football Jersey $1,176 Donation from Sean & Natalie McElmury   
      (Action).


Motion___________________                 Second___________________


G.        Approve 9th Grade Retreat Donations. (Action).
Alliance Bank- $50            Lake Pepin Veterinary Clinic- $200
Ripley Dental- $100            Lake City Women’s Club- $200
Mahn Funeral Home- $100        Lake City Family Physicians- $100
Acrotech- $1000            Schleicher Funeral Home- $25


Motion___________________                 Second___________________


    
H.       Approve Eagle Bluff Donations. (Action).
    
Acrotech- $2000            Lake City Sportsman’s Club- $500
    Ardent Mills- $2000            Frontenac Sportsman’s Club- $3000


Motion___________________                 Second___________________


    
I.    Ag Department Ford 8N Tractor Donation by Dennis Webber.  Donated to Ag
         Mechanics for repair, restoration, and resale / fundraiser. (Action)


Motion___________________                 Second____________________

</pre>

It's somewhat orderly, sure. And, yeah, I could use a calculator or spreadsheet
to answer both my questions. But that wouldn't feel like magic. So here's how,
after some trial and error, I was able to do it with a one-liner in iTerm2
running on OSX.

## Step one: pbpaste

In OSX, the system pasteboard is accessible from the command line using [pbcopy
and pbpaste][1]. All we need to do to get the raw text into terminal was to copy
from its original location (a browser window in this case) and then use pbpaste.

<code>pbpaste</code>

## Step two: grep

Grep has spent the last 42 years single-mindedly carrying out its mission:
[to "print lines matching a pattern"][plmp]. After reading a bit of the [Wikipedia
entry][wi] I think this post is doubly indebted to this utility. Firstly, and
most obviously, I used it to accomplish what I wanted to do. The other reason is
that, according to at least on commentator, I have grep to thank for
"irrevocably ingraining" the unix convention of [small, composable tools][sct]
that is the basis of this post.

Grep was the first tool that came to mind when I started thinking about ways to
get the numbers out from among all those letters and symbols and spaces. But I
was skeptical. It says right on the can that grep prints "lines". The donation
data at times has multiple comments, meaning the amounts for more than one
donation sometimes end up on the same row. On the [man page][plmp] I found the
-o option with the following description:

> Print only the matched (non-empty) parts of a matching line, with each such
> part on a separate output line.

The next step is creating a pattern to match all the numbers and none of the
non-numbers. We'll use the -E option (for extended regex) with the pattern
'[0-9]{1,6},*[0-9]{1,6}'. The comma in the middle was required because only some
of the donations of at least four figures use commas.

<code>grep -Eo '[0-9]{1,6},*[0-9]{1,6}'</code>

## Step three: tr

It would have been simpler if there were no commas in our data but, to
paraphrase a famous general(?), we go into data analysis with the data we have.
Our data has some commas. We have to accept that. The problem now, to paraphrase
a fake Mahatma Ghandhi quote, is how to get our data to be the data we want to
analyze in the world. When you think about it, that's the problem of this entire
post, but I digress. One answer to this dilemma is the utility [tr][tr]. tr can
"translate or delete characters." We're going to ask it to delete our commas
using the -d argument.

<code>tr -d ','</code>

## Step four: awk

Once we have the data in a format we can use, our next step is to use the data
to answer the questions that we have. For this we turn to [awk][awk]. The
following short awk program will add each line to the total of all the previous
lines: <code>{ sum+=$1}</code>. After adding up all the lines, we want to print
the total, the final value of the variable <code>sum</code>. We'll accomplish
this by appending an <code>END</code> block with a print statement and a comma
separate list with two items, to values of <code>sum</code> and the built-in
variable <code>NR</code> (aka the number of donations).

<code>awk '{ sum+=$1} END {print sum, NR + 1}'</code>

## All together now

### The command

<code>pbpaste | grep -Eo '[0-9]{1,6},*[0-9]{1,6}' | tr -d , | awk '{ sum+=$1}
END {print sum, NR}'</code>

### The result

<pre>60451 14</pre>

So that's it, then: 14 donations totaling $60,451. Would it have been faster to
use a spreadsheet or even a calculator or pen and paper? Maybe. (The answer is
"definitely" if you include the time it took to compose a blog post about it.)
But imagine if you were looking at pages and pages of this kind of information
and it's easy to see how this sort of thing might prove useful, at least to get
a quick estimate.

[1]: https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/pbpaste.1.html
[wi]: https://en.wikipedia.org/wiki/Grep#History
[sct]: https://en.wikipedia.org/wiki/Unix_philosophy
[plmp]: https://linux.die.net/man/1/grep
[tr]: https://linux.die.net/man/1/tr
[awk]: https://linux.die.net/man/1/awk
