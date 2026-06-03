# Remove files 

> syntax: rm <filename>

**Parameters**
1. -i : used for interactive mode while deleting 
        eg: rm -i file-*.txt (prompt do you want to delete or not)


# Clear Terminal 
> Just Found Out **Ctrl + L** will clear the terminal 


# append && write operator (> && >>)

> note: if you use ">" this will write the file removing everything but ">>" this will append the text.


# Grep 
> used to find the matching files or text 

> syntax: grep <textmatch> <parameter> <filepath>

**Parameters**
1. -A1: give me text after one line of matching text.
2. -B1: give me text before one line of matching text.
3. -C1: give me text before and after line of matching text.
4. -i: doesnt care casesensitive.
4. -o: show only matched text.


# Pager
> Recently found out that **less** and **more** are pager.

### what is pager?
pager is a program that lets to view large data in one window without scrollbar.


# Manuals page
> We can see manuals of the command using ** man ** command.

> syntax: man ls

> note: man also have attributes like man 1 ls it will show first page of manuals.

> Quick fact: man only works with external commands, If we want to see manual page of internal commands like **history** we need to use **help** command like **help history** then it will show the manual page for the history.


# type command ===(super useful)=== 
> used to know if the command is built in or external.

> syntax: type ls 

**parameter** 
1. -a (important) : shows every possible availabe types for the given command.
