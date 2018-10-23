# modille.github.io

[http://modille.github.io/](http://modille.github.io/)

## Setup

```sh
brew install rbenv
rbenv install -f .ruby-version
rbenv init

gem install bundler
bundle install
```

## Serve locally

```sh
bundle exec jekyll serve
```

## Add post

```sh
title="hello world"
touch _posts/$(date "+%Y-%m-%d")-${title// /-}.md
```
