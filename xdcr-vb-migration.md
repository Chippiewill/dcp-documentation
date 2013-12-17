## Overview

This design doc is to explain possible optimization in XDCR when vbucket (partition) migration
happens due to failover or rebalance.


## Problem: unoptimized vbucket (partition) replicator restart on migration

A bit background on how XDCR works. Today each vbucket has its own replicator responsible for replicating all mutations in that vbucket, on their arrival order.  But during topology change, which can be either source and destination, vb replicator will stop or crash on different errors such as not_my_vb as designed, at the old host node, and restart itself at new host node after migration. In particular there would be two scenarios:

##### topology change at source.  
Say, vbucket 123 migrated from node N1 to N2 because this topology change. A new vbucket replicator process (Erlang thread) will start at N2, today we do not replicate checkpoint files to replica within cluster thus the replicator of vb 123 at N2 will start from scratch. It will start from mutation seqNo = 0 and scan all mutations by either sending getMeta to remote side (non-optimistic XDCR), or sending setWithMeta with the doc directly (optimistic XDCR). After scanning all mutations already replicated, the new vb replicator will reach the point to resume replication at N2. In this process, you may see Metadata operations (non-optimstic XDCR) at destination side. If CAPI mode XDCR is used, you will see metadata opertaions even for optimistic XDCR. Please note that,  in optimistic XMem mode, even you do not see anything on UI at destination. There are a bunch of setWithMeta/delWithMeta sent to the destination. However since they are all rejected by remote ep_engine, we would not see them on UI. But you can query that by looking at ep_engine stats. Namely,

- ep_num_ops_set_meta_res_failed: Number of setWithMeta ops that failed
- ep_num_ops_del_meta_res_failed: Number of delWithMeta ops that failed

##### topology change at destination.
Similar to 1), old replicator will crash because of `not_my_vb` errors returned to source during topology change at destination.  The new vb replicator will be started 30 second later by default, which is configurable. If at this time the topology change at destination is ready and source cluster gets the updated vb map, a new vb replicator will be able to replicate to the new host node at destination cluster.  Again, since we do not have checkpoint files replicated to replica at the destination cluster, the new vb replicator at source need to scan from scratch by sending a bunch of getMeta or setMeta operations to remote node, which is similar to 1).





*emphasize* or _emphasize_ ( Cmd + I )

**Sometimes I want a lot of text to be bold.
Like, seriously, a _LOT_ of text**

#### Blockquotes

> Right angle brackets &gt; are used for block quotes.

#### Links and Email

An email <example@example.com> link.

Simple inline link <http://chenluois.com>, another inline link [Smaller](http://smallerapp.com), one more inline link with title [Resize](http://resizesafari.com "a Safari extension").

A [reference style][id] link. Input id, then anywhere in the doc, define the link with corresponding id:

[id]: http://mouapp.com "Markdown editor on Mac OS X"

Titles ( or called tool tips ) in the links are optional.

#### Images

