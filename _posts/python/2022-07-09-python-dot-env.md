---
title:  "Python: How to use python-dotenv"
date:   2022-07-09 12:08:00 +0500
categories: python
tags:
  - python3
---

## Python-dotenv (Keep your secrets safe)
Do you know how to keep your secrets safe during development and production? In this article, I am going to guide you on how to work with _SECRETS_ and _KEYS_ without exposing them to the outside world, and keep them safe during development.

Python-dotenv is a Python module that allows you to specify environment variables in a traditional UNIX-like “.env” (dot-env) file within your Python project directory.

Environment variables are the set of key-value pairs for the current user environment. They are generally set by the operating system and the current user-specific configurations.

Python-dotenv helps us work with _SECRETS_ and _KEYS_ without exposing them to the outside world.

### Installation

```console
#Create a new virtual environment
python3 -m venv venv
#activate
source venv/bin/activate
#install
pip install python-dotenv
```

## Using the python-dotenv module

- First, you need to create a new .env file, and then add the name and value of the variables as key-value pairs.

```terminal
#.env file
ID = "12345689"
SECRET_KEY = "gsabijwjnciiwbjksa"
```

- Create app.py file in same location.

```terminal
└─[$] <> tree -a
.
├── app.py
├── .env
```

- Import and Call python-dotenv.

```terminal
## importing the load_dotenv from the python-dotenv module
from dotenv import load_dotenv

load_dotenv()
```

- Access the Environment Variables

```terminal
# importing the load_dotenv from the python-dotenv module
from dotenv import load_dotenv
# Provides ways to access the Operating System and allows us to read the environment variables
import os

load_dotenv()

my_id = os.getenv("ID")
my_secret_key = os.getenv("SECRET_KEY")

def myEnvironment():
    print(f'My id is: {my_id}.')
    print(f'My secret key is: {my_secret_key}.')

if __name__ == "__main__":
    myEnvironment()
```

- Output

```terminal
ID = "12345689"
SECRET_KEY = "gsabijwjnciiwbjksa"
```

A large number of security vulnerabilities can be resolved by taking care of leaked credentials, and the python-dotenv helps develop a safer project environment to work with, both during and after development.

In case you accidentally exposed your secret/key, do not panic because you can always generate a new key. Also, I would recommend generating new keys before deployment as a safety measure.

> Don't use _USERNAME_ in your environment as a key, it will conflict with your host system and load system username.
{: .prompt-warning }
