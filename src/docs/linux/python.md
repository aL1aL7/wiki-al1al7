# Python Basics

## Python virtual environment

Using virtual python environments offer the following benefits:

* work on multiple projects with different dependencies at the same time
* create portable projects
* no risk of version conflicts
* avoid the need for global package installation

```bash title="Setup new virtual environment"
python -m venv /path/to/new/virtual/environment
```

 ```bash title="Activate virtual environment - Linux (bash/zsh)"
 source <path_to_virtual_environment>/bin/activate
 ```

 ```bash title="Activate virtual environment - Windows (PowerShell)"
 source <path_to_virtual_environment>/Scripts/Activate.ps1
 ```

```bash title="Deactivate virtual environment - Linux/Windows"
deactivate
```
