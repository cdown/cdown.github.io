task :default => [:deploy]

task :deploy => :build do
  sh "rsync -av --delete deploy/ guthrie:/srv/http/chrisdown.name/"
end

task :build do
  sh "jekyll build"
end
