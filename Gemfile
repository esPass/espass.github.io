source 'https://rubygems.org'

require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)

gem 'github-pages'

group :development do
  gem 'colored'
  gem 'terminal-table'
  gem 'fuzzy_match'
end

group :test do
  gem 'html-proofer', '~> 2.6'
  gem 'rake'
  gem 'rspec'
  gem 'nokogiri'
  gem 'rubocop'
end
