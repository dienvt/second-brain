
## Check version homebrew
We need to check homebrew currently is using for M1 or Intel chip
Switch to M1 chip
```
export PATH=/opt/homebrew/bin:/opt/homebrew/sbin:$PATH
```


## Install ruby using rbenv
```
brew update

brew upgrade rbenv

brew install ruby-build
```

**Note**: If `ruby -v` still the old version. Execute this

```
export PATH="$HOME/.rbenv/shims:$PATH"
eval "$(rbenv init -)"
```


## Stuckes
### mysql2
```
brew install mysql

gem install mysql2 -- --with-ldflags=-L$(brew --prefix zstd)/libiit
```

### rmagick

```
 brew install imagemagick
```