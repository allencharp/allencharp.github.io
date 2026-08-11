source "https://rubygems.org"

# jekyll 3.9.x allows kramdown 2.x, which fixes CVE-2020-14001 / CVE-2021-28834
gem "jekyll", "~> 3.9.0"

# Vendored hacker theme; seo-tag is used by the theme layout
gem "jekyll-seo-tag"

# jekyll 3.9 defaults to kramdown input: GFM
gem "kramdown-parser-gfm"

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
