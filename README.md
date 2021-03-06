# Some `pre-commit` hooks I use

Various git pre-commit or pre-push hooks using http://pre-commit.com/. 


1. Have Python's pip package manager installed. `brew install python`
2. `pip install pre-commit`
    * If your version of `pre-commit` is less than 0.7, you need `pip install --upgrade pre-commit`
3. Put a `.pre-commit-config.yaml` at the root of a repo. See below.
4. `pre-commit install` # this installs pre-commit hooks
5. `pre-commit install --hook-type pre-push`

If you have many repos to protect, you can use `install-hooks-in-repo`
to shorten steps 3-5. For example, if this repo has been cloned to
where you clone all your repos, you can protect them all like this:

```
echo * | xargs -n1 pre-commit-hooks/install-hooks-in-repo pre-commit-hooks/dot-pre-commit-config.yaml.sample
```

`install-hooks-in-repo` will ignore non-directories and directories that aren't a git repo. It creates a log in `/tmp/install-hooks-in-repo.log`.

------------------

Below please find a short sample
`.pre-commit-config.yaml`. [dot-pre-commit-config.yaml.sample](https://github.com/marick/pre-commit-hooks/blob/master/dot-pre-commit-config.yaml.sample)
contains a longer one.

```yaml
-   repo: https://github.com/marick/pre-commit-hooks.git
    sha: 569f6300cbbedf177c56eda8023350620998ff8f
    stages: [commit, push]
    hooks:
    -   id: only-branch-pushes
        args: [prevent, ^(production|master)$]

    -   id: prohibit-suspicious-patterns
        args: [AKIA[[:alnum:]]{14}, --] # matches AWS keys

    -   id: prohibit-suspicious-files
        args: [".log$", --] # Don't commit log files. They can contain personal info.
```

General notes:
*  The `sha` can be updated to the repo's most recent version with `pre-commit autoupdate`.

Notes on `only-branch-pushes`:
* Despite the name, `only-branch-pushes` actually can be applied during the `commit` stage (as it is here, according to the `stages` key). The name is not so good, but I'm too lazy to change it.
* The first argument must be `prevent`. The second is used in a
  `=~`-style shell pattern match. That is, it's
  compared like this:
  
      if [[ "$branch" =~ $pattern ]]; then
  
  If you want to prohibit just one branch, such as `production`, do this:
  
      args: [prevent, ^production$]
  

Notes on `prohibit-suspicious-patterns`:
* The first arg searches for a pattern that matches AWS keys. I use it
  as a
  [suspenders and belt](http://www.investopedia.com/terms/b/belt-and-suspenders.asp)
  strategy, alongside the `detect-aws-keys` hook from the
  [standard repo](https://github.com/pre-commit/pre-commit-hooks). 
* The patterns are
  [Ruby regular expressions](http://ruby-doc.org/core-1.9.3/Regexp.html),
  compiled with `Regexp.compile`.
* As such, you can't use the "outside-the-pattern" syntax for
  modifiers like "match any case. That is, instead of `/TODO/i`, you
  must use this:
  
        args: ["(?i-mx:TODO)", --]

* If you want to disable checking for a particular line, include `git commit ok` on that line. (You can replace the spaces with any *single* character.)
* In violation of the spirit of `pre-commit`, this hook assumes Ruby already exists on your system.

Notes on `prohibit-suspicious-files`:
* Patterns are also Ruby regular expressions.
* The hook assumes Ruby already exists on your system.
