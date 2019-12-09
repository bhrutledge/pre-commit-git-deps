# Using a Git repo in pre-commit `additional_dependencies`

The released version of flake8 flags errors for the new `:=` operator in Python 3.8, because the released versions of pycodestyle and pyflakes don't support it. However, both projects have merged fixes into their `master` branches. I can workaround this in a virtual environment by installing pyflakes and pycodestyle from their Git repos:

```
python3.8 -m venv venv
source venv/bin/activate
pip install flake8 git+https://github.com/PyCQA/pyflakes git+https://gitlab.com/PyCQA/pycodestyle
flake8 test_walrus.py
```

In tox, this is possible by specifying `deps` for a `testenv`. However, the same approach doesn't seem to be working with pre-commit's `additional_dependencies`. I reported this as a [pre-commit issue](https://github.com/pre-commit/pre-commit/issues/1238), then discovered a workaround, which I've implemented in a [pull request](https://github.com/bhrutledge/pre-commit-git-deps/pull/1).

Output from initial commit:

```
$ tox
flake8 create: /Users/brian/Code/pre-commit-git-deps/.tox/flake8
flake8 installdeps: flake8, git+https://github.com/PyCQA/pyflakes, git+https://gitlab.com/PyCQA/pycodestyle
flake8 installed: entrypoints==0.3,flake8==3.7.9,mccabe==0.6.1,pycodestyle==2.5.0,pyflakes==2.1.1
flake8 run-test-pre: PYTHONHASHSEED='2831974751'
flake8 run-test: commands[0] | flake8 test_walrus.py
pre-commit create: /Users/brian/Code/pre-commit-git-deps/.tox/pre-commit
pre-commit installdeps: pre-commit
pre-commit installed: aspy.yaml==1.3.0,cfgv==2.0.1,identify==1.4.8,nodeenv==1.3.3,pre-commit==1.20.0,PyYAML==5.2,six==1.13.0,toml==0.10.0,virtualenv==16.7.8
pre-commit run-test-pre: PYTHONHASHSEED='2831974751'
pre-commit run-test: commands[0] | pre-commit run --all-files
[INFO] Initializing environment for https://gitlab.com/pycqa/flake8.
[INFO] Initializing environment for https://gitlab.com/pycqa/flake8:git+https://github.com/PyCQA/pyflakes,git+https://gitlab.com/PyCQA/pycodestyle.
[INFO] Installing environment for https://gitlab.com/pycqa/flake8.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
flake8...................................................................Failed
hookid: flake8

"pyflakes" failed during execution due to "'FlakesChecker' object has no attribute 'NAMEDEXPR'"
Run flake8 with greater verbosity to see more details
test_walrus.py:1:8: E203 whitespace before ':'
test_walrus.py:1:9: E701 multiple statements on one line (colon)
test_walrus.py:1:9: E231 missing whitespace after ':'

ERROR: InvocationError for command /Users/brian/Code/pre-commit-git-deps/.tox/pre-commit/bin/pre-commit run --all-files (exited with code 1)
___________________________________ summary ____________________________________
  flake8: commands succeeded
ERROR:   pre-commit: commands failed
```
