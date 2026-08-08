source "https://rubygems.org"

gem "jekyll", "~> 3.8.3"

# Vendored hacker theme; seo-tag is used by the theme layout
gem "jekyll-seo-tag"

# Pin old ffi so dependency resolution works on system ruby 2.6
# (bundler 1.17 picks up ffi-1.17.x which needs ruby >= 3.0)
gem "ffi", "~> 1.15.5"

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.6"
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem "tzinfo-data", platforms: [:mingw, :mswin, :x64_mingw, :jruby]

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.0" if Gem.win_platform?
