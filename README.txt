What we did:
————————————
1. We only want Black Dogs of the British Isles. (Sorry, Anubis). So:
   - Go to Wikipedia: https://en.wikipedia.org/wiki/Black_dog_(ghost)
   - Copy everything between "England" and the end of "Channel Islands and Isle
     of Man"
   - Paste that into Atom or another plain text editor (not Word!) This strips
     formatting from the web page removes URLs, etc.
   - Save this file as "source-wikipedia.txt" so we remember what it is.



2. Wikipedia includes a lot of text in square brackets.[citation needed]
   (Honestly, I'm a bit surprised there isn't more [citation needed] on this page.
   If anyone likes editing Wikipedia and wants to prompt other editors to look
   for primary sources for some of this stuff, have at it.)



We don’t care about [citation needed], so:

- in Terminal:

sed -e 's/\[[^][]*\]//g' source-wikipedia.txt >> source-wikipedia-nobrackets.txt

This generates a new file called "source-wikipedia-nobrackets.txt" that removes
all the Wikipedia UI stuff: footnotes, [citation needed], and so on.

(If you're on Windows, or you want to do this in Atom, there’s a bunch of plugins
you can use, or you can just do it manually if you’re dealing with a short source file.



3. We want single sentences for our bot. So put everything on its own line.
   In Atom, do a find-replace for:

". "

with

".
"

This will put each sentence on its own line.

3a) Oh hey, now you can tell that there's some lines that start with bullets.
Let's also get rid of them. In Atom, just find-replace again:

"  • "

to

""


4. Now we need to extract all the sentences that say something interesting.
Go back to wikipedia and skim the article for declarative phrases and then look
for what they have in common, writing them down.

It looks like a lot of these are saying "the such-and-such dog was said to..."
and so on.


5. Let's copy every sentence that includes those phrases.

Using Terminal:
grep -i 'is said\|was said\|has been\|described as\|the form\can be\|anyone\|someone\|person\|omen\|when' source-wikipedia-each-sentence-on-own-line.txt > output.txt

You can specify as many or as few phrases or individual terms as you want.
You just have to remember to put them between single quotes and separate them
by: \| (backslash, pipe.)

grep is looking for instances of these exact phrases, so "is said to" will match
"it is said to be seen by…" but will not match "it is said that the…"

Tip 1: open output file in Atom and run different commands to see the list change.

Tip 2: the -i makes the grep command case-insensitive; by default, grep will not
match "It is" if you search for "it is".



6. This is a pretty good list of declarative statements. Let's start with this.
To convert this into a JSON file that Tracery can read, do some more find-replace:

Including the line returns, replace:

".
"

with:

"
    ,
    "

We can then add an opening quote to the first line and remove the last trailing
comma manually. This is getting close to being proper JSON, but it isn't there yet.

Let's set this up to be a new Tracery symbol called "facts".
Turn on JSON syntax highlighting… and it looks good!

But there's some red flags here. It looks like we have some "unescaped" quotes.
Because we're surrounding each of these facts with quotes, the ones that
themselves contain quotes will break. Let's manually escape those by putting a
backslash in front of them \" .

Tip: syntax highlighting will tell you what you're doing wrong!


7. Let's test it out!

7a) In Atom, select your entire JSON file and copy it.
7b) Open the Tracery tester here: http://www.brightspiral.com/tracery/
7c) Switch the Tracery tester to JSON mode, and paste your code in. If your
    syntax is correct, it'll automatically generate 5 random examples on the
    right side of the screen.



Troubleshooting:
————————————————
1. Make sure you don't see anything amiss in your text editor. Is it a big sea
   of red? Let's start there!
2. If you don't have any syntax highlighting, or it looks weird, make sure
   that Atom is set to read JSON files.
3. If it looks fine, let's check that it's valid JSON using a JSON validator
   like https://jsonlint.com/
4. If it's valid JSON, are you sure you have an origin symbol?
5. If the origin symbol exists, does it actually use expansion symbols? And do
   they point to real symbols that exist? Did you do the thing I always do
   where you have a symbol called "names", plural and your origin uses
   "#name# went to the store", singular?





Next steps:
———————————
As-is, this grammar is great if you're making something like the Women of
Bletchley (@BletchleyRoll) account: Each of its tweets features a different
woman who worked at Bletchley Park during World War II, drawn from the official
Bletchley Park Roll of Honour: https://bletchleypark.org.uk/roll-of-honour

It includes everyone from Bombe operator Audrey Abbott to Anne Zuppinger,
Bombe operator recruiter and trainer.

For our purposes, we want this to be more of a remix:
"The such-and-such hound of [place] is said/was seen/etc."

We can generate these based on individual phrases within this source material,
but we'll have to break them up, and that requires manual work.

As a first step to categorizing phrases, let's change the "fact" symbol to
instead contain expansion symbols to two new symbols: one of full statements
("The black dog of Devon is said to appear at noon.") and one containing
fragments: "It is a heckin good pupper #yolo."






Types of statements about black dogs:
————————————————
1. Folklore:
  - "A black dog shows up whenever…"
  - "If you see the black dog…"
  - "On the full moon, it is said…"

2. Specific events:
  - "In 1667, the Earl of Lancashire was bitten by a black dog on his birthday and then died"

3. Some guy was coming home from the pub and saw something or maybe didn't:
    - "The Skriker (or Shrieker) […] also wanders invisibly in the woods at night uttering loud, piercing shrieks."

4. Things that aren't actually weird or supernatural in the first place:
  - "Dogs who are allowed to stray in this area late at night have often mysteriously disappeared."
  - "A man was journeying along a lonely forest road at night when a large black dog appeared at his side and remained there until the man left the forest."
