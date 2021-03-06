#############################################################################
#
# Modified version of jekyllrb Rakefile
# https://github.com/jekyll/jekyll/blob/master/Rakefile
#
#############################################################################

require 'rake'
require 'date'
require 'yaml'

CONFIG = YAML.load(File.read('_config.yml'))
USERNAME = CONFIG["username"] || ENV['GIT_NAME']
REPO = CONFIG["repo"] || "#{USERNAME}.github.io"
POSTS_BRANCH = CONFIG['posts_branch'] || "posts"
PAGES_BRANCH = CONFIG['pages_branch'] || "pages"

# For local serving
HOST = ENV['JEKYLL_HOST'] || "127.0.0.1"

# Determine source and destination branch
# User or organization: source -> master
# Project: master -> gh-pages
# Name of source branch for user/organization defaults to "source"
if REPO == "#{USERNAME}.github.io"
  SOURCE_BRANCH = CONFIG['branch'] || "source"
  DESTINATION_BRANCH = "master"
else
  SOURCE_BRANCH = "master"
  DESTINATION_BRANCH = "gh-pages"
end

def check_source
  puts "Refresh origin"
  sh "git remote rm origin"
  sh "git remote add --fetch origin https://$GIT_NAME:$GH_TOKEN@github.com/#{USERNAME}/#{REPO}.git"
  sh "current=`git rev-parse --abbrev-ref HEAD`; "\
     "for remote in `git branch -r | grep -v /HEAD`; do git checkout --track $remote ; done; "\
     "git checkout $current"

  unless Dir.exist? "_posts"
      sh "git worktree add -f _posts #{POSTS_BRANCH}"
  end

  unless Dir.exist? "_pages"
    sh "git worktree add -f _pages #{PAGES_BRANCH}"
  end

  unless Dir.exist? "#{CONFIG["destination"]}"
      sh "mkdir -p #{CONFIG["destination"]} && git worktree add -f #{CONFIG["destination"]} #{DESTINATION_BRANCH}"
  end

end

namespace :site do
  desc "Generate the site"
  task :build do
    check_source
    sh "bundle exec jekyll build"
  end

  desc "Generate the site and serve locally"
  task :serve do
    check_source
    sh "bundle exec jekyll serve"
  end

  desc "Generate the site, serve locally and watch for changes"
  task :watch do
    check_source
    sh "bundle exec jekyll serve --watch --host #{HOST}"
  end

  desc "Generate the site and push changes to remote origin"
  task :deploy do
    # Detect pull request
    if ENV['TRAVIS_PULL_REQUEST'].to_s.to_i > 0
      puts 'Pull request detected. Not proceeding with deploy.'
      exit
    end

    # Configure git if this is run in Travis CI
    if ENV["TRAVIS"]
      sh "git config --global user.name $GIT_NAME"
      sh "git config --global user.email $GIT_EMAIL"
      sh "git config --global push.default simple"
    end

    # Check source
    check_source

    sh "git checkout #{SOURCE_BRANCH}"

    # Generate the site
    sh "bundle exec jekyll build"

    # Commit and push to github
    sha = `git log`.match(/[a-z0-9]{40}/)[0]
    Dir.chdir(CONFIG["destination"]) do
      # check if there is anything to add and commit, and pushes it
      sh "if [ -n '$(git status)' ]; then
            git add --all .;
            git commit -m 'Updating to #{USERNAME}/#{REPO}@#{sha}.';
            git push --quiet origin #{DESTINATION_BRANCH};
         fi"
      puts "Pushed updated branch #{DESTINATION_BRANCH} to GitHub Pages"
    end
  end
end