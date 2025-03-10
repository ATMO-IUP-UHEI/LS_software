# LS_software
This repository contains my tools for easy overview.
Some day, there will be a documentation on how to use them in here.

## submodules
The tools are included in this git repo as submodules.
The url to the github location is selected as the submodule url and each submodule contains the following post commit hook, which is to be placed into each submodules respective hooks folder (.git/hooks/post-commit):
This ensures that the root repo (this one) is always updated when one of the submodules has a change.

```
#!/bin/sh

SUBMODULE_PATH=$(git rev-parse --show-toplevel)
SUBMODULE_NAME=$(basename "$SUBMODULE_PATH")
ROOTREPO_PATH=$(dirname "$SUBMODULE_PATH")

echo "Updating root repository with lastest commit from $SUBMODULE_NAME..."

cd "$ROOTREPO_PATH" || { echo "Failed to navigate to root repo"; exit 1; }

if [ -d .git ]; then
    git add "$SUBMODULE_NAME"
    git commit -m "Updated submodule $SUBMODULE_NAME."
    git push origin main
else
    echo "not in root repo, skipping update."
fi
```
```
