# A sample Gemfile
source "https://rubygems.org"


# Added	by me jfraj to allowed github-page to host the blog
# as instructed by http://jekyllrb.com/docs/github-pages/
require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)
gem 'github-pages', versions['github-pages']
gem 'jekyll'