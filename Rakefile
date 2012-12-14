namespace :watch do
  desc "watch sass"
  task :sass do
    system('bundle exec sass --watch _sass/main.scss:assets/stylesheets/main.css')
  end

  desc 'watch jekyll'
  task :jekyll do
    system('bundle exec jekyll --auto --server')
  end
end
