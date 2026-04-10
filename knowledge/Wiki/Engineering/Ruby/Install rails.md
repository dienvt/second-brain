---
title: "install gpg if missing"
date: 2026-01-14
tags:
  - engineering
  - ruby
---

### install ruby
```shell
# install gpg if missing
brew install gpg


```

### Then I tried to install openssl

`rvm pkg install openssl`

```ruby
$ rvm pkg install openssl
$ rvm remove 2.4
$ rvm install 2.4 --with-openssl-dir=$HOME/.rvm/usr
$ gem install bundler
```

```
RUBY_CONFIGURE_OPTS="--with-openssl-dir=/opt/homebrew/opt/openssl@1.1" rbenv install 3.0.6
```

```
export GEM_HOME="$HOME/.gem"
```

https://stackoverflow.com/a/54873916



worked
```ruby
RUBY_CFLAGS="-Wno-error=implicit-function-declaration" RUBY_CONFIGURE_OPTS="--with-openssl-dir=`brew --prefix openssl` --with-readline-dir=`brew --prefix readline`" sudo arch -x86_64 rbenv install --verbose 3.2.3
```