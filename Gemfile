source "https://rubygems.org"

gem "jekyll", "~> 4.3"
gem "just-the-docs", "~> 0.10.0"

group :jekyll_plugins do
  gem "jekyll-seo-tag"
  gem "jekyll-sitemap"
  gem "jekyll-include-cache"
end

# Windows and JRuby does not include zoneinfo files
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]

# Lock http_parser.rb for JRuby builds
gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]
