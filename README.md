# action-cleanup
Github action for remove temp files after job run.

## Description

Path's to remove should pass via envs with prefix `CLEANUP_PATH_`
directly with `env` action key or via `$GITHUB_ENV`.

Every path will remove with command `rm -rf`.

If found variable with empty value or with next values:
- `.`
- `..`
- `/`
action will skip this envs.

Action try to remove all passed path's. If at least one path not removed, action will fail.

If action should not fail, pass `continue-on-error: true` to action, like:

```yaml
- name: Run action
  uses: name212/action-cleanup@v1
  continue-on-error: true
```

## Usage

```yaml
- uses: name212/action-cleanup@v1
  # Pass path's via envs with prefix CLEANUP_PATH_
  env:
    CLEANUP_PATH_BUILD_DIR: "./build"
  with:
    ##! if uses or action_dir were not passed. Action will fail 
    
    # Run `rm` with `-v` flag and enable some debug logs.
    # Optional. Default 'true'
    verbose: 'true'
```

## Examples

### Pass envs directly from outputs

```yaml
name: Pull request changes
on:
  pull_request:
    types:
      - opened
      - synchronize
      - reopened

jobs:
  my:
    name: "Run cleanup tests pass directly via envs"
    runs-on: ubuntu-latest
    timeout-minutes: 5

    steps:
    - &checkout_step
      name: Checkout
      uses: actions/checkout@v6.0.2
      with:
        fetch-depth: 0
        submodules: "recursive"
        ref: ${{ github.event.pull_request.head.sha }}

    - name: Create files
      id: files
      run: |
        empty=""
        not_exists=".not_exists"
        first=".first_file"
        second="$(realpath .)"
        second="${second}/second-file"

        dir_with_files="./my-dir"

        mkdir -p "$dir_with_files"

        echo "first" > "$first"
        echo "second" > "$second"
        echo "third" > "${dir_with_files}/file.txt"
        
        echo "first=${first}" >> $GITHUB_OUTPUT
        echo "second=${second}" >> $GITHUB_OUTPUT
        echo "empty=${empty}" >> $GITHUB_OUTPUT
        echo "not_exists=${not_exists}" >> $GITHUB_OUTPUT
        echo "dir_with_files=${dir_with_files}" >> $GITHUB_OUTPUT

    - name: Run action
      uses: name212/action-cleanup@v1
      env:
        CLEANUP_PATH_FILE_FIRST: ${{ steps.files.outputs.first }}
        CLEANUP_PATH_FILE_SECOND: ${{ steps.files.outputs.second }}
        CLEANUP_PATH_FILE_EMPTY: ${{ steps.files.outputs.empty }}
        CLEANUP_PATH_FILE_NOT_EXISTS: ${{ steps.files.outputs.not_exists }}
        CLEANUP_PATH_FILE_NOT_EXISTS_IN_OUTPUTS: ${{ steps.files.outputs.no_output }}
        CLEANUP_PATH_DIR: ${{ steps.files.outputs.dir_with_files }}
```

### Pass envs via GITHUB_ENV

```yaml
name: Pull request changes
on:
  pull_request:
    types:
      - opened
      - synchronize
      - reopened

jobs:
  my:
    name: "Run cleanup tests envs via GITHUB_ENV"
    runs-on: ubuntu-latest
    timeout-minutes: 5

    steps:
    - &checkout_step
      name: Checkout
      uses: actions/checkout@v6.0.2
      with:
        fetch-depth: 0
        submodules: "recursive"
        ref: ${{ github.event.pull_request.head.sha }}

    - name: Create files
      id: files
      run: |
        empty=""
        not_exists=".not_exists"
        first=".first_file"
        second="$(realpath .)"
        second="${second}/second-file"

        dir_with_files="./my-dir"

        mkdir -p "$dir_with_files"

        echo "first" > "$first"
        echo "second" > "$second"
        echo "third" > "${dir_with_files}/file.txt"
        
        echo "CLEANUP_PATH_FILE_FIRST=${first}" >> $GITHUB_ENV
        echo "CLEANUP_PATH_FILE_SECOND=${second}" >> $GITHUB_ENV
        echo "CLEANUP_PATH_FILE_EMPTY=${empty}" >> $GITHUB_ENV
        echo "CLEANUP_PATH_FILE_NOT_EXISTS=${not_exists}" >> $GITHUB_ENV
        echo "CLEANUP_PATH_DIR=${dir_with_files}" >> $GITHUB_ENV

    - name: Run action
      uses: name212/action-cleanup@v1
```