An inline image ![Smaller icon](http://smallerapp.com/favicon.ico "Title here"), title is optional.

A ![Resize icon][2] reference style image.

[2]: http://resizesafari.com/favicon.ico "Title"

#### Inline code and Block code

Inline code are surround by `backtick` key. To create a block code:

	Indent each line by at least 1 tab, or 4 spaces11u.
    var Mou = exactlyTheAppIwant; 

####  Ordered Lists

Ordered lists are created using "1." + Space:

1. Ordered list item
2. Ordered list item
3. Ordered list item

#### Unordered Lists

Unordered list are created using "*" + Space:

* Unordered list item
* Unordered list item
* Unordered list item 

Or using "-" + Space:

- Unordered list item
- Unordered list item
- Unordered list item

#### Hard Linebreak

End a line with two or more spaces will create a hard linebreak, called `<br />` in HTML. ( Control + Return )  
Above line ended with 2 spaces.

#### Horizontal Rules

Three or more asterisks or dashes:

***

---

- - - -

#### Headers

Setext-style:

This is H1
==========

This is H2
----------

atx-style:

# This is H1
## This is H2
### This is H3
#### This is H4
##### This is H5
###### This is H6


### Extra Syntax

#### Footnotes

Footnotes work mostly like reference-style links. A footnote is made of two things: a marker in the text that will become a superscript number; a footnote definition that will be placed in a list of footnotes at the end of the document. A footnote looks like this:

That's some text with a footnote.[^1]

[^1]: And that's the footnote.


#### Strikethrough

Wrap with 2 tilde characters:

~~Strikethrough~~


#### Fenced Code Blocks

Start with a line containing 3 or more backticks, and ends with the first line with the same number of backticks:

```
Fenced code blocks are like Stardard Markdown’s regular code
blocks, except that they’re not indented and instead rely on
a start and end fence lines to delimit the code block.
```

#### Tables

A simple table looks like this:

First Header | Second Header | Third Header
------------ | ------------- | ------------
Content Cell | Content Cell  | Content Cell
Content Cell | Content Cell  | Content Cell

If you wish, you can add a leading and tailing pipe to each line of the table:

| First Header | Second Header | Third Header |
| ------------ | ------------- | ------------ |
| Content Cell | Content Cell  | Content Cell |
| Content Cell | Content Cell  | Content Cell |

Specify alignement for each column by adding colons to separator lines:

First Header | Second Header | Third Header
:----------- | :-----------: | -----------:
Left         | Center        | Right
Left         | Center        | Right


### Shortcuts

#### View

* Toggle live preview: Shift + Cmd + I
* Toggle Words Counter: Shift + Cmd + W
* Toggle Transparent: Shift + Cmd + T
* Toggle Floating: Shift + Cmd + F
* Left/Right = 1/1: Cmd + 0
* Left/Right = 3/1: Cmd + +
* Left/Right = 1/3: Cmd + -
* Toggle Writing orientation: Cmd + L
* Toggle fullscreen: Control + Cmd + F

#### Actions

* Copy HTML: Option + Cmd + C
* Strong: Select text, Cmd + B
* Emphasize: Select text, Cmd + I
* Inline Code: Select text, Cmd + K
* Strikethrough: Select text, Cmd + U
* Link: Select text, Control + Shift + L
* Image: Select text, Control + Shift + I
* Select Word: Control + Option + W
* Select Line: Shift + Cmd + L
* Select All: Cmd + A
* Deselect All: Cmd + D
* Convert to Uppercase: Select text, Control + U
* Convert to Lowercase: Select text, Control + Shift + U
* Convert to Titlecase: Select text, Control + Option + U
* Convert to List: Select lines, Control + L
* Convert to Blockquote: Select lines, Control + Q
* Convert to H1: Cmd + 1
* Convert to H2: Cmd + 2
* Convert to H3: Cmd + 3
* Convert to H4: Cmd + 4
* Convert to H5: Cmd + 5
* Convert to H6: Cmd + 6
* Convert Spaces to Tabs: Control + [
* Convert Tabs to Spaces: Control + ]
* Insert Current Date: Control + Shift + 1
* Insert Current Time: Control + Shift + 2
* Insert entity <: Control + Shift + ,
* Insert entity >: Control + Shift + .
* Insert entity &: Control + Shift + 7
* Insert entity Space: Control + Shift + Space
* Insert Scriptogr.am Header: Control + Shift + G
* Shift Line Left: Select lines, Cmd + [
* Shift Line Right: Select lines, Cmd + ]
* New Line: Cmd + Return
* Comment: Cmd + /
* Hard Linebreak: Control + Return

#### Edit

* Auto complete current word: Esc
* Find: Cmd + F
* Close find bar: Esc

#### Post

* Post on Scriptogr.am: Control + Shift + S
* Post on Tumblr: Control + Shift + T

#### Export

* Export HTML: Option + Cmd + E
* Export PDF:  Option + Cmd + P


### And more?

Don't forget to check Preferences, lots of useful options are there.

Follow [@chenluois](http://twitter.com/chenluois) on Twitter for the latest news.

For feedback, use the menu `Help` - `Send Feedback`