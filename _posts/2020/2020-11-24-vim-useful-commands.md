---
layout: post
title: VIM Useful Commands
category: tool
tags: [tool]
excerpt: VIM Useful Commands
---

## Inserting Text  

```
a: Append text after the cursor.  
i: Insert text before the cursor.
o: Begin a new line below the cursor and insert text.
```


## Deleting a Text  

```
x :  Delete character under and after the cursor.
dd: Delete the current line.
d0: Delete everything from where your cursor is to the beginning of the line.

```



## Substituting  

```
:%s[ubstitute]/{search_text}/{replace_text}/<command_flag>
[eg]: :%s/YYC/yyc/gc
c: Confirm each substitution.
i: Ignore case for the pattern.
g: Replace all occurrences in the line. Without this arugument, 
   replacement occurs only for the first occurrence in each line.
```



## Copying and Moving Text  

```
yy: Yank the current line [into register x]. 
p : Put the text [from register x] after the cursor.
```



## Undo/Redo/Repeat  

```
u: undo changes.
.: Repeat last change.
```



## Moving Around  

```
    k
h       l
    j
0    : To the first character of the line (exclusive).
Home : To the first character of the line (exclusive).
^    : To the first non-blank character of the line.
$/End: To the end of the line.
gg   : places the cursor at the start of the file.
G    : Places the cursor at the end of the file.
123G : Go to the specific line number.
w    : Places the cursor at the next word.
```



## Searching  

```
/{pattern}: Search forward for the occurrence of {pattern}.
```



## Useful    

```
guu: lowercase line.
gUU: uppercase line.
```



## Reference  

- [Vim Commands Cheat Sheet](http://www.fprintf.net/vimCheatSheet.html){:target="_blank"}    
