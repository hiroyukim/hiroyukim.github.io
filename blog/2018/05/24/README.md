## Personal asciidoctor setup

### Install

#### create dir

```shell
mkdir -p $HOME/local/asciidoctor
cd $HOME/local/asciidoctor
```

#### Gemfile

```ruby
# frozen_string_literal: true
source "https://rubygems.org"

# gem "rails"
gem 'asciidoctor'
gem 'asciidoctor-diagram'
gem 'pygments.rb'
```

#### bundle install

```sh
bundle install --path vendor/bundler
```

### command

#### open_asciidoctor.sh

```shell
#!/bin/sh
cd $HOME/local/asciidoctor
bundle exec asciidoctor -r asciidoctor-diagram -o /tmp/hoge.html $1 && open /tmp/hoge.html
```

```shell
chmod +x open_asciidoctor.sh
./open_asciidoctor.sh /path/to/your.asciidoc
```

### See

+ [https://asciidoctor.org/]()
+ [https://asciidoctor.org/docs/user-manual/]()
+ [https://takumon.github.io/asciidoc-syntax-quick-reference-japanese-translation/]()


### Docker

+ [https://github.com/liquidz/docker-asciidoctor-jp]()
+ [https://hub.docker.com/r/asciidoctor/docker-asciidoctor/]()
