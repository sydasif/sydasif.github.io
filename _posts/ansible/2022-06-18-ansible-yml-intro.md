---
title:  "Part 07: Ansible and YAML"
date:   2022-06-18 09:55:00 +0500
categories: ansible
tags:
  - automation
  - yaml
---

## YAML Introduction
[YAML](https://yaml.org/) is the abbreviated form of *“YAML Ain’t markup language”* is a data serialization language which is designed to be human-friendly and works well with other programming languages for everyday tasks. YAML is a less complex than XML or JSON and it allows you to provide configuration settings.

### YAML Basic Rules

- YAML files should end in .yaml or .yml.
- YAML is case sensitive.
- YAML does not allow the use of tabs, 2 spaces are used instead of tabs.
- Hash `#` is used as comments.
- YAML files are supposed to start with three dashes `---`.
- In general, YAML strings don’t have to be quoted, although you can quote them if you prefer.
- In some scenarios in Ansible, you will need to quote strings. These typically involve the use of `double curly braces` for variable substitution.
- YAML has a native Boolean type and provides you with a wide variety of strings that can be interpreted as true or false, and not case sensitive, `true` or `True` and `false` or `False`.

### YAML Basic Data Types

#### key-value Pair

A single value and a variable, it can be string, integer, float or bool.

```yml
max_retry: 100            # integer
database: user            #string
pass: true                # bool
date: 1990-02-06 14:33:22 # ISO 8601 standard for time and date
```

#### Mapping

YAML dictionaries are like objects in JSON, dictionaries in Python, or hashes in Ruby. Technically, these are called mappings in YAML, but I call them dictionaries here to be consistent with the official Ansible documentation. They look like this:

```yml
animals:
  name: cat
    color: white
  name: dog
    color: black
```

#### Sequence

YAML lists are like arrays in JSON and Ruby or lists in Python. Technically, these are called sequences in YAML, but I call them lists here to be consistent with the official Ansible documentation. They are delimited with hyphens, like this

```yml
# list
employee:
  - name: Jack
  - name: Syd

employee: ["Jack", "Syd"]    # list

# complex list (mapping and sequence)
employee:
  - name: Jack
    age: 28
  - name: Syd
    age: 34

# complex list (mapping and sequence)
employee:
  - {name: "Jack", age: "22"}
  - {name: "Syd", age: "42"}
```

### Line Folding and Break

When writing playbooks, you’ll often encounter situations where you’re passing many arguments to a module. You might want to break this up across multiple lines in your file, but you want Ansible to treat the string as if it were a single line, For example:

```yml
employee:
  - name: Jack
    age: 28
description: >      # This is a single line
  This is a 1st line
  this is a not 2nd line and part of 1st line
  this is a also part of 1st line.
```

You can also do line breaks with YAML by using the `( | )` character. The YAML parser will place line breaks.

```yml
employee:
  - name: Syd
    age: 42
signature: | # line breaks
  Mike
  Academy
  E-mail - abc@abc
```

[See more on YAML](https://www.tutorialspoint.com/yaml/index.htm)
