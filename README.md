# [whereisFelix.today?](https://whereisfelix.today)


## Development

```
bundle install
```

```
bundle exec jekyll s
```

Whenever you update `data.erb` you need to run

```
bundle exec ruby graphs/update.rb
```

to speed things up and skip converting images on the fly, you can use

```
SKIP=1 be ruby graphs/update.rb
```

## Backend

The server code is on https://github.com/KrauseFx/whereisfelix.today-backend, which used to also include an Angular-based frontend, but now got replaced by a simple Jekyll application
