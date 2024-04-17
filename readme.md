[Reddit post](https://www.reddit.com/r/learnpython/comments/1c5g2rz/could_you_explain_how_to_correctly_import_modules/).

Hi everyone,

I really need some help because I'm probably doing everything wrong and it's driving me crazy.

I'm having trouble understanding how modules should be structured and how I should correctly import them.

This is my directory structure:

```text
planner
    ├── planner.pl
    ├── MILP 
    │   ├── __init__.py
    │   ├── __main__.py
    │   └── MILP.py
    ├── tests 
    │   ├── test.pl
    │   └── test.py
    └── utility 
        ├── Graph 
        │   ├── Graph.py
        │   └── __init__.py
        ├── utility.pl
        ├── utility.py
        └── __init__.py
```

And I want to use the class and functions from `Graph.py` inside `MILP.py`.

Let's say that I'm in the planner directory, I've tried to import it as

```python
from ..utility.Graph import Graph
```
and to execute `MILP.py` with `python3 -m MILP` but I get

```text
ImportError: attempted relative import beyond top-level package
```

So I tried to make the higher folder (planner) a module by adding a `__init__.py` and call

```bash
python3 -m planner.MILP
```

from outside planner, but I get

```text
ModuleNotFoundError: No module named 'MILP'
```

Is there a way to make the utility module (and what is inside it) always available? Like, if I compile with CMake and I want to include some header files that are somewhere else I can tell the compiler where to find those other header files. I've seen some questions that were saying to modify sys.path, but also many people not recommending doing it.

I'm really just confused why Python would not keep modules relative: if I want to import from a module that is in the directory before the one I'm calling from, why not just check that that directory exists and that it is a module and then let me use it?

**EDIT:**

Okay, so I was writing this yesterday evening, but then Reddit decided to delete all my edit and I hadn't had the time to write it all again. 

Basically I was doing two errors. The first is the main one and is something that was not clear to me at all, and that is the difference between script and module. Even though now that I think about it is obvious, I'm still not 100% sure I understand why some cases do not work as I intend them to do. 

Before getting to it, I "understood" my mistakes because I was following [u/Buttleston](https://www.reddit.com/user/Buttleston/) suggestion of creating a [repo](https://www.github.com/myoses/pythontest) to share my problem. Obviously before uploading it, I had to test if everything was not working as intended, but I found out that it actually was not not working as intended.

The repo structure is as follows:

```text
pythontest
    ├── main.py
    ├── mod1 
    │   ├── __main__.py
    │   └── Mod1.py
    └── mod2 
        ├── mod3 
        │   └── Mod3.py
        └── Mod2.py
```

Each `Mod*.py` has a class with the same name and my goal is to use the `Mod3` class inside of the `Mod1` class. The `main.py` _script_ in the main folder is to test that I can safely import everything and run everything smoothly.

So inside `Mod1.py` I have the instruction to import the `mod3` _module_:

```python
from mod2.mod3.Mod3 import Mod3
```

If we run `python3 main.py`, this works without any problem, but we already knew that this was going to work because we are calling the script from within the same directory and the modules are placed inside the same directory, so there is no reason that it should not work. 

Let's stick for a second to scripting to understand where all my problems came from. Let's add the following to `Mod1.py`:

```python
if __name__ == "__main__":
    Mod1()
```

So now, if we consider `Mod1.py` as a _script_, we should be able to execute it without problems, but lo and behold, this is not the case, because it is a _script_:

```bash
python3 mod1/Mod1.py
Traceback (most recent call last):
  File "/home/user/pythontest/mod1/Mod1.py", line 1, in <module>
    from mod2.mod3.Mod3 import Mod3
ModuleNotFoundError: No module named 'mod2'
```

I'm still not 100% sure why this should not work since I'm still calling the script from the main directory, but I reckon that when calling a script with something like `dir/script.py` the actual directory from which it is called is actually `dir`, and hence it won't see the modules that are outside of `dir`. 

Indeed, if we move inside the `mod1` directory and call the `Mod1.py` script, we do not expect it to work and it returns the same exact error: 

```bash
python3 Mod1.py
Traceback (most recent call last):
  File "/home/user/pythontest/mod1/Mod1.py", line 1, in <module>
    from mod2.mod3.Mod3 import Mod3
ModuleNotFoundError: No module named 'mod2'
```

Okay, so let's consider `Mod1` as a module! Let's add a `__main__.py` file in which we put the same invocation as before: 

```python
if __name__ == "__main__":
    Mod1()
```

and let's go back to the main directory (`pythontest`). Now we can run the module by executing:

```bash
python3 -m mod1
```

And it works perfectly, because we are indeed considering the _module_ as a _module_.

Final thing that is not clear to me. I would expect that by exiting `pythontest` I should be able to call the `mod1` module by executing:

```bash
python3 -m pythontest.mod1
```

But this is not working and returns the error:

```text
ModuleNotFoundError: No module named 'mod2'Traceback (most recent call last):
  File "/usr/lib/python3.10/runpy.py", line 196, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "/usr/lib/python3.10/runpy.py", line 86, in _run_code
    exec(code, run_globals)
  File "/home/user/pythontest/mod1/__main__.py", line 1, in <module>
    from .Mod1 import Mod1
  File "/home/user/pythontest/mod1/Mod1.py", line 1, in <module>
    from mod2.mod3.Mod3 import Mod3
ModuleNotFoundError: No module named 'mod2'
```

Obviously I had to check whether by changing the import to 

```python
from pythontest.mod1.mod2.Mod3 import Mod3
```

it would have worked, and obviously it did! So I wondered, what if I put it relative? But now it becomes even more strange! If I write: 

```python
from .mod2.mod3.Mod3 import Mod3
```

So we say that from `mod1` the interpreter should look in the same directory for `mod2`, but we obtain the following error:

```
ModuleNotFoundError: No module named 'mod2'Traceback (most recent call last):
  File "/usr/lib/python3.10/runpy.py", line 196, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "/usr/lib/python3.10/runpy.py", line 86, in _run_code
    exec(code, run_globals)
  File "/home/user/pythontest/mod1/__main__.py", line 1, in <module>
    from .Mod1 import Mod1
  File "/home/user/pythontest/mod1/Mod1.py", line 1, in <module>
    from .mod2.mod3.Mod3 import Mod3
ModuleNotFoundError: No module named 'pythontest.mod1.mod2'
```

And then it dawned on me: why is it `No module named 'pythontest.mod1.mod2'`? Why `pythontest.mod1.mod2` and not `pythontest.mod2`? Well because we have said before that the modules are looked from the script (or module) that tries to find them, so in this case when we are in `pythontest.mod1.Mod1` and we say `from .mod2 ...` we are saying to look inside `mod1` for `mod2`. 

And indeed, by changing the import to 

```python
from ..mod2.mod3.Mod3 import Mod3
```

we fix the problem. YOU MAY think that this is the end, but unfortunately it is not. 
If we enter again `pythontest` and run `python3 -m mod1` I should well expect that this works, and why should it not? Obviously it does not: 

```
Traceback (most recent call last):
  File "/usr/lib/python3.10/runpy.py", line 196, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "/usr/lib/python3.10/runpy.py", line 86, in _run_code
    exec(code, run_globals)
  File "/home/enrico/pythontest/mod1/__main__.py", line 1, in <module>
    from .Mod1 import Mod1
  File "/home/enrico/pythontest/mod1/Mod1.py", line 1, in <module>
    from ..mod2.mod3.Mod3 import Mod3
ImportError: attempted relative import beyond top-level package
```

So the point is that to correctly import a module it matters from where you are executing the script/module. In my opinion this still makes no sense and there must be a better way to import modules that I do not know about, so if any of you can point me to the right direction it would be great.
