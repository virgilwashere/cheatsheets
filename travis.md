---
title: .travis.yml
category: Devops
layout: 2017/sheet   # 'default' | '2017/sheet'

# Optional:
updated: 2019-05-31       # To show in the updated list
ads: false                # Add this to disable ads
weight: -5                # lower number = higher in related posts list
deprecated: false         # Don't show in related posts
prism_languages: [yaml]    # Extra syntax highlighting
intro: |
  [Travis CI] is continuous integration for your GitHub projects, and automation for testing building and deploying.
tags:
  - Featured
---

## `.travis.yml`

A `.travis.yml` configuration file

[Travis CI] supports many [programming languages](#languages).

* [customizing your build][custom]
* [build stages][stages]
* [installing dependencies][dependencies]
* [setting up databases][database]
* [security best practices][security]

Travis CI isn't just for running [tests](#tests):

* [deploy] to [GitHub pages][gh-pages]
* run apps on [Heroku]
* upload [RubyGems]
* send [notifications]

See: <https://docs.travis-ci.com/>

## Build stages

The complete job lifecycle, including three optional deployment phases and after checking out the git repository and changing to the repository directory, is:

1. OPTIONAL Install [`apt addons`](/user/installing-dependencies/#installing-packages-with-the-apt-addon)
2. OPTIONAL Install [`cache components`](/user/caching)
3. `before_install`
4. `install`
5. `before_script`
6. `script`
7. OPTIONAL `before_cache` (for cleaning up cache)
8. `after_success` or `after_failure` (`$TRAVIS_TEST_RESULT` available)
9. OPTIONAL `before_deploy`
10. OPTIONAL `deploy`
1. OPTIONAL `after_deploy`
2. `after_script`

A *build* can be composed of many jobs.

### stage names

<details><summary>named build stages</summary>

```yaml
jobs:
  include:
    - stage: test
      script: ./test 1
    - # stage name not required, will continue to use `test`
      script: ./test 2
    - stage: deploy
      script: skip     # usually you do not want to rerun any tests
      deploy: &heroku
        provider: heroku
        # ...
```

</details>

## Languages

### Node

```yaml
language: node_js
node_js:
  - "7"
  - "6"
  - "5"
  - "4"

before_install:
  - curl -o- -L https://yarnpkg.com/install.sh | bash -s -- --version version-number
  - export PATH="$HOME/.yarn/bin:$PATH"

cache:
  yarn: true

script: echo "Running tests against $(node -v) ..."

jobs:
  include:
    - stage: npm release
      node_js: "7"
      script: echo "Deploying to npm ..."
      deploy:
        provider: npm
        api_key: $NPM_API_KEY
        on: deploy-npm-release
```

* Provides: 0.10, 0.8, 0.6, 0.11 (latest dev)

default:

* `install:` `npm install`
* `test:` `npm test`

### Ruby

```yaml
language: ruby
rvm:
- 2.3.0
- 2.2.0
- 2.0.0
- 1.9.3

# bundler to support rails
before_install:
  - gem uninstall -v '>= 2' -i $(rvm gemdir)@global -ax bundler || true
  - gem install bundler -v '< 2'
# RubyGems
  - gem update --system 2.1.11
  - gem --version

bundler_args: --without production debug --retry 5
gemfile: gemfiles/Gemfile.ci
```

<details><summary>gemfiles/Gemfile.ci</summary>

```ruby
group :production do
  gem 'unicorn'
  gem 'newrelic_rpm'
end
group :debug do
  gem 'debugger'
  gem 'debugger-linecache'
  gem 'rblineprof'
end
```

</details>

* Defaults install to `bundle install`
* Defaults test to `rake`

### Bash

Setting `language` to **`bash`**, `sh` or `shell` is equivalent to **`minimal`**.

```yaml
language: bash
before_script:
  - bash --version
```

* No default install
* No default script

### Environment vars

A repository's `.travis.yml` file can have "encrypted values", such as environment variables, notification settings, and deploy api keys.

To encrypt an environment variable, try this:

```console
$ gem install travis
$ travis login --pro
$ travis encrypt --com SOMEVAR="secretvalue" -add env.global
secure: ".... encrypted data ...."
```

```yaml
env:
  global:
    - secure: mcUCykGm4bUZ3CaW6AxrIMFzuAYjA98VIz6YmYTmM0/8sp/B/54JtQS/j0ehCD6B5BwyW6diVcaQA2c7bovI23GyeTT+TgfkuKRkzDcoY51ZsMDdsflJ94zV7TEIS31eCeq42IBYdHZeVZp/L7EXOzFjVmvYhboJiwnsPybpCfpIH369fjYKuVmutccD890nP8Bzg8iegssVldgsqDagkuLy0wObAVH0FKnqiIPtFoMf3mDeVmK2AkF1Xri1edsPl4wDIu1Ko3RCRgfr6NxzuNSh6f4Z6zmJLB4ONkpb3fAa9Lt+VjJjdSjCBT1OGhJdP7NlO5vSnS5TCYvgFqNSXqqJx9BNzZ9/esszP7DJBe1yq1aNwAvJ7DlSzh5rvLyXR4VWHXRIR3hOWDTRwCsJQJctCLpbDAFJupuZDcvqvPNj8dY5MSCu6NroXMMFmxJHIt3Hdzr+hV9RNJkQRR4K5bR+ewbJ/6h9rjX6Ot6kIsjJkmEwx1jllxi4+gSRtNQ/O4NCi3fvHmpG2pCr7Jz0+eNL2d9wm4ZxX1s18ZSAZ5XcVJdx8zL4vjSnwAQoFXzmx0LcpK6knEgw/hsTFovSpe5p3oLcERfSd7GmPm84Qr8U4YFKXpeQlb9k5BK9MaQVqI4LyaM2h4Xx+wc0QlEQlUOfwD4B2XrAYXFIq1PAEic=
```

See <https://docs.travis-ci.com/user/environment-variables/> for more.

### git controls

#### clone depth and submodules

```yaml
git:
  depth: 3
  depth: false
  submodules: false
```

#### Branches

```yaml
# blocklist
branches:
  except:
  - legacy
  - experimental

# safelist
branches:
  only:
  - master
  - stable

# To build all branches:
branches:
  only:
  - gh-pages
  - /.*/
```

### Package dependencies

### [apt] and [homebrew] packages

[apt]: <https://docs.travis-ci.com/user/installing-dependencies#installing-packages-with-the-apt-addon/>
[homebrew]: <https://docs.travis-ci.com/user/installing-dependencies/#installing-packages-on-macos>

Via `addons` or `before_install` script.

#### addons

```yaml
addons:
  apt:
    update: true
    sources:
    - ubuntu-toolchain-r-test
    packages:
    - gcc-4.8
    - openssl
    # System: Required language pack isn’t installed
    - language-pack-en
    - language-pack-de
  homebrew:
    update: true
    packages:
    - openssl
    - coreutils
```

#### scripted packages

```yaml
before_install:
- if [ "$TRAVIS_OS_NAME" = "linux" ]; then sudo apt-get -qq -y update         ; fi
- if [ "$TRAVIS_OS_NAME" = "linux" ]; then sudo apt-get -qq -y install gcc-4.8; fi
```

### Custom test command

```yaml
script: make test
before_script: make pretest
after_script:  make clean

before_script:
  - make pretest1
  - make pretest2
```

### Notifications

For Slack, set up a [new Travis CI integration](https://my.slack.com/services/new/travis).

```yaml
notifications:
  slack:
    rooms:
      - <account>:<token>#development
      - <account>:<token>#general
    on_success: change # <always|never|change> default: always
    on_failure: always # <always|never|change> default: always

  email:
    - dropbox+travis@ricostacruz.com

  email:
    recipients:
      - dropbox+travis@ricostacruz.com
    on_success: change # <always|never|change> default: change
    on_failure: always # <always|never|change> default: always

  irc: "chat.freenode.net#travis"
```

<details><summary>Slack notification with encrypted credentials</summary>

```bash
travis encrypt "<account>:<token>#development" --add notifications.slack.rooms
travis encrypt "<account>:<token>#channel" --add notifications.slack.rooms
```

This is how a setup with encrypted credentials would look:

```yaml
notifications:
  slack:
    rooms:
      - secure: "sdfusdhfsdofguhdfgubdsifgudfbgs3453durghssecurestringidsuag34522irueg="
      - secure: "sdfusdhfsdofguhdfgubdsifgudfbgs3453durghssecurestringidsuag34522irueg="
    on_success: change
```

</details>

### References

* <https://docs.travis-ci.com/user/customizing-the-build/>
* <https://docs.travis-ci.com/user/languages/javascript-with-nodejs/>
* <https://docs.travis-ci.com/user/languages/ruby/>
* <https://docs.travis-ci.com/user/installing-dependencies>

[Travis CI]: <https://travis-ci.com>

[custom]: <https://docs.travis-ci.com/user/customizing-the-build/>
[matrix]: <https://docs.travis-ci.com/user/customizing-the-build/#build-matrix>
[stages]: <https://docs.travis-ci.com/user/build-stages/>
[dependencies]: <https://docs.travis-ci.com/user/installing-dependencies/>
[database]: <https://docs.travis-ci.com/user/database-setup/>
[security]: <https://docs.travis-ci.com/user/best-practices-security/>

[deploy]: <https://docs.travis-ci.com/user/deployment/>
[gh-pages]: <https://docs.travis-ci.com/user/deployment/pages/>
[Heroku]: <https://docs.travis-ci.com/user/deployment/heroku/>
[RubyGems]: <https://docs.travis-ci.com/user/deployment/rubygems/>
[notifications]: <https://docs.travis-ci.com/user/notifications/>