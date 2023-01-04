# Reproducing procedure for pants+pyright+implicit namespace not working

## Without pants

Non-pants way to execute `pyright` in `libs/a`:

```shell
cd libs/a
python3 -m venv .venv
source .venv/bin/activate
pip install pip==22.2.2
pip install -r requirements.txt pyright==1.1.258
```

then run `pyright`, witness it works:

```shell
> pyright . # from the libs/a folder (see above)
WARNING: there is a new pyright version available (v1.1.258 -> v1.1.281).
Please install the new version or set PYRIGHT_PYTHON_FORCE_VERSION to `latest`

No configuration file found.
pyproject.toml file found at /home/churlin/dev/pants-implicit-namespaces-typecheck/libs/a.
Loading pyproject.toml file at /home/churlin/dev/pants-implicit-namespaces-typecheck/libs/a/pyproject.toml
Assuming Python version 3.10
Assuming Python platform Linux
Auto-excluding **/node_modules
Auto-excluding **/__pycache__
Auto-excluding **/.*
stubPath /home/churlin/dev/pants-implicit-namespaces-typecheck/libs/a/typings is not a valid directory.
Searching for source files
Found 1 source file
pyright 1.1.258
0 errors, 0 warnings, 0 informations
Completed in 0.564sec
```

## With pants

Pants fails typechecking `libs/a` in isolation 💣

```shell
> PANTS_SHA=df1c5b2418d9c62622b6039bb0fcc91b08f1dee4 ./pants check libs/a::
22:19:42.92 [ERROR] Completed: Typecheck using Pyright - pyright - pyright failed (exit code 1).
Loading configuration file at /tmp/pants-sandbox-U297Ol/pyrightconfig.json
Assuming Python version 3.10
Assuming Python platform Linux
Auto-excluding **/node_modules
Auto-excluding **/__pycache__
Auto-excluding **/.*
Searching for source files
Found 1 source file
pyright 1.1.258
/tmp/pants-sandbox-U297Ol/libs/a/mycorp/a/a_mod.py
  /tmp/pants-sandbox-U297Ol/libs/a/mycorp/a/a_mod.py:1:20 - error: "b" is unknown import symbol (reportGeneralTypeIssues)
1 error, 0 warnings, 0 informations 
Completed in 0.727sec

stubPath /tmp/pants-sandbox-U297Ol/typings is not a valid directory.



✕ pyright failed.
```

However, it succeeds typechecking the entire repo 🍰 ✨

```shell
> PANTS_SHA=df1c5b2418d9c62622b6039bb0fcc91b08f1dee4 ./pants check ::
22:19:50.43 [INFO] Completed: Typecheck using Pyright - pyright - pyright succeeded.
Loading configuration file at /tmp/pants-sandbox-9BvBCf/pyrightconfig.json
Assuming Python version 3.10
Assuming Python platform Linux
Auto-excluding **/node_modules
Auto-excluding **/__pycache__
Auto-excluding **/.*
Searching for source files
Found 3 source files
pyright 1.1.258
0 errors, 0 warnings, 0 informations 
Completed in 0.745sec

stubPath /tmp/pants-sandbox-9BvBCf/typings is not a valid directory.



✓ pyright succeeded.
```

Note that, in both cases, the correct number of files is found by pyright
(see the `Found X source file(s)` line).
