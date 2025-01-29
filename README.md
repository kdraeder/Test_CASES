# Test_CASES
Test the rule set options and inheritance to forked repositories

Motivation; protect branch names once they have been pushed to github.

Summary of what's available
---------------------------

Goals;
*  prevent branch names from being "reused", which could result in existing data
      being overwritten (but still stored in the earlier commits)
      and confusion between 2 experiments that should be kept separate.
*  prevent non-owners or non-teammates from deleting or pushing to an existing branch.
*  introduce minimal work to the process of setting up experiments and maintaining the repository.

A first strategy to meet those goals is to have code in the `case` setup script that checks for
previous use of the name.  This relies on the user having the current branch list.\
`cases` is a local directory where multiple cases are created.\
`${case}_git` is a directory created under git control.\
`caseroot` is the CESM case directory created in ${case}_git.\
```
setenv caseroot     ${cases}/${case}_git/${case}
...
git clone git@github.com:NCAR/DART_CASES.git  ${cases}/${case}_git
git branch -a | grep $case
if ($status == 0) then
   echo "ERROR: branch $case already exists.  Pick a new case name"
   exit 9
endif

${CIMEROOT}/scripts/create_newcase --case $caseroot ...
...
cd ${caseroot}

...most of the setup script...

# Put experiment under git(hub) control
git checkout -b ${case}
# Add the important files to the repository
./list_of_files_to_commit.csh
git commit -m "First commit of files in case $case "

# After everything is ready to run
git push -u origin ${case}

```


A branch can be protected individually, but that doesn't seem to offer the protections I want.

To define rule sets, which can apply to multiple branch names, under the repo name click through \
`Settings: Code and Automation: Rules: Rulesets: New Ruleset:`
  - "New branch ruleset"
  - "New tag ruleset"
  - "Import a ruleset"
Check that it applies to "All" in the menu of branches.

Rule Sets
+   A named list of rules that applies to a repository.
+   Allow certain users to bypass the rules in the ruleset.  (bypass users can only be prevented from doing these things in repos
      set up by organizations that use GitHub Enterprise Cloud)
+   Use rulesets to target branches or tags in a repository.
    -   Use fnmatch to define patterns to target.
    -   \* matches any char string except /.
    -   include any number of slashes after qa with qa/\*\*/\*
    -   extend the qa string with qa\*\*/\*\*/\*
    -   \[ \] for selected characters, but [^ ] doesn't work.
+   Block pushes to a repository and the repository's entire fork network.\
    I want to prevent 
+   Push rules do not require any branch targeting because they apply to every push to the repository.\
     Can restrict file paths, extensions, and size.  
+   Rule sets can work in tandem with branch protection.
+   Advantages;
    - Multiple rule sets can apply simultaneously
    - Can be turned on or off (status switch)
    - People with read access can view the active rule sets.
    - rules can control the metadata of commits; message, author's email

Forks of the repository inherit the push rule sets of the repository. ("y" = available, "i" = inherited).\
These were harvested from `Settings: General` of base and forked repositories.
```
Base  Fork
  y     i      Restrict creations (to people with bypass)
  y     i      Restrict updates (to...)
  y     i      Restrict deletions (to...)
  y     i      Block force pushes (which can overwrite other users commits)
  y     i      Require merge queue
  y     i      Require code scanning results
  y     y      Require a pull request before merging
  y     y      Require status checks to pass before merging
        y      Require conversation resolution before merging
  y     y      Require signed commits
  y     y      Require linear history
  y     y      Require deployments to succeed before merging
        y      Lock branch
        y      Do not allow bypassing the above settings
