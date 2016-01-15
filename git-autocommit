#!/bin/bash

GIT_AUTOCOMMIT_VERSION="1.3"
GIT_AUTOCOMMIT_MESSAGE="<git-autocommit>"
GIT_AUTOCOMMIT_INTERVAL_SECONDS=300 # Five minutes.
GIT_AUTOCOMMIT_VERBOSE=0
GIT_AUTOCOMMIT_QUIET=0

git_autocommit_commit_count=0

set -o errexit

function fatal
{
    echo $1 >&2
    exit 1
}

function print_autocommit_stats
{
    if [ $GIT_AUTOCOMMIT_QUIET -eq 0 ]; then
        ((++git_autocommit_commit_count))
        printf "\rLast commit: $(date +'%Y-%m-%d %H:%M:%S'), commits: $git_autocommit_commit_count ..."
    fi
}

function verbose_log
{
    if [ $GIT_AUTOCOMMIT_VERBOSE -ne 0 ] && [ $GIT_AUTOCOMMIT_QUIET -eq 0 ]; then
        echo ""
        echo "$1"
    fi
}

# Returns 0 if changes have been auto-commited, 1 if not.
function autocommit
{
    local files=$(git status --porcelain)
    if [ ! -z "$files" ]; then
        verbose_log "$files"
        # Add everything that is there.
        git add -A >/dev/null
        # Commit
        git commit -q -m "$GIT_AUTOCOMMIT_MESSAGE" >/dev/null
        print_autocommit_stats
        return 0
    fi

    return 1
}

# Returns 0 if true, 1 if false.
function is_last_commit_autocommit
{
    local line=$(git log --pretty=oneline -n 1)
    local array_line=($line)
    local commit=${array_line[0]}
    local message=${array_line[1]}

    if [ "$message" == "$GIT_AUTOCOMMIT_MESSAGE" ]; then
        return 0 # Yes
    fi

    return 1 # No
}

function soft_reset_to_parent_commit
{
    local line_count=0
    autocommit_count=0
    git log --pretty=oneline | while read line; do
        ((++line_count))
        local array_line=($line)
        local commit=${array_line[0]}
        local message=${array_line[1]}

        if [ "$message" != "$GIT_AUTOCOMMIT_MESSAGE" ]; then
            # Stop if the last commit is not an autocommit message.
            if [ $line_count -eq 1 ]; then
                return
            fi

            echo "Soft resetting to $line, squashed $autocommit_count autocommit(s)."
            git reset --soft "$commit"
            echo "Now commit your combined changes or execute 'git reset HEAD@{1}' to undo."
            return
        else
            ((++autocommit_count))
        fi
    done
    if [ "$(git ls-files --modified --others --exclude-standard | wc -l)" -gt 0 ]; then
        echo "WARNING: There are local, unstaged modifications."
    fi
}

function show_help
{
cat <<EOM
git-autocommit -- Periodically commits uncommitted changes.
Version $GIT_AUTOCOMMIT_VERSION, Copyright 2015 by Ralf Holly.
Licensed under the terms of the MIT License, see LICENSE file.

Options:
    -h          Show help.
    -s          Soft reset to parent commit of an auto-commit sequence.
                (useful for squashing autocommits into a meaningful topic commit.)
    -i <secs>   Set check-for-modifications interval to <secs> seconds.
    -q          Quiet (no output).
    -V          Enable verbose output.
EOM
}

# If ':' at beginning -> silent error mode!
while getopts ":hsVqi:" opt; do
    case $opt in
        h)
            show_help
            exit
            ;;
        s)
            soft_reset_to_parent_commit
            exit
            ;;
        V)
            GIT_AUTOCOMMIT_VERBOSE=1
            ;;
        i)
            GIT_AUTOCOMMIT_INTERVAL_SECONDS=$OPTARG
            if [ -z "$GIT_AUTOCOMMIT_INTERVAL_SECONDS" ] || [ $GIT_AUTOCOMMIT_INTERVAL_SECONDS -le 0 ]; then
                fatal "Please provide a positive poll interval"
            fi
            verbose_log "Setting poll interval to $GIT_AUTOCOMMIT_INTERVAL_SECONDS seconds"
            ;;
        q)
            GIT_AUTOCOMMIT_QUIET=1
            ;;
        :)
            fatal "Option -$OPTARG requires an argument"
            ;;
        ?)
            fatal "Wrong option provided: -$OPTARG"
            ;;
    esac
done

# If there are no changes that have been auto-committed just now.
if ! autocommit; then

    if is_last_commit_autocommit; then
        echo -n "There are autocommits. Squash first? [y/N] "
        read answer
        case "$answer" in
            [yY] | yes | YES)
                soft_reset_to_parent_commit
                exit
                ;;
        esac
    fi
fi

while true; do
    autocommit || true
    sleep $GIT_AUTOCOMMIT_INTERVAL_SECONDS
done

