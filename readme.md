See my [Travis builds](https://travis-ci.org/deanturpin).

# Travis CI - repo configuration

## C++11 - simple builds
If you just want to get something building quickly the default Trusty build
has clang5 pre-installed. No need for complicated matrices.

## C++ builds
Building for clang 5 and gcc 7. These could be run as separate jobs but it you
then have to deal with each build trying to deploy.
```yaml
script:
  - make CXX=clang++-5.0
  - make clean
  - make CXX=g++-7
  - cppcheck --enable=all .

matrix:
  include:
    - os: linux
      addons:
        apt:
          update: true
          sources:
            - ubuntu-toolchain-r-test
            - llvm-toolchain-trusty-5.0
          packages:
            - clang++-5.0
            - g++-7
            - cppcheck
```
## C++ code coverage
Login to [codecov.io](https://codecov.io/) with your GitHub credentials and
simply push your coverage files via Travis CI using the generic upload script as
a build rule. Build your C++ using the gcc ```--coverage``` flag (which uses
gcov).

```yaml
script:
  - make
  - bash <(curl -s https://codecov.io/bash)
```

## Deploying to GitHub Pages
In the Travis CI repo settings create a private environment variable "api_key"
containing your GitHub API key. This replaces the ```${api_key}``` below. All
branches are built in Travis CI by default but in this example on the master
branch will be deployed. Deploying for the first time will create a "gh-pages"
branch and set up the username.github.io/repo static web page. I quite like to
use this to generated "live" readmes containing recent data.

```yaml
deploy:
  provider: pages
  github-token: ${api_key}
  skip-cleanup: true
  on:
    branch: master
```

## bash with dot
```yaml
script: make
install: sudo apt install graphviz
```

## Python with requests
```yaml
language: python
python: "3.5"
script: make

# This is implicit
# pip install -r requirements.txt
```

Also need to add ```requirements.txt``` containing a list of libraries:
```bash
requests
```

# Travis CI - triggering builds using the API
You can configure a hourly cron job via the Travis settings but for more
frequent builds set up your own cron job on a Linux web server and use the
Travis API. Note the .org in the API URL, if you use the wrong one (.com) it
will simply report "access denied". There's a different API key for the Pro
account too.

```bash
# Travis
@hourly nice ~/trigger.sh
```

```bash
#!/bin/bash

body=$(cat <<!
{
"request": {
"branch":"master",
"message":"cron $(TZ=BST-1 date)"
}}
!
)

curl -s -X POST \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Travis-API-Version: 3" \
  -H "Authorization: token TOKEN" \
  -d "$body" \
  'https://api.travis-ci.org/repo/USERNAME%2FREPO/requests' >& /dev/null
```

# Clang format on pre-commit
Global configuration to run clang-format on all C++ files as they are pushed to
the server. See [githooks](https://github.com/deanturpin/githooks).

```bash
for file in $(git diff-index --cached --name-only HEAD); do
  if [[ $file == *.cpp || $file == *.h ]]; then
    clang-format -i "$file"
    git add "$file"
  fi
done
```

# Compiler options
```bash
# Standard
--all-warnings
--extra-warnings
-pedantic-errors

# Warnings that are not invoked by all and extra.
-Wshadow
-Wfloat-equal
-Weffc++
-Wdelete-non-virtual-dtor
```

# Uptime monitoring
See [uptime robot](https://stats.uptimerobot.com/V7YEVs8gv).

# References
* https://docs.travis-ci.com/user/languages/cpp/
* https://arne-mertz.de/2017/04/continuous-integration-travis-ci/
