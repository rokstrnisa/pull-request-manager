# Pull Request Manager

## Description

A program that continually checks and potentially merges
[GitHub pull requests](http://help.github.com/pull-requests/).

## Protocol

The protocol used to choose a pull request to process is the following:

1. Find all open pull requests associated with the specified repositories;
2. From those, select only pull requests made by the administrators of these
   repositories;
3. For each pull request:
   * read its comments, and determine whether the bot's last attempt has
     `succeeded`, and whether either the repository's branch to which the pull
     request was made, or the pull request itself, has `changed` since the
     last attempt;
   * if an administrator of the repository has posted a comment that starts
     with `@<bot_name>`, followed by a regular expression specified on
     a line in `positive.txt` (case ignored), followed by `.` or `!`,
     followed by any text, and the bot has previously `succeeded` or the refs
     `changed`, prefer processing this pull request;
   * otherwise, process a pull request only if no pull request satisfies the
     previous step, and the corresponding refs of the pull request have
     `changed`.

Once a pull request has been chosen, it is processed as follows:

1. Try building the underlying system with the changesets from the pull
   request:

        make <component-for-pull-request's-repository>-build
        make api-build

   If these steps fail, report the problem as a comment on the pull request,
   and stop processing this pull request.

   While currently not enabled, it is easy to configure the bot to only
   re-build the system if refs have not `changed`. This is disabled, since it
   is safer to re-build the system before a merge even if the refs have not
   `changed`, because the bot is not checking refs of all dependent
   repositories (many of which are not on GitHub).

2. If a merge is requested (through administrator's _positive_ comment; see
   above), verify that refs have not `changed` since the pull request started
   being processed:
   * if refs have `changed`, report the problem as a comment on the pull
     request, and stop processing this pull request;
   * otherwise, push the changesets of the pull request into the requested
     branch of the main repository, comment about this on the pull request, and
     close the pull request.

The above process is repeated continuously. After every processed pull request,
or if there is no pull request to process, the program waits for a while
(1min). If a connection error occurs, the program waits for a longer period of
time (10min) before retrying. Privileges are refreshed every 5 runs.

In a pull request's description, one can express its dependencies on other pull
request. For example, one can write the following:

    Dependencies: 23@my-repo

The above implies that this pull request will not be processed before pull
request `23` in `my-repo` (of the same GitHub organisation) has been merged.

## Dependencies on other libraries

The program currently depends on:

* [python-github2](https://github.com/xen-org/python-github2), a Python
implementation of [GitHub's API](http://develop.github.com/).

## Setup

Clone the module and its submodule(s) with:

    git clone git://github.com/xen-org/pull-request-manager.git
    cd pull-request-manager
    git submodule init
    git submodule update

The following steps are required to run the program:

* Make sure the dependencies are satisfied;

* Create `settings.py`, and within define `bot_email`, `bot_api_token`, and
  `builds_path` variables with appropriate values; and,

* Create a `github-xen-git` target in your `.ssh/config` file, which points to
the bot's private key:

          Host github-xen-git
            HostName github.com
            User git
            IdentityFile /home/roks/.ssh/id_rsa_xen_git

## Running

To start the program, execute the following command:

    python main.py

## Extras

The tool also features support for
[JIRA](http://www.atlassian.com/software/jira/) through the
[jiralib](https://github.com/xen-org/jiralib) project. To use it, define
`jira_url`, `jira_username`, and `jira_password` variables in `settings.py`.
See the `closeTicket` function within `main.py` for more information.

## Feedback and Contributions

Feedback and contributions are welcome. Please submit contributions
via GitHub pull requests.
