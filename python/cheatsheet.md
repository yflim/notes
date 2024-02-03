
# Table of Contents

1.  [Miscellaneous](#org462872e)
2.  [Packages & Modules](#org9acb775)
    1.  [Modules](#org917a673)
    2.  [Packages](#orgf03ef52)
3.  [System utils](#orge7b61de)
4.  [Types](#org83c6eaa)
5.  [Data](#orgcee7d51)
    1.  [Pandas](#orgdabc849)
6.  [Visualisation](#org7f0ee46)
    1.  [Seaborn](#org321d0c4)
7.  [Jupyter](#org43ab9d3)
    1.  [Help](#org6106fe5)
    2.  [Editing](#org6bfe4a1)
    3.  [Etc.](#org2b75782)
8.  [&ldquo;Functional programming&rdquo;](#org9572720)
    1.  [Keyword arguments & Arbitrary argument lists](#org946810d)
    2.  [Unpacking argument lists](#org369bb4d)
9.  [Data types](#org346f847)
    1.  [Builtins](#orgbf0b5f4)
    2.  [Collections](#org60e1a92)
    3.  [Type hints](#orgf34d1e4)
10. [Gotchas](#org814afde)
11. [Help](#org437aeff)
12. [Integrate into the above](#org63d776c)
13. [**PSA**](#org5d39960)
14. [Jupyter](#org116a413)
15. [NumPy](#org3e61c55)
16. [Pandas](#org15e25ab)
17. [Plotting](#org191e1d3)
18. [Mod Cons](#org8e730c6)
19. [Typing](#orga59284f)
20. [Performance](#org9f421a4)
21. [Python environment management](#orgbdae5fc)
    1.  [Update & install](#orgc4895c8)
        1.  [On Ubuntu](#orgf22cc2e)
        2.  [Mac OS](#orgb3b85ce)
    2.  [General](#org495889a)
        1.  [Tool to consider when it gets better](#org96d9878)
    3.  [In code](#orga9d0d0a)
    4.  [Project-specific](#orgd737021)



<a id="org462872e"></a>

# Miscellaneous

-   Reload module without restarting interpreter:
    
        import importlib
        importlib.reload(mod)


<a id="org9acb775"></a>

# Packages & Modules


<a id="org917a673"></a>

## Modules

-   A module is a file containing Python definitions and statements.
    -   `import module` does not add definitions from `module` directly to current namespace
    -   `from module import ...` adds definitions directly
    -   `as` can also be used with `from ...`, e.g. `from module import fn as function`
    -   More import examples:
        `import foo.bar`
        `from foo import bar`
        `from foo import *`


<a id="orgf03ef52"></a>

## Packages

-   A package is a (possibly nested) collection of modules
-   `__init__.py` is required to make Python treat the directory as a package
    -   Invoked when the package or a module in the package is imported
    -   Can be empty, execute init code for the package, or define `__all__`
        -   `__all__` lists module names that should be imported by (the discouraged) `from package import *`, e.g.
            `__all__ = ["echo", "surround", "reverse"]`
        -   if `__all__` is not defined, `from package import *` runs any init code in, and imports names defined and submodules explicitly loaded by, `__init__.py`
-   Importing modules from subpackages e.g.:
    `import pkg.sub_pkg1.mod1`
    `from pkg.sub_pkg2 import mod3`


<a id="orge7b61de"></a>

# System utils

-   get pwd: `os.getcwd()`

    import os
    
    # Run Unix shell commands
    os.system('ls')
    
    # List files in directory
    for f in os.listdir('data'):
        if isfile(f'data/{f}'):
            print(f)


<a id="org83c6eaa"></a>

# Types

-   Check type: `isinstance(x, str)`


<a id="orgcee7d51"></a>

# Data


<a id="orgdabc849"></a>

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


<a id="org7f0ee46"></a>

# Visualisation


<a id="org321d0c4"></a>

## Seaborn

Draw a plot of two variables with bivariate and univariate graphs (e.g. scatterplot with histograms):
`sns.jointplot(data=df, x='x', y='y')`


<a id="org43ab9d3"></a>

# Jupyter


<a id="org6106fe5"></a>

## Help

-   Function help: inside parentheses, hit `Shift+Tab`


<a id="org6bfe4a1"></a>

## Editing

-   Change cell type: to markdown `m`, to code `y`
-   Clear cell output: `Ctrl+o`
-   Indent `Cmd+]`
-   Dedent `Cmd+[`


<a id="org2b75782"></a>

## Etc.

-   Command palette (listing commands with shortcuts): `Cmd+Shift+p`


<a id="org9572720"></a>

# &ldquo;Functional programming&rdquo;

-   Call method on object or module: `(getattr(obj, attr_name_str))`
    -   E.g.:
        
            (getattr('', 'join'))(['a', 'b', 'c'])
            #-> 'abc'
            (getattr(chile1_df[1], 'le')(pd.Timestamp('2022-03-01')) & (chile1_df[0] == 'SOSP')).sum()
            #-> ...


<a id="org946810d"></a>

## Keyword arguments & Arbitrary argument lists

-   E.g. `getattr(df[col], op)(pd.Timestamp(*ts_args, **ts_kwargs))`


<a id="org369bb4d"></a>

## Unpacking argument lists

-   I.e. sort of the reverse of what happens in the preceding section/example, e.g.
    
        args = [3, 6]
        list(range(*args))


<a id="org346f847"></a>

# Data types


<a id="orgbf0b5f4"></a>

## Builtins

    import builtins
    something = (0,0)
    match type(something):
          case builtins.list:
              print("list")
          case builtins.tuple:
              print("tuple")


<a id="org60e1a92"></a>

## Collections

e.g. `from collections.abc import Collection`


<a id="orgf34d1e4"></a>

## Type hints

-   Multiple types:
    
        def fn(arg: int | str = ''):
            return arg
-   `Pandera` for Pandas (or more general?) type hinting


<a id="org814afde"></a>

# Gotchas

-   `list()` apparently mutates its input when given a map.
    -   When called again on the same variable without doing anything else to the latter in between, returns an empty result

-   ???!!!???
    
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


<a id="org437aeff"></a>

# Help

-   Help: `help(module)`, `help(fn)`
-   Display the printable representation of an object: `repr()`


<a id="org63d776c"></a>

# TODO Integrate into the above


<a id="org5d39960"></a>

# **PSA**

-   `python<v> -m <command>` **whenever applicable and in doubt!!!**


<a id="org116a413"></a>

# Jupyter

-   Run on WSL: `python<-v> -m jupyter notebook --no-browser`
-   `jupyter kernelspec list`
-   ipynb <-> py conversion:
    `jupytext --to notebook notebook.py`
    `jupytext --to py notebook.ipynb`
-   Add a kernel manually else Jupyter will clobber any existing kernel for any pre-existing venv with the same name (i.e. sane practice): `python3.11 -m ipykernel install --user --name <non-confusing-name-for-jupyter>`
-   Uninstall kernel (note asymmetry with install command&#x2026;): `python3.11 -m ipykernel uninstall <name>`


<a id="org3e61c55"></a>

# NumPy

-   Length of each string in an array: `np.char.str_len(['python', 'is', 'snake', 'oil'])`
-   Element-wise truth value of `x1` and `x2`: `np.logical_and(x1, x2)`
-   Reduce: e.g. `np.logical_and.reduce(df)` reduces each column of df


<a id="org15e25ab"></a>

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


<a id="org191e1d3"></a>

# Plotting

-   Seaborn boxplot: `showfliers=False` to omit outliers


<a id="org8e730c6"></a>

# Mod Cons

-   Reload module (etc.):
    
        from importlib import reload
        reload(module_name_or_alias)
-   Automatic docstring generation in `sphinx-doc` Emacs minor mode: `C-c M-d`


<a id="orga59284f"></a>

# Typing

-   Single dispatch for overloading with single signature


<a id="org9f421a4"></a>

# Performance

-   `timeit`: measure execution time of arbitrary statement
    
    -   in REPL session, need to set `globals` to current set of global vars with `globals()`
    -   can limit number of executions with `number`
    
        import timeit
        timeit.timeit('fibonacci(35)', globals=globals(), number=1)
-   `functools.cache` caches in memory the result of a function for a (each?) particular set of arguments


<a id="orgbdae5fc"></a>

# Python environment management


<a id="orgc4895c8"></a>

## Update & install


<a id="orgf22cc2e"></a>

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


<a id="orgb3b85ce"></a>

### Mac OS

`brew install python@3.11`


<a id="org495889a"></a>

## General

<Global Python / package information: `python -m site`
Show user site packages directory: `python -m site --user-site`
List user site packages (with versions): `pip list --user`
Debian-style package info: `pip show <pkg>`
User package installation (which is default, but just to be safe): `pip install --user <pkg>`
Upgrade: `pip install <pkg> -U`
Uninstall along with sub-dependencies: `pip-autoremove pkg`


<a id="org96d9878"></a>

### Tool to consider when it gets better

[pigar](<https://github.com/damnever/pigar>): generates requirements.txt based on imports


<a id="orga9d0d0a"></a>

## In code

Package location: `<pkg>.__path__`
Module location: `<module>.__file__`


<a id="orgd737021"></a>

## Project-specific

-   If a .env file is present in project, `pipenv shell` and `pipenv run` will automatically load it
    -   Experience suggests steering clear of `pipenv`, `virtualenv`, and other snake oils though
-   Create virtual env: `python[3] -m venv --system-site-packages [--clear] [--upgrade-deps] .venv` (clear to overwrite, upgrade-deps to upgrade `pip` and `setuptools`?) or `virtualenv .venv`
-   Activate virtual env: `source .venv/<bindir>/activate` or `[py -m] pipenv shell`
-   Install packages from requirements file: `py -m pip install -r requirements.txt`
-   Show available package versions: `py -m pip install <pkg>==`
-   Force reinstall in case of breaking changes, e.g. with `sqlsnakeoil`: `pip install --force-reinstall -v "SQLAlchemy==1.4.46"`
-   Write (only local) requirements to requirements.txt: `python -m pip freeze -l`
-   Deactivate virtual env: `exit` if `pipenv shell` used for activation. And on Windows, sorry&#x2026;?
-   Upgrade package: `pip install <pkg> -U`
-   Controlled burn workflow:
    
    1.  Install top-level requirements and keep manual list in e.g. `requirements-toplevel.txt`
        `pip freeze > requirements.txt`
    2.  Whenever an included package is no longer included:
    
        pip freeze > uninstall.txt
        pip uninstall -r uninstall.txt
        pip install -r requirements-toplevel.txt
        pip freeze > requirements.txt

