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

```shell
# from the repo root
> PANTS_SHA=a4054868c11690874005999cb80100e4da107538 ./pants check ::
11:05:32.08 [WARN] The target libs/a/mycorp/a/a.py:../../mycorp-a imports `mycorp.b`, but Pants cannot safely infer a dependency because more than one target owns this module, so it is ambiguous which to use: ['libs/b/mycorp/b/__init__.py', 'libs/b/mycorp/b/__init__.py:../../mycorp-b'].

Please explicitly include the dependency you want in the `dependencies` field of libs/a/mycorp/a/a.py:../../mycorp-a, or ignore the ones you do not want by prefixing with `!` or `!!` so that one or no targets are left.

Alternatively, you can remove the ambiguity by deleting/changing some of the targets so that only 1 target owns this module. Refer to https://www.pantsbuild.org/v2.16/docs/troubleshooting#import-errors-and-missing-dependencies.
11:05:32.08 [WARN] The target libs/a/mycorp/a/a.py imports `mycorp.b`, but Pants cannot safely infer a dependency because more than one target owns this module, so it is ambiguous which to use: ['libs/b/mycorp/b/__init__.py', 'libs/b/mycorp/b/__init__.py:../../mycorp-b'].

Please explicitly include the dependency you want in the `dependencies` field of libs/a/mycorp/a/a.py, or ignore the ones you do not want by prefixing with `!` or `!!` so that one or no targets are left.

Alternatively, you can remove the ambiguity by deleting/changing some of the targets so that only 1 target owns this module. Refer to https://www.pantsbuild.org/v2.16/docs/troubleshooting#import-errors-and-missing-dependencies.
11:05:32.08 [WARN] Pants cannot infer owners for the following imports in the target libs/a/mycorp/a/a.py:../../mycorp-a:

  * mycorp.b (line: 1)

If you do not expect an import to be inferrable, add `# pants: no-infer-dep` to the import line. Otherwise, see https://www.pantsbuild.org/v2.16/docs/troubleshooting#import-errors-and-missing-dependencies for common problems.
11:05:32.08 [WARN] Pants cannot infer owners for the following imports in the target libs/a/mycorp/a/a.py:

  * mycorp.b (line: 1)

If you do not expect an import to be inferrable, add `# pants: no-infer-dep` to the import line. Otherwise, see https://www.pantsbuild.org/v2.16/docs/troubleshooting#import-errors-and-missing-dependencies for common problems.
11:05:32.59 [INFO] Completed: Building requirements.pex
11:05:33.40 [INFO] Completed: Building requirements_venv.pex
11:05:35.05 [ERROR] Completed: Typecheck using Pyright - pyright - pyright failed (exit code 1).
Loading configuration file at /tmp/pants-sandbox-u0dO6d/pyrightconfig.json
Assuming Python version 3.10
Assuming Python platform Linux
Auto-excluding **/node_modules
Auto-excluding **/__pycache__
Auto-excluding **/.*
Searching for source files
Found 3 source files
pyright 1.1.258
/tmp/pants-sandbox-u0dO6d/libs/a/mycorp/a/a.py
  /tmp/pants-sandbox-u0dO6d/libs/a/mycorp/a/a.py:1:20 - error: "b" is unknown import symbol (reportGeneralTypeIssues)
1 error, 0 warnings, 0 informations
Completed in 0.752sec

stubPath /tmp/pants-sandbox-u0dO6d/typings is not a valid directory.
```
