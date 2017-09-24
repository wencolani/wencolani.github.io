---
layout: post
title: Vim Command
date: 2017-09-24 15:32:24.000000000 +09:00
---

learning book: file:///Users/zhangwen/Downloads/vim-1.0.pdf
current stage: Chapter 3 --- page 59  

vimtutor --- an interactive tutorial in UNIX version vim 

## under the normal mode:
i --- insert mode: insert a character before the character under the cursor
* a --- insert mode: insert a character after the character under the cursor
* o --- inserr mode: open a new line and put hte cursor to the begining of the new line
* x --- delete current letter
* enter --- move to the begining of the next line 
* h --- move left 
* j --- move down 
* k --- move up 
* l --- move right 
* u --- undo the last operation
* : --- enter command mode
* dd --- delete a line
* w --- move cursor forward one word
* b --- move cursor backward one word
* $ --- move to the end of the line(also e.g.3$)
* ^ --- move to the first nonblank character of the line
* fx --- searches the right line for the single character x
* Fx --- secrches the left line for the single character x 
* tx --- searches the right line for this single character x and stop one character before x (search till)
* Tx --- for the left side 
* f<esc> --- abord forward search
* G --- move the last line 
* 3G --- move to 3 line 
* 1G --- move to the top of the file
* % --- move to the first line 
* 50% --- move to the half part of the file
* 75% --- move to the 75% of the file
* CTRL+g --- show where you are in the file 
* CTRL+u --- scroll up half a screen of text
* CTRL+d --- scroll down hald a screen of the text
* v --- visual mode
* dw --- delete the rest of the current word where the cursor in 
* d$ --- delete the current line from where the cursor in 
* 3dw --- delete one word three times
* d3w --- delete three word one time 
* 3d2w --- delete two words three times
* c --- change text, it deletes the current choosed(in visul mode) otherwise the whole line  and leave you in the insert mode
* cw --- change the current word
* cc --- change the whole line (as dd)
* c$(or C) --- change from current place to the end of the line
* . --- repeat the last delete or change command 
* J --- join the current with the next line (also 3J)
* rx --- replace current character with x (also 3rx)
* 3rx --- replace threee character with x 
* 3r[Enter] --- replace 3 characters with one [Enter]
* ~ --- change a character's case, changes uppercase to lowercase and vice versa
* qx --- record keystrokes into the register named x   /  --- use q to quit the record  / --- use @x to reuse the operation / --- the difference from qx to . is that qx could record multiple steps

## under the command mode
*q! --- quit without save the changes
* :h(help) --- help document
* :h x --- get help on x commond 
* :set number --- show the line number
* :set nonumber --- turn off showing the number
* :help v_d --- get the help about what the delete(d) command does in visual mode 

##under the visual mode
* J --- join the selected lines
