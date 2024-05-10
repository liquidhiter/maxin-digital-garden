---
{"dg-publish":true,"permalink":"/00-notes/error-terminator/","noteIcon":"","created":"2024-01-26T23:38:05.074+01:00","updated":"2024-01-26T23:48:49.553+01:00"}
---

- git pull

    ```bash
    hint: You have divergent branches and need to specify how to reconcile them.
    hint: You can do so by running one of the following commands sometime before
    hint: your next pull:
    hint: 
    hint:   git config pull.rebase false  # merge (the default strategy)
    hint:   git config pull.rebase true   # rebase
    hint:   git config pull.ff only       # fast-forward only
    hint: 
    hint: You can replace "git config" with "git config --global" to set a default
    hint: preference for all repositories. You can also pass --rebase, --no-rebase,
    hint: or --ff-only on the command line to override the configured default per
    hint: invocation.
    fatal: Need to specify how to reconcile divergent branches.
    ```
    - cause of the issue: 
        - Git wants to know how it should handle the situation where our remote branch is out of sync with our
        local branch.
    - solution: `git pull --rebase`
    - suggested reading: 
        - [Git merge strategy options & examples | Atlassian Git Tutorial](https://www.atlassian.com/gittutorials/using-branches/merge-strategy)
        - [git - Error "Fatal: Not possible to fast-forward, aborting" - Stack Overflow](https://stackoverflowcom/questions/13106179/error-fatal-not-possible-to-fast-forward-aborting)

------------------------------------------------------------------------------------------------------------
- Missing new line complained by Pycharm
    - REF: https://stackoverflow.com/questions/5813311/whats-the-significance-of-the-no-newline-at-end-of-file-log
    - Similar warnings can be seen in the git tools
    - Cause: historically come back to the Unix system, at that time a new was determined to be a must-have end file character to simplify the process of files. (Imagine add the contents of one file to another or concatenate two files together...)

------------------------------------------------------------------------------------------------------------
- Unsupported bracket in Jelly
    ```bash
     Incremental build: disabled. Enable with --incremental
      Generating... 
       Jekyll Feed: Generating feed for posts
       Imagemagick: Disabled in site.config
         AutoPages: Disabled/Not configured in site.config.
        Pagination: Complete, processed 1 pagination page(s)
    Liquid Exception: Liquid syntax error (line 54): Variable '\/{\/{0, 0\/}' was not properly terminated with regexp: /\}\}/ in /home/runner/work/liquidhiter.github.io/liquidhiter.github.io/_posts/2023-06-24-coursera-algorithm-specialization-week2.md
                        ------------------------------------------------
        Jekyll 4.3.2   Please append `--trace` to the `build` command 
                        for any additional information or backtrace. 
                        ------------------------------------------------
    ```
    - code resulted in the above error: `std::unordered_map<int, long long> memorizedFibonacciNums /\{/\{0, 0/\} , /\{1, 1/\}/\};`
    - solution:
        ```c++
         std::unordered_map<int, long long> memorizedFibonacciNums;
         memorizedFibonacciNums[0] = 0;
         memorizedFibonacciNums[1] = 1;
        ```
    - note:
        - to pass the compilation: `/\` are added before each bracket manually.... 
        - this is actually very stupid. I don't know why Jelly complained about the common usage of initializer list...

------------------------------------------------------------------------------------------------------------