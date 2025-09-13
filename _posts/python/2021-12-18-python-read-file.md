---
title:  "Python: Reading and Writing Files"
date:   2021-12-18 15:02:15 +0500
categories: python
tags:
  - python3
---

## Reading and Writing Files
Python has several functions for creating, reading, updating, and deleting files.

### Syntax

To open a file for reading it is enough to specify the name of the file:

`f = open("file.txt")`

The code above is the same as:

`f = open("file.txt", "rt")`

Because "r" for read, and "t" for text are the default values, you do not need to specify them.

### Mode can be

- "r" - Read - Default value. Opens a file for reading, error if the file does not exist
- "a" - Append - Opens a file for appending, creates the file if it does not exist
- "w" - Write - Opens a file for writing, creates the file if it does not exist
- "x" - Create - Creates the specified file, returns an error if the file exists

In addition you can specify if the file should be handled as binary or text mode

- "t" - Text - Default value. Text mode
- "b" - Binary - Binary mode (e.g. images)

It is good practice to use the _with_ keyword when dealing with file objects. The advantage is that the file is properly closed after its suite finishes.

```py
>>> with open('file.txt') as f:
...     read_data = f.read()
```

If you’re not using the _with_ keyword, then you should call _f.close()_ to close the file and immediately free up any system resources used by it.

### Methods of File Objects

To read a file’s contents, call _f.read()_.

```py
>>> f = open('device')
>>> print(f.read())
192.168.10.10
192.168.10.11
>>> f.close()
```

_f.readline()_ reads a single line from the file.

```py
>>> f = open('device')
>>> print(f.readline())
192.168.10.10
>>> print(f.readline())
192.168.10.11
>>> f.readline()
# empty line
>>> f.close()
```

For reading lines from a file, you can loop over the file object. This is memory efficient, fast, and leads to simple code:

```py
>>> f = open('device')
>>> for ip in f:
...     print(ip)
...
192.168.10.10

192.168.10.11
```

If you want to read all the lines of a file in a list you can also use _f.readlines()_.

```py
>>> f = open('device')
>>> print(f.readlines())
['192.168.10.10\n', '192.168.10.11\n', '192.168.10.12\n'] # return a list
```

### Write to an Existing File

To write to an existing file, you must add a parameter to the open() function:

- "a" - Append - will append to the end of the file
- "w" - Write - will overwrite any existing content

`f.write(string)` writes the contents of string to the file, returning the number of characters written.

```py
>>> f = open('device', 'a') # file will open in append mode
>>> f.write('192.168.10.12') # this line will add as last line in file
>>> f.close()

>>> f = open('device')
>>> for ip in f:
...     print(ip)
...
192.168.10.10

192.168.10.11

192.168.10.12
```

### Create a New File

To create a new file in Python, use the open() method, with one of the following parameters:

- "x" - Create - will create a file, returns an error if the file exist
- "a" - Append - will create a file if the specified file does not exist
- "w" - Write - will create a file if the specified file does not exist

#### Example

Create a file called "file.txt":

`f = open("file.txt", "x")`

> Result: a new empty file is created!

Create a new file if it does not exist:

`f = open("file.txt", "w")`

[More](https://docs.python.org/3.8/tutorial/inputoutput.html#reading-and-writing-files)
