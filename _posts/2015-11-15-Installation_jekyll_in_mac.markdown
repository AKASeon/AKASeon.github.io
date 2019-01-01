---
layout: post
title:  "Installation Jekyll in mac"
date:   2015-11-15
categories: Jekyll
---
# jekyll 설치

## 요구 사항
jekyll 홈페이지 에서 요구 하는 사항은 아래와 같다.
설치 하려는 jekyll 에 맞춰서 요구 사항을 맞춰야 하겠다.

- Ruby (including development headers, v1.9.3 or above for Jekyll 2 and v2 or above for Jekyll 3)
- RubyGemsLinux, Unix, or Mac OS X
- NodeJS, or another JavaScript runtime (Jekyll 2 and earlier, for CoffeeScript support).
- Python 2.7 (for Jekyll 2 and earlier)

## ruby 설치
ruby 를 설치 하는것은 여러 가지 방법이 있다.

### brew 로 설치
제일 쉬운것이 아마 brew 로 설치 하는것이다.  
언어 같은 경우는 버젼 별 관리등의 이유로 brew 를 사용하는것은 선호하지 않지만.. 많이 쉽다.  
```
brew install ruby
```

### 스크립트로 설치
installer 스크립트로 설치 할수 있다.
1. ruby-build
2. ruby-install
3. RubyInstaller
4. 등등..

아래 링크에 보면 보다 더 자세히 설명 되어 있다.
https://www.ruby-lang.org/en/documentation/installation/#ruby-build

#### ruby-build
brew 로 ruby-build 를 설치 하거나.  
git 에서 ruby-build 를 내려 받는다.

```
ruby-build [version] [설치될 directory]
```
-v 를 추가적으로 추면 설치 하는 과정을 볼수 있다.  

설치될 directory 를 지정 하였으면 환경 변수를 잡아 줘야 한다.

```
# Ruby
export RUBY_HOME=[설치된 directory]
export PATH=$RUBY_HOME/bin:$PATH
```

### ruby 설치 여부 확인
version 과 설치된 디렉토리를 확인한다.

```
DoO@DooSeonui-MacBook-Air memo/$ ruby -v
ruby 2.2.0p0 (2014-12-25 revision 49005) [x86_64-darwin15]
DoO@DooSeonui-MacBook-Air memo/$ which ruby
/Users/DoO/opt/ruby/2.2.0/bin/ruby
```

## jekyll 설치

```
gem install jekyll
```

### jekyll 확인

버젼 확인

```
jekyll -v

```

설치된 위치 확인

```
which jekll
```

### 추가 plugin 설치
#### redcarpet
markdown 문서를 html 로 변환해주는 패키지이다.
jekyll 기본으로 설정 되어 있는 markdown 으로 변환해주는 것도 있지만 markdown 문법을 완벽하게 지원하지 않는다.  
그렇기 때문에 redcarpet 을 설치하여 redcarpet  을 사용하자.

```
gem install redcarpet
```
#### Pygments
syntax highligting 도구

```
pip install Pygments
```

#### 문제
jekyll serve 로 markdwon 문서를 읽을 시 아래와 같은 문제가 있을수 있다.

```
DoO@DooSeonui-MacBook-Air memo/$ jekyll serve
Configuration file: /Users/DoO/Study/memo/_config.yml
       Deprecation: The 'pygments' configuration option has been renamed to 'highlighter'. Please update your config file accordingly. The allowed values are 'rouge', 'pygments' or null.
            Source: /Users/DoO/Study/memo
       Destination: /Users/DoO/Study/memo/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
  Dependency Error: Yikes! It looks like you don't have pygments or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. The full error message from Ruby is: 'cannot load such file -- pygments' If you run into trouble, you can find helpful resources at http://jekyllrb.com/help/!
  Liquid Exception: pygments in /Users/DoO/Study/memo/_posts/2015-10-20-welcome-to-jekyll.markdown
             ERROR: YOUR SITE COULD NOT BE BUILT:
                    ------------------------------------
                    pygments
```

이것은 ruby 에 필요한 패키지가 없어서 그런 것이다.  
gem 을 이용하여 추가적으로 패키지를 설치 하자.

```
gem install pygments.rb
```


# 참조
https://www.ruby-lang.org/
https://jekyllrb.com
https://github.com/sstephenson/ruby-build#readme
http://xthy.github.io/blog/2013/12/23/Syntax-Highlight/
