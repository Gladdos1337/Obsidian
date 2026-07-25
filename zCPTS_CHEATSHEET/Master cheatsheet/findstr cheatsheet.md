
To search for _hello_ or _there_ in file _x.y_, type:

```
findstr hello there x.y
```

To search for _hello there_ in file _x.y_, type:

```
findstr /c:"hello there" x.y
```

To find all occurrences of the word _Windows_ (with an initial capital letter W) in the file _proposal.txt_, type:

```
findstr Windows proposal.txt
```

To search every file in the current directory and all subdirectories that contained the word _Windows_, regardless of the letter case, type:

```
findstr /s /i Windows *.*
```

To find all occurrences of lines that begin with _FOR_ and are preceded by zero or more spaces (as in a computer program loop), and to display the line number where each occurrence is found, type:

```
findstr /b /n /r /c:^ *FOR *.bas
```

To list the exact files that you want to search in a text file, use the search criteria in the file _stringlist.txt_, to search the files listed in _filelist.txt_, and then to store the results in the file _results.out_, type:

```
findstr /g:stringlist.txt /f:filelist.txt > results.out
```

To list every file containing the word _computer_ within the current directory and all subdirectories, regardless of case, type:

```
findstr /s /i /m \<computer\> *.*
```

To list every file containing the word computer and any other words that begin with comp, (such as compliment and compete), type:

```
findstr /s /i /m \<comp.* *.*
```