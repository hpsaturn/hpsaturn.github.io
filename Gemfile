source "https://rubygems.org"

# Ruby toolchain version. Bundler records the running version as RUBY VERSION
# in Gemfile.lock (Dependabot reads it to decide what is resolvable), and the
# exact development/CI version is pinned in .ruby-version (3.1.7). The
# requirement stays on the 3.1 line so that Debian 12's stock Ruby 3.1.2 can
# also run local builds; bumping to 3.2+ must be a deliberate decision because
# several gems (e.g. nokogiri >= 1.19) gate themselves on it.
ruby "~> 3.1"

gem "jekyll", "~> 4.4"
gem "jekyll-sitemap"
gem "jekyll-gist"
gem "jekyll-mentions"
gem "jekyll-feed"
gem "kramdown-parser-gfm"

gem "webrick", "~> 1.9"

# NOTE: jekyll-commonmark-ghpages was removed on this branch.
# It is a GitHub-Pages-only helper that pins jekyll (< 4.0), and the
# site uses kramdown as its markdown engine (see _config.yml), so it is
# not needed. If you want CommonMark output, use jekyll-commonmark instead.
