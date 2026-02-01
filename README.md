# rpt - Repository Tools

A collection of Python scripts for managing multiple Git repositories.

## Scripts

- **repo-clone.py** - Clone a repository by name, searching across multiple remotes
- **repo-status.py** - Show the status of all repositories in configured directories
- **repo-update.py** - Update (pull) all repositories in configured directories

## Setup

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Shell Aliases (optional)

Add the following to your `~/.bash_aliases` to run the scripts from anywhere without manually managing the virtual environment:

```bash
# Helper function to run rpt scripts with venv
function _rpt_run() {
    local script="$1"
    shift
    local rpt_dir="$HOME/repos/rpt"
    local venv_dir="$rpt_dir/venv"
    
    # Create venv if it doesn't exist
    if [ ! -d "$venv_dir" ]; then
        echo "Creating virtual environment..."
        python3 -m venv "$venv_dir" || return 1
    fi
    
    # Activate venv and install deps if needed
    source "$venv_dir/bin/activate"
    if ! python3 -c "import click" 2>/dev/null; then
        echo "Installing dependencies..."
        pip install -r "$rpt_dir/requirements.txt" || return 1
    fi
    
    # Run the script with all arguments
    "$rpt_dir/$script" "$@"
    
    deactivate
}

function rst() { _rpt_run "repo-status.py" "$@"; }
function ru() { _rpt_run "repo-update.py" "$@"; }
function rc() { _rpt_run "repo-clone.py" "$@"; }
```

This creates a single virtual environment at `~/repos/rpt/venv` that is reused regardless of where you call the commands from. Dependencies are installed automatically on first use.

- `rst` - repo-status.py
- `ru` - repo-update.py  
- `rc` - repo-clone.py

## Configuration Files

Both config files are optional and should be created in the root of this project. They are gitignored since they contain user-specific settings.

### repos.conf

Specifies local directories containing Git repositories. Used by `repo-status.py` and `repo-update.py` to know which directories to scan.

By default, `~/repos/` is always included. Add additional directories, one per line:

```
~/work/projects/
~/personal/code/
/opt/repos/
```

### remotes.conf

Specifies Git remote base URLs for `repo-clone.py` to search when cloning a repository by name. Add one base URL per line:

```
git@github.com:myusername/
git@github.com:myorg/
git@gitlab.com:myorg/
```

Lines starting with `#` are treated as comments.

When you run `repo-clone.py myrepo`, it will attempt to clone from each remote in parallel:
- `git@github.com:myusername/myrepo.git`
- `git@github.com:myorg/myrepo.git`
- `git@gitlab.com:myorg/myrepo.git`

If multiple remotes have the repository, all copies are cloned into numbered directories.
