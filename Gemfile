# frozen_string_literal: true

source "https://rubygems.org"

# Standalone Jekyll 4.x (no longer using the github-pages gem / Jekyll 3.x).
# The site is built and deployed via GitHub Actions, so we are not tied to the
# GitHub Pages Jekyll version any more.
gem "jekyll", "~> 4.3"

# Plugins previously provided via github-pages. jekyll-remote-theme is
# intentionally dropped: the beautiful-jekyll theme is vendored locally in
# _layouts / _includes / assets, so no remote fetch is needed.
group :jekyll_plugins do
  gem "jekyll-feed"
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-redirect-from"
end

# Needed for local development on Ruby 3.0+
gem "webrick", "~> 1.8"
