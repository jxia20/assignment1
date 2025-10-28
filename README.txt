CSCI-6500/4500 Distributed Computing over the Internet
Programming Assignment #1
Fall 2025
Overview

This project implements a MapReduce engine in Pict and applies it to two problems:
A Substring search over John Milton's sonnet "On His Blindness"
and Document statistics such as characters, words e.t.c.

The code is organized into:
  List/util helpers (append, reverse, sort, group, etc.)
  Generic MapReduce pipelines
  Problem 1: substring search ("is")
  Problem 2: text statistics

We define `inputS` as well, which is the stored sonnet.

Part 1: MapReduce Engine

In our solution, we factor this into reusable helpers:

  flatMapPairs / flatMapPairsS
  Apply a user-supplied map function over each pair in the input list,
  collect and concatenate the results.

  msortPairs / msortPairsS
  Merge-sort a list of key/value pairs by the key.
  For Int keys we use compareInt.
  For String keys we use compareWordLen

  groupByKey / groupByKeyS
  Input: a list of key/value pairs that is already sorted by Key.
  Output: a list of Key/(List Value), where all values for the same key
  are grouped together in order.

  flatMapGroups / flatMapGroupsS
  Apply the reduce function to each Key/(List Value)
  group and concatenate the results.

Using these, we implement:

  mapReduce  [#V #W mapF cmp reduceF input q]
  mapReduceS [#V #W mapF cmp reduceF input q]

Part 2: Substring Search
Given a search string, such as "is" in our case, return words that contain that substring, 
along with the line numbers in which they appear. 
Sort words by increasing length, and for ties, sort lexicographically.

Our pipeline for this problem:

mapSearch:
  Input: [lineNumber lineText]
  Split the line on delimiters: ",;:.!?\t\n"
  Ignore empty tokens
  Lowercase each token
  Check if the original token contains the target substring "needle"
  For each match, emit wordLowercase/lineNumber

  This gives us many pairs like ["his" 10], ["his" 11], ["his" 11], ["his" 12], etc.

compareWordLen:
  Custom String compare used for sorting: first sort by length, then sort lexicographically

groupByKeyS:
  After sorting, we group identical words. The output looks like:
    ["his" (cons > 10 11 11 12 nil)]

reduceKeepLines:
  For each grouped pair word/(List lineNumbers), 
  we sort the list of line numbers and wrap it back up as a singleton list.
  We keep duplicate line numbers if a word appears multiple times on that same line.

Printing:
   printGroupsS / printGroupSOneShot format the final result as:
     word (count) : lineNumber followed then by a newline.

We lowercase all words, so "Is" and "is" are treated as the same word.
We strip punctuation by splitting, so "need," and "need" are the same word.
We consider a match if the substring appears anywhere in the word

Part 2: Readability Stats
We seek to compute
  "Characters" (total characters including spaces),
  "Characters per Word",
  "Words",
  "Words per Line"
for the full sonnet.

Our pipeline for this problem:

mapStats:
  For each line in the poem:
  chars is the total characters in the raw line (string.size!), which includes spaces and punctuation.
  words is the number of nonempty tokens after splitting on punctuation and whitespace.
  tchars is the total characters across just the word tokens excluding spaces/punctuation
  1 is the number of lines contributed by this entry.

We emit a single pair: stats/(chars words tchars 1)

2. We then group everything under key "stats" using groupByKeyS.

reduceStats:
  sum all (chars words tchars 1) across all lines into totals:
    characters spaces included,
    word count,
    total token characters,
    line count.
  Compute:
    Characters = characters
    Characters per Word = token chars / words
    Words = words
    Words per Line = words / lines

  Emit these as a list:
    (cons > ["Characters"]
            ["Characters per Word"]
            ["Words"]
            ["Words per Line"]
    nil)

printLabelFloatList prints the output as we expect

Behavior:
  Characters counts spaces (matches assignment spec).
  Characters per Word" uses only token characters divided by word count so spaces/punctuation are ignored.
  We currently convert integer-like values to Float before printing

Our Helper Functions
We implemented a number of helper functions:
  append / rev / length
  merge sorts for Ints and key/value pairs
  helpers for grouping
  printers for debugging
  clampNonNeg, sub, toLowerStr to slice subtrings safely without crashes

Limitations or Notable deviations:
  We have two mapReduce variants. One for Int keys and one for String keys, instead
  of a singular mapReduce that can account for both.

  We cast integers to float for consistency when reducing stats. This can lead to
  characters printing as a floating point string but the values still match

Outputs:
  Problem 1:
  With needle = "is", on the Milton sonnet lines,
  the pipeline finds matches like "is", "his", "this", etc.,
  sorts them by word length with shortest first, and prints line numbers.

  Problem 2:
  The stats pipeline reproduces the expected overall structure:
    Characters
    Characters per Word
    Words
    Words per Line
  using the entire 14-line sonnet as input.
