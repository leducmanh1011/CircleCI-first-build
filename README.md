# CircleCI your first green build

A project build by Ruby on Rails with tool CircleCI

## What is CI/CD?

The CI/CD pipeline is one of the best practices for devops teams to implement, for delivering code changes more frequently and reliably

Continuous integration (CI) and continuous delivery (CD) embody a culture, set of operating principles, and collection of practices that enable application development teams to deliver code changes more frequently and reliably. The implementation is also known as the CI/CD pipeline.

CI/CD is one of the best practices for devops teams to implement. It is also an agile methodology best practice, as it enables software development teams to focus on meeting business requirements, code quality, and security because deployment steps are automated.

## CircleCI documents
This document provides a step-by-step tutorial for getting your first successful (green) build on CircleCI.
[Getting started](https://circleci.com/docs/2.0/getting-started/#section=getting-started)

- Creating a repository
- Setting up CircleCI
- Digging into your first pipeline
- Collaborating with teammates

## File config
CircleCI require a file config.yml with path:

```ruby
mkdir .circleci
touch .circleci/config.yml
```

Example config.yml

```
version: 2.1

executors:
  default:
    working_directory: ~/CircleCI-first-build
    docker:
      - image: circleci/ruby:2.6.2
        environment:
          BUNDLE_JOBS: 3
          BUNDLE_PATH: vendor/bundle
          BUNDLE_RETRY: 3
          BUNDLER_VERSION: 1.17.2
          RAILS_ENV: test
          DB_USERNAME: root
          DB_PASSWORD: ""
          DB_HOST: 127.0.0.1
      - image: circleci/mysql:5.7
        command: [--default-authentication-plugin=mysql_native_password]
        environment:
          MYSQL_ALLOW_EMPTY_PASSWORD: 'true'
          MYSQL_ROOT_HOST: '%'

commands:
  configure_bundler:
    description: Configure bundler
    steps:
      - run:
          name: Configure bundler
          command: |
            echo 'export BUNDLER_VERSION=$(cat Gemfile.lock | tail -1 | tr -d " ")' >> $BASH_ENV
            source $BASH_ENV
            gem install bundler

jobs:
  build:
    executor: default
    steps:
      - checkout
      - restore_cache:
          keys:
            - CircleCI-first-build-{{ .Branch }}-{{ checksum "Gemfile.lock" }}
            - CircleCI-first-build-
      - configure_bundler
      - run:
          name: Install bundle
          command: bundle install
      - run:
          name: Wait for DB
          command: dockerize -wait tcp://127.0.0.1:3306 -timeout 1m
      - run:
          name: Setup DB
          command: bundle exec rails db:create db:schema:load --trace
      - save_cache:
          key: CircleCI-first-build-{{ .Branch }}-{{ checksum "Gemfile.lock" }}
          paths:
            - vendor/bundle
      - persist_to_workspace:
          root: ~/
          paths:
            - ./CircleCI-first-build

workflows:
  version: 2
  integration:
    jobs:
      - build
```

## Demo
![image](https://user-images.githubusercontent.com/46446038/110265768-eb351280-7fee-11eb-9b54-a990f1b6a302.png)
