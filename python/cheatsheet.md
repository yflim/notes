
# Table of Contents

1.  [Miscellaneous](#org78ddf4b)
2.  [Packages & Modules](#org0184b96)
3.  [System utils](#org5b64aea)
4.  [Types](#org8c03b05)
5.  [Data](#org02b04b9)
    1.  [Pandas](#orgc1c806c)
    2.  [virtual env, etc.](#orgb3f2532)
6.  [Visualisation](#orgc288804)
    1.  [Seaborn](#orge8e678e)
7.  [Jupyter](#org5fa879b)
    1.  [Help](#orgc2e6953)
    2.  [Editing](#orga1b515d)
    3.  [Etc.](#org42da28f)
8.  [Integrate into the above](#orgd706160)
9.  [Help](#orgf5870f9)
10. [Basics](#orgdd53c5e)
11. [Gotchas](#orge1ec3e5)
12. [&ldquo;Functional programming&rdquo;](#org401306b)
    1.  [Keyword arguments & Arbitrary argument lists](#orgb3c9ed9)
    2.  [Unpacking argument lists](#orgb3f2ab7)
13. [Data types](#org7cfd5ba)
    1.  [Collections](#orgfd3daef)
    2.  [Type hints](#orgc4325cf)
14. [Modules, packages, oh my!](#orgd10a748)
    1.  [Modules](#org96ae9ac)
    2.  [Packages](#orga078e34)
15. [System environment](#orgb46374a)
16. [**PSA**](#org2ce012c)
17. [Jupyter](#org4445f81)
18. [NumPy](#org6a35b0c)
19. [Pandas](#org79671f4)
20. [Plotting](#org09034cd)
21. [Mod Cons](#org061542a)
22. [Typing](#orgee8a954)
23. [Performance](#org7ff3a49)
24. [Python environment management](#orgf90d05f)
    1.  [Update & install](#org8738b12)
        1.  [On Ubuntu](#orgfea80b7)
        2.  [Mac OS](#orgd28f6dc)
    2.  [General](#orgd8a8dc7)
        1.  [Tool to consider when it gets better](#orgda29aa7)
    3.  [In code](#org27300cf)
    4.  [Project-specific](#orgdd0d291)



<a id="org78ddf4b"></a>

# Miscellaneous

-   Help: `help(module)`, `help(fn)`
-   Reload module without restarting interpreter:
    
        import importlib
        importlib.reload(mod)


<a id="org0184b96"></a>

# Packages & Modules

-   Import examples:
    `import foo.bar`
    `from foo import bar`
    `from foo import *`
-   If <span class="underline"><span class="underline">init.py</span></span> is present in a package directory, it is invoked when the package or a module in the package is imported.
-   Subpackages e.g.:
    `import pkg.sub_pkg1.mod1`
    `from pkg.sub_pkg2 import mod3`


<a id="org5b64aea"></a>

# System utils

    import os
    
    # Run Unix shell commands
    os.system('ls')
    
    # List files in directory
    for f in os.listdir('data'):
        if isfile(f'data/{f}'):
            print(f)


<a id="org8c03b05"></a>

# Types

-   Check type: `isinstance(x, str)`


<a id="org02b04b9"></a>

# Data


<a id="orgc1c806c"></a>

## Pandas

-   Sorting: `sort_index`, `sort_values`

-   Trim df or series values at lower or upper threshold (like min() or max()):

`df.clip(lower=None, upper=None, axis=None, inplace=False, *args, **kwargs)`

-   Cumulative max/min over df or series axis:

`df.cummax(axis=None, skipna=True, *args, **kwargs)`

-   Drop specified labels from rows or columns (rows in example):

`test_dataset = dataset.drop(train_dataset.index)`

-   Drop column from dataframe:

`train_labels = train_features.pop('MPG')`

-   Drop NaNs from Series or Dataframe:

`df.dropna(axis=0, how='any', thresh=None, subset=None, inplace=False)`

-   Return whether any element is True in Series or Dataframe:

`df.any(axis=0, bool_only=None, skipna=True, level=None, **kwargs)`

-   Sample randomly from dataframe:

`train_dataset = dataset.sample(frac=0.8, random_state=0)`

-   Negation: `\~`, e.g. `\~a.isin(b)`

-   Multi-indexes:
    -   Returns Series with index levels route<sub>id</sub>, trip<sub>id</sub>
        `route_trip_stop_counts = stop_times.groupby('route_id').trip_id.value_counts()`
    -   Get index level values:
        `route_trip_stop_counts.index.get_level_values(0)`
    -   Drop index level 1 i.e. trip<sub>id</sub>, then get get variance for each route<sub>id</sub>:
        `route_trip_stop_counts.droplevel(1).groupby('route_id').var()`
    -   Reset index:
        -   Get df with columns route<sub>id</sub>, trip<sub>id</sub> in place of the original index:
            `route_trip_stop_counts.rename('stop_count').reset_index()`
        -   Get df with route<sub>id</sub> as index and additional column trip<sub>id</sub>:
            `route_trip_stop_counts.rename('stop_count').reset_index(level='trip_id')`


<a id="orgb3f2532"></a>

## TODO virtual env, etc.

-   Groupby
    
    -   `df.groupby('A')` is just syntactic sugar for `df.groupby(df['A'])`
    
        import pandas as pd
        
        df = pd.DataFrame(
          [
              ("bird", "Falconiformes", 389.0),
              ("bird", "Psittaciformes", 24.0),
              ("mammal", "Carnivora", 80.2),
              ("mammal", "Primates", np.nan),
              ("mammal", "Carnivora", 58),
          ],
          index=["falcon", "parrot", "lion", "monkey", "leopard"],
          columns=("class", "order", "max_speed"))
        return df
        # default is axis=0
        grouped = df.groupby("class")
        grouped = df.groupby("order", axis="columns")
        grouped = df.groupby(["class", "order"])

-   [Windowing API](https://pandas.pydata.org/docs/user_guide/window.html)
-   [Resampling API](https://pandas.pydata.org/docs/user_guide/timeseries.html#resampling)


<a id="orgc288804"></a>

# Visualisation


<a id="orge8e678e"></a>

## Seaborn

Draw a plot of two variables with bivariate and univariate graphs (e.g. scatterplot with histograms):
`sns.jointplot(data=df, x='x', y='y')`


<a id="org5fa879b"></a>

# Jupyter


<a id="orgc2e6953"></a>

## Help

-   Function help: inside parentheses, hit `Shift+Tab`


<a id="orga1b515d"></a>

## Editing

-   Change cell type: to markdown `m`, to code `y`
-   Clear cell output: `Ctrl+o`
-   Indent `Cmd+]`
-   Dedent `Cmd+[`


<a id="org42da28f"></a>

## Etc.

-   Command palette (listing commands with shortcuts): `Cmd+Shift+p`


<a id="orgd706160"></a>

# TODO Integrate into the above


<a id="orgf5870f9"></a>

# Help

-   View documentation: `help(fn)`
-   Display the printable representation of an object: `repr()`


<a id="orgdd53c5e"></a>

# Basics

    import builtins
    something = (0,0)
    match type(something):
          case builtins.list:
              print("list")
          case builtins.tuple:
              print("tuple")


<a id="orge1ec3e5"></a>

# Gotchas

-   Total bullshit: `list()` apparently mutates its input when given a map.
    -   When called again on the same variable without doing anything else to the latter in between, returns an empty result
-   WTF???!!!???
    
        None == None
        # True
        None != None
        # False
        ## BUT!!!
        pd.Series([None] * 10) != pd.Series([None] * 10)
        # 0    True
        # 1    True
        # 2    True
        dtype: bool
        pd.Series([None] * 10) == pd.Series([None] * 10)
        # 0    False
        # 1    False
        # 2    False


<a id="org401306b"></a>

# &ldquo;Functional programming&rdquo;

-   Call method on object or module: `(getattr(obj, attr_name_str))`
    -   E.g.:
        
            (getattr('', 'join'))(['a', 'b', 'c'])
            #-> 'abc'
            (getattr(chile1_df[1], 'le')(pd.Timestamp('2022-03-01')) & (chile1_df[0] == 'SOSP')).sum()
            #-> ...


<a id="orgb3c9ed9"></a>

## Keyword arguments & Arbitrary argument lists

-   E.g. `getattr(df[col], op)(pd.Timestamp(*ts_args, **ts_kwargs))`


<a id="orgb3f2ab7"></a>

## Unpacking argument lists

-   I.e. sort of the reverse of what happens in the preceding section/example, e.g.
    
        args = [3, 6]
        list(range(*args))


<a id="org7cfd5ba"></a>

# Data types


<a id="orgfd3daef"></a>

## Collections

e.g. `from collections.abc import Collection`


<a id="orgc4325cf"></a>

## Type hints

-   Multiple types:
    
        def fn(arg: int | str = ''):
            return arg
-   `Pandera` for Pandas (or more general?) type hinting


<a id="orgd10a748"></a>

# Modules, packages, oh my!


<a id="org96ae9ac"></a>

## Modules

-   A module is a file containing Python definitions and statements.
    -   `import module` does not add definitions from `module` directly to current namespace
    -   `from module import ...` adds definitions directly
    -   `as` can also be used with `from ...`, e.g. `from module import fn as function`


<a id="orga078e34"></a>

## Packages

-   A package is a (possibly nested) collection of modules
-   `__init__.py` is required to make Python treat the directory as a package
    -   Can be empty, execute init code for the package, or define `__all__`
        -   `__all__` lists module names that should be imported by (the discouraged) `from package import *`, e.g.
            `__all__ = ["echo", "surround", "reverse"]`
        -   if `__all__` is not defined, `from package import *` runs any init code in, and imports names defined and submodules explicitly loaded by, `__init__.py`


<a id="orgb46374a"></a>

# System environment

-   get pwd: `os.getcwd()`
-   If a .env file is present in project, `pipenv shell` and `pipenv run` will automatically load it


<a id="org2ce012c"></a>

# **PSA**

-   `python<v> -m <command>` **whenever applicable and in doubt!!!**


<a id="org4445f81"></a>

# Jupyter

-   Run on WSL: `python<-v> -m jupyter notebook --no-browser`
-   `jupyter kernelspec list`
-   ipynb <-> py conversion:
    `jupytext --to notebook notebook.py`
    `jupytext --to py notebook.ipynb`
-   Add a kernel manually else Jupyter will clobber any existing kernel for any pre-existing venv with the same name (i.e. sane practice): `python3.11 -m ipykernel install --user --name <non-confusing-name-for-jupyter>`
-   Uninstall kernel (note asymmetry with install command&#x2026;): `python3.11 -m ipykernel uninstall <name>`


<a id="org6a35b0c"></a>

# NumPy

-   Length of each string in an array: `np.char.str_len(['python', 'is', 'snake', 'oil'])`
-   Element-wise truth value of `x1` and `x2`: `np.logical_and(x1, x2)`
-   Reduce: e.g. `np.logical_and.reduce(df)` reduces each column of df


<a id="org79671f4"></a>

# Pandas

-   Merge two dataframes with overlapping index, keeping column values from left DataFrame
    -   Option 1 (assuming date is already set as index): `df1.combine_first(df2)`
    -   Option 2: `pd.concat([df1, df2]).drop_duplicates(["date"], keep="first", ignore_index=True)`
-   When in doubt, always `.values` instead of losing more perfectly good hours of life to bullshit state-
    (index-) instead of value-based operations
-   Modify series in place: `s.update([4,5,6])`
-   Define order for series, e.g. `df['m'] = pd.Categorical(df['m'], ["March", "April", "Dec"])`
-   Count distinct values:
    
        port_r_df.port_name.nunique()
        # or
        port_r_df.groupby(['port_id']).port_name.nunique()
-   True if all elements within a series or along a dataframe axis are non-zero, not-empty or not-False, e.g.:
    -   Column-wise values all return true: `df.all()`
    -   Row-wise values all return true: `df.all(axis=1)`
-   `any()`: analogous to `all()`


<a id="org09034cd"></a>

# Plotting

-   Seaborn boxplot: `showfliers=False` to omit outliers


<a id="org061542a"></a>

# Mod Cons

-   Reload module (etc.):
    
        from importlib import reload
        reload(module_name_or_alias)
-   Automatic docstring generation in `sphinx-doc` Emacs minor mode: `C-c M-d`


<a id="orgee8a954"></a>

# Typing

-   Single dispatch for overloading with single signature


<a id="org7ff3a49"></a>

# Performance

-   `timeit`: measure execution time of arbitrary statement
    
    -   in REPL session, need to set `globals` to current set of global vars with `globals()`
    -   can limit number of executions with `number`
    
        import timeit
        timeit.timeit('fibonacci(35)', globals=globals(), number=1)
-   `functools.cache` caches in memory the result of a function for a (each?) particular set of arguments


<a id="orgf90d05f"></a>

# Python environment management


<a id="org8738b12"></a>

## Update & install


<a id="orgfea80b7"></a>

### On Ubuntu

Updating Python:

    sudo apt install software-properties-common
    sudo add-apt-repository ppa:deadsnakes/ppa
    # Adding a repository will usually trigger an update. If not, run manually:
    sudo apt update
    apt list | grep python3.11
    sudo apt install python3.11
    # Check:
    python3.11 -V
    # Create symbolic links to python 3 binaries, e.g.
    sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 1
    sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1
    # Choose default:
    sudo update-alternatives --config python3


<a id="orgd28f6dc"></a>

### Mac OS

`brew install python@3.11`


<a id="orgd8a8dc7"></a>

## General

 <Global Python / package information: `python -m site`
Show user site packages directory: `python -m site --user-site`
List user site packages (with versions): `pip list --user`
Debian-style package info: `pip show <pkg>`
User package installation (which is default, but just to be safe): `pip install --user <pkg>`
Upgrade: `pip install <pkg> -U`
Uninstall along with sub-dependencies: `pip-autoremove pkg`


<a id="orgda29aa7"></a>

### Tool to consider when it gets better

[pigar](<https://github.com/damnever/pigar>): generates requirements.txt based on imports


<a id="org27300cf"></a>

## In code

Package location: `<pkg>.__path__`
Module location: `<module>.__file__`


<a id="orgdd0d291"></a>

## Project-specific

Create virtual env: `python[3] -m venv --system-site-packages [--clear] [--upgrade-deps] .venv` (clear to overwrite, upgrade-deps to upgrade `pip` and `setuptools`?) or `virtualenv .venv`
Activate virtual env: `source .venv/<bindir>/activate` or `[py -m] pipenv shell`
Install packages from requirements file: `py -m pip install -r requirements.txt`
Show available package versions: `py -m pip install <pkg>==`
Force reinstall in case of breaking changes, e.g. with `sqlsnakeoil`: `pip install --force-reinstall -v "SQLAlchemy==1.4.46"`
Write (only local) requirements to requirements.txt: `python -m pip freeze -l`
Deactivate virtual env: `exit` if `pipenv shell` used for activation. And on Windows, sorry&#x2026;?
Upgrade package: `pip install <pkg> -U`
Controlled burn workflow:

1.  Install top-level requirements and keep manual list in e.g. `requirements-toplevel.txt`

`pip freeze > requirements.txt`

1.  Whenever an included package is no longer included:

    pip freeze > uninstall.txt
    pip uninstall -r uninstall.txt
    pip install -r requirements-toplevel.txt
    pip freeze > requirements.txt

