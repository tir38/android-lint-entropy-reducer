#Android Lint Entropy Reducer

Over time Lint errors and warnings build up in a project. It's a daunting task to remove all of these errors at once. It's also not very helpful to fix a bunch of errors if more keep cropping up over time. By including this script as part of your build process, you can always ensure that the number of Lint errors and warnings never rises above current levels. Any reduction in errors or warnings count will be preserved. This will trend the number of Lint errors/warnings to zero.




##First Time Setup
0. ensure that Travis is set to "build pull requests"
1. create new branch
2. include this script in project and set executable `chmod +x lint-up.rb`
3. setup script parameters (see below)
3. update `travis.yml` (see below)
4. commit changes
5. push to origin
6. open PR against new branch
7. The script will run and create a baseline of existing error/warning count
8. merge new branch into master

Now any new pull requests will have to maintain or reduce the number of Lint errors/warnings.


##Update Travis.yml

Include this script in your `travis.yml` file:

```
script:
    - ./gradlew clean assembleDebug
    - ./gradlew clean test
    - ./scripts/lint-up.rb
```

You are encouraged to place this script in a separate directory (here called `scripts`). It also makes sense to run Lint *after* assemble and test. There is no point in running Lint checks if your app isn't compiling and passing tests.

Because any increase in Lint errors/warnings should fail the build, you need to call this script from the `script` stage. Read more about breaking Travis builds [here](http://docs.travis-ci.com/user/customizing-the-build/#Breaking-the-Build).

##Script Parameters

At the top of the lint-up.rb file is a list of user-adjustable parameters that define the behavior of the script. Defaults have been set so that you don't **have** to make any changes.

####TRAVS_GIT_USERNAME

At some point, if the number of errors or warnings decreases, this script will create a new git commit with updated results. These results will become the new baseline for future checks. This parameter will set the git username used by the script.


```
# User name for git commits made by this script.
TRAVS_GIT_USERNAME="Travis CI server"
```

####LINT_REPORT_FILE

In your build.gradle file, you set where the generated Lint report should be saved. This script needs access to that file.


```
# File name and relative path of generated Lint report. 
# Must match build.gradle file:
#   lintOptions {
#       htmlOutput file("[FILE_NAME].html")
#   }
LINT_REPORT_FILE="app/lint-report.html"
```

####PREVIOUS_LINT_RESULTS_FILE

This script will generate a baseline report file that represents the current error/warning count. This file's contents will be updated as the error/warning count is reduced. You should not move this file after beginning using this script. If you do, be sure to update this setting.

```
# File name and relative path of previous results of this script.
PREVIOUS_LINT_RESULTS_FILE="lint-report/lint-results.xml"
```

####CHECK_WARNINGS 

This script can perform separate reduction checks for errors as well as warnings. If you choose to ignore warnings in your build.gradle file, you can choose to ignore warnings in this script (and only track errors).

```
# Flag to evaluate warnings. true = check warnings; false = ignore warnings
CHECK_WARNINGS=true
```

####CUSTOM_LINT_FILE

This script can also run custom Lint rules. They need to be included in a separate .jar or .aar. Read more [here](https://engineering.linkedin.com/android/writing-custom-lint-checks-gradle) and [here](http://tools.android.com/tips/lint-custom-rules). This parameter can be left blank (either nil or an empty string)

```
# File name and relative path to custom lint rules; Can be nil or "".
CUSTOM_LINT_FILE="lint_rules/lint.jar"
```