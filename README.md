Hi,

I'm using Tower with a JavaScript / Node project that uses Husky (https://github.com/typicode/husky) and lint-staged (https://github.com/okonet/lint-staged) to prevent developers (such as myself) from committing JavaScript source files which fail to pass ESLint (http://eslint.org/), i.e. a staged file has problems which the developer should fix before committing it.

Supposedly Tower supports git hooks (https://www.git-tower.com/help/mac/faq-and-tips/faq/hook-scripts), such as the `pre-commit` script that Husky configures after installing it (or after running `npm install`). Husky's pre-commit script is indeed executed if I use command-line `git` to stage and then attempt to commit a file with ESLint errors. However, Tower does not appear to invoke the pre-commit script, thereby allowing me to commit a source file with errors in it.

I have no idea why this is happening, so below you'll find my best effort to document steps to reproduce.

## Relevant URLs
- Reduced Test Case GitHub Project: https://github.com/kohlmannj/git-tower-husky-precommit-bug
- Video of the Issue: https://vimeo.com/217758811

## Steps to Reproduce
1. Clone the repository https://github.com/kohlmannj/git-tower-husky-precommit-bug
2. Open the cloned working copy in Terminal and run `npm install`.
3. Open index.js in the folder and uncomment the second line of the file (which has an error which ESLint cannot auto-fix).
4. Use Tower or command-line Git to stage index.js.
5. Use Tower or command-line Git to make a commit

## Test Environment
- macOS Sierra 10.12.5
- Tower 2.6.1 (Build 347)
- Git 2.12.0
- Node 6.10.2
- NPM 4.5.0

## Expected Behavior (reproducible with command-line Git)
1. The remote repository is successfully cloned to local folder "git-tower-husky-precommit-bug"
2. NPM successfully installs the dependencies specified in package.json. This includes configuring Husky and lint-staged, which may print their own initialization messages in addition to NPM's command line output.
3. The second line of index.js, which contains a variable named "obvious_lint_error", is un-commented. This variable naming style violates the ESLint "camelcase" rule as an "error", meaning that ESLint cannot auto-fix the problem.
4. Git successfully stages index.js without any issues.
5. The Git pre-commit script set up by Husky is invoked before committing, which in turn runs ESLint on index.js. ESLint finds two errors which cannot be auto-fixed: "camelcase" and "no-unused-vars". Because of these errors, the Git commit is aborted.

## Actual Behavior (reproducible with Tower)
1. Same as Expected
2. Same as Expected 
3. Same as Expected
4. Same as Expected
5. Tower fails to execute the Husky-prepared pre-commit script, which in turns allows the ESLint-violating index.js to be committed despite the fact that it contains errors.

## How Husky + lint-staged + ESLint + Git are Supposed to Work
When attempting to make a commit with Git:

1. git invokes the pre-commit script written by Husky
2. the pre-commit script executes `npm run precommit`
3. the NPM `precommit` script runs lint-staged
4. lint-staged runs `eslint --fix` on staged JavaScript files only
5. eslint exits with an error if there are still errors in the staged JavaScript files, even after ESLint has autofixed everything it can
6. git fails to commit the staged files due to eslint exiting with errors

## Video of the Issue
I've created a screencast of the issue here: https://vimeo.com/217758811

This video shows command-line git correctly preventing me from committing an ESLint-violating staged file, while Tower incorrectly allows me to commit the file.
