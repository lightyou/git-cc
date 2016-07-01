# git-cc

Simple bridge between base ClearCase or UCM and Git.

## Warning

I wrote this purely for fun and to see if I could stop use ClearCase at work
once and for all.

I will probably continue to hack away at it to suite my needs, but I would
love to see it get some real-world polish. (Actually what I would love to see
more is for ClearCase to die, but don't think that's going to happen any time
soon).

Suggestions on anything I've done are more than welcome.

Also, I have made a change recently to support adding binary files which uses
git-cat. Unfortunately git-cat doesn't handle end line conversions and so I
have made gitcc init set core.autocrlf to false. This is only relevant for
Windows users. Don't try changing this after your first commit either as it
will only make matters worse. My apologies to anyone that is stung by this.

## Installation

The easiest way to install git-cc, is to use the Python package installer pip
and install it directly from its GitHub repo. Execute the following command on
the command prompt to install the latest version:

    C:\> pip install git+git://github.com/charleso/git-cc.git#egg=git_cc

If you installed Python from python.org, pip is included with Python 2 >= 2.7.9
and Python 3 >= 3.4. If you do not have pip, [this section] [pip-installation]
from the Python Packaging User Guide describes how to install it.

In case pip or git cannot reach GitHub, for example when such access is not
allowed in the place where you work, you can download the [zip file] [zip-file]
with the latest version from the GitHub repo. Unzip it and use pip to execute
the following command in the root of the directory tree:

    C:\master> pip install .
    
Finally, if you cannot use pip, you can also use the old-skool approach to
install Python packages:

    C:\master> python setup.py install

## Workflow

Initialise:

    git init
    gitcc init d:/view/xyz
    gitcc rebase
    # Get coffee
    # Do some work
    git add .
    git commit -m "I don't actually drink coffee"
    gitcc rebase
    gitcc checkin

Initialise (fast):

Rebase can be quite slow initially, and if you just want to get a snapshot of
ClearCase, without the history, then this is for you:

    gitcc init d:/view/xyz
    gitcc update "Initial commit"

Other:

These are two useful flags for rebase which is use quite frequently.

    gitcc rebase --stash

Runs stash before the rebase, and pops it back on afterwards.

    gitcc rebase --dry-run

Prints out the list of commits and modified files that are pending in ClearCase.

To synchronise just a portion of your git history (instead of from the
very first commit to HEAD), mark the start point with the command:

    gitcc tag <commit>

To specify an existing ClearCase label while checking in, in order to let your
dynamic view show the version of the element(s) just checked in if your
confspec is configured accordingly, use the command:

    gitcc checkin --cclabel=YOUR_EXISTING_CC_LABEL

Note that the CC label will be moved to the new version of the element, if it is already used.

## Configuration

The file .git/gitcc contains configuration options for gitcc. For example, it
allows you to limit which branches and folders you import from:

    [core]
    include = FolderA|FolderB
    exclude = FolderA/sub/folder|FolderB/other/file
    users_module_path = users.py
    debug = False
    type = UCM
    [master]
    clearcase = D:\views\co4222_flex\rd_poc
    branches = main|ji_dev|ji_*_dev|iteration_*_dev
    [sup]
    clearcase = D:\views\co4222_sup\rd_poc
    branches = main|sup

In this case there are two separate git branches, master and sup, which
correspond to different folders/branches in ClearCase.

You can add a mapping for each user in your ClearCase history. This is done via
a separate Python module that you provide. An example users module looks
like this:

    users = {
        'charleso': "Charles O'Farrell",\
        'jki': 'Jan Kiszka <jan.kiszka@web.de>',\
    }
     
    mailSuffix = 'example.com'

You specify the path to the users module as the value of key
'users\_module\_path' in the gitcc config file. In the example above, the value
specified is 'users.py'. If the path is relative, it is taken relative to the
location of the config file. So in this example, gitcc will import users.py
from directory .git. But you can also use an absolute users module path.

If you do not specify the users module path in the config file, the ClearCase
user information will be used.

## Notes

Can either work with static or dynamic views. I use dynamic at work because
it's much faster not having to update. I've done an update in rebase anyway,
just-in-case someone wants to use it that way.

Can also work with UCM, which requires the 'type' config to be set to 'UCM'.
This is still a work in progress as I only recently switched to this at work.
Note the the history is still retrieved via lshistory and not specifically from
any activity information. This is largely for convenience for me so I don't have
to rewrite everything. Therefore things like 'recommended' baselines are ignored.
I don't know if this will cause any major dramas or not.

## Troubleshooting

1. WindowsError: [Error 2] The system cannot find the file specified

You're most likely running gitcc under Windows Cmd. At moment this isn't
supported. Instead use Git Bash, which is a better console anyway. :-)

If you have both msysgit and Cygwin installed then it may also be
[this](https://github.com/charleso/git-cc/issues/10) problem.

2. cleartool: Error: Not an object in a vob: ".".

The ClearCase directory you've specified in init isn't correct. Please note
that the directory must be inside a VOB, which might be one of the folders
inside the view you've specified.

3. fatal: ambiguous argument 'ClearCase': unknown revision or path not in the working tree.

If this is your first rebase then please ignore this. This is expected.

4. pathspec 'master_cc' did not match any file(s) known to git

See Issue [8](https://github.com/charleso/git-cc/issues/8).

## Behind the scenes

A smart person would have looked at other git bridge implementations for
inspiration, such as git-svn and the like. I, on the other hand, decided to go
cowboy and re-invent the wheel. I have no idea how those other scripts do their
business and so I hope this isn't a completely stupid way of going about it.

I wanted to have it so that any point in history you could rebase on-top of the
current working directory. I've done this by using the ClearCase commit time
for git as well. In addition the last rebased commit is tagged and is used
to limit the history query for any chances since. This tagged changeset is
therefore also used to select which commits need to be checked into ClearCase.

## Problems

It is worth nothing that when initially importing the history from ClearCase
that files not currently in your view (ie deleted) cannot be reached without
a config spec change. This is quite sad and means that the imported history is
not a true one and so rolling back to older revisions will be somewhat limited
as it is likely everything won't compile. Other ClearCase importers seem
restricted by the same problem, but none-the-less it is most frustrating. Grr!

[pip-installation]: https://packaging.python.org/en/latest/installing/#requirements-for-installing-packages
[zip-file]: https://github.com/charleso/git-cc/archive/master.zip