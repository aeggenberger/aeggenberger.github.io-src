Title: Is d3 awk for the web?
Date: 2016-9-5 12:00
Category: Thinking about working with data
Tags: d3, awk, gawk, data processing
Slug: d3-awk-for-web

Maybe! I don't know.

But there are some similarities that helped me make the leap from the (my?)
usual procedural javascript style to the more declarative, functional style
embraced by d3.

## 'Data Driven'

Let's begin with awk's concept of records. These are the discrete chunks of data
on which awk operates. At its most basic, awk goes through records one by one,
optionally printing all or part of the record, transforming the record, updating
a state variable to be accessed later -- all kinds of things. This is analogous
to the d3 concept of data. In d3, data is [bound][1] to the DOM. Once the enter
method is called, d3 iterates through the data in discrete chunks and does what
it will with it.

In both instances, it's the programmer's job to describe the data they want want
to work with and what should happen when it's found, rather than set up a whole
bunch of scaffolding and boilerplate code (for loops, etc.) to process the data.
The documentation for both [gawk][2] and [d3][3] describe this approach as "data
driven." D3 itself is sort of but not quite an initialism for "Data-Driven
Documents."

## Similar philosophy, different use cases

It took me years of procedural hacking before I realized the usefulness of
declarative syntax and functional programming. Now I use them whenever I can.
Maybe someday I'll be able to use them whenever they are appropriate. The model
is flexible enough that it can be used at different ends of my data pipeline.
For instance, when I was creating a database table with a column for all the
different types of police calls in Lake City, I didn't have a master list to
work with. Instead I had years worth of weekly reports. Some calls, like
"traffic violations," appear virtually every week. Others, like homicide, are
extremely rare. Using a very short awk script, I was able to scan every file and
return only the unique call types. As a bonus, I was able to format the output
as a valid sql query that I ran against my database. I might use awk again to
generate insert queries, one per week, into the table. 

Once I have my data, I'm going to want to present it to our readers. That's
where d3 and its heavy focus on presentation comes in.

[1]:http://alignedleft.com/tutorials/d3/binding-data
[2]:https://www.gnu.org/software/gawk/manual/gawk.html#Getting-Started
[3]:https://d3js.org/
