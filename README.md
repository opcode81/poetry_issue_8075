# Poetry Bug Reproduction for Issue #9075

Steps to reproduce (tested under Ubuntu):

1. Backup existing cache
   
       mv ~/.cache/pypoetry ~/.cache/pypoetry.bak 

2. Link `./pypoetry` as the new cache

       ln -s "$(readlink -f ./pypoetry)" ~/.cache/pypoetry

3. Create a Python 3.11 environment, e.g. via conda

       conda create -n poetry_cache_analysis python=3.11

4. Apply `poetry install` in the environment:

       conda activate poetry_cache_analysis
       poetry install

5. Start the Python interpreter and observe that a class which is present in the requested commit hash of tianshou cannot be imported:

       >>> from tianshou.highlevel.env import EnvMode
       Traceback (most recent call last):
       File "<stdin>", line 1, in <module>
       ImportError: cannot import name 'EnvMode' from 'tianshou.highlevel.env'

6. Observe that uninstalling tianshou, deleting the cache and repeating the install resolves the issue:

        pip uninstall tianshou
        rm ~/.cache/pypoetry
        rm poetry.lock
        poetry install

    (Removing the lock file probably isn't necessary.)
    Then in Python:

        >>> from tianshou.highlevel.env import EnvMode
        >>> EnvMode
        <enum 'EnvMode'>

    Therefore, Poetry incorrectly resolved the requested tianshou version and used one of the older, cached versions even though that version is not associated with the requested commit hash.

7. Restore the cache we backed up up in the first step.