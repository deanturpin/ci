# Travis config
## Deploying to GitHub Pages
In the Travis repo settings create a private environment variable "api_key" containing your GitHub API key. This replaces the ```${api_key}``` below. All branches are built in Travis by default but in this example on the master branch will be deployed.

```yaml
deploy:
  provider: pages
  github-token: ${api_key}
  skip-cleanup: true
  on:
    branch: master
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
language: bash
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
