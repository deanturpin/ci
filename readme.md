See my [Travis builds](https://travis-ci.org/deanturpin).

# Travis CI config
## Deploying to GitHub Pages
In the Travis CI repo settings create a private environment variable "api_key"
containing your GitHub API key. This replaces the ```${api_key}``` below. All
branches are built in Travis CI by default but in this example on the master
branch will be deployed. Deploying for the first time will create a "gh-pages"
branch and setup the username.github.io/repo static web page.

```yaml
deploy:
  provider: pages
  github-token: ${api_key}
  skip-cleanup: true
  on:
    branch: master
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

## C++
```yaml
dist: trusty
sudo: false
language: c++

addons:
  apt:
    update: true
    sources: ubuntu-toolchain-r-test
    packages: g++-6

script:
  - make
  - bash <(curl -s https://codecov.io/bash)

before_deploy: rm -f .gitignore

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
deploy:
  provider: pages
  github-token: ${api_key}
  skip-cleanup: true
  on:
    branch: master
```

# Travis API - triggering builds
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
