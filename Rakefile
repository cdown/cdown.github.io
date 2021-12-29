require 'date'

task :default => [:deploy]

cc_hdr = "Cache-Control: public, max-age=%d"

# 1 hour for anything unspecified
generic_cc = cc_hdr % (60 * 60)
generic_excludes = ""

# 24 hours for static content
static_cc = cc_hdr % (24 * 60 * 60)
static_includes = ""
["css/*", "images/*", "fonts/*", "photos/*", "talks/*"].each do |path|
  static_includes += " --include '#{path}' "
  generic_excludes += " --exclude '#{path}' "
end

# 5 minutes for posts written in the last 7 days, as updates may be more
# frequent
this_week_post_cc = cc_hdr % (5 * 60)
this_week_post_includes = ""
(Date.today-6..Date.today).map do |date|
  date_glob = date.strftime("%Y/%m/%d") + "/*"
  this_week_post_includes += " --include '#{date_glob}' "
  generic_excludes += " --exclude '#{date_glob}' "
end

# HACK: s3cmd modify returns 1 if there was nothing matched, what? This will
# get overwritten by the next step, but ugh...
this_week_post_includes += " --include 404.html "

# Cache 404s for 2 minutes, since if something is broken we want to fix it
# fast, but a mass storm to S3 backend should still be avoided
enoent_cc = cc_hdr % (2 * 60)
generic_excludes += " --exclude 404.html "

task :sync => :build do
  sh "s3cmd sync --no-mime-magic --no-preserve --cf-invalidate --delete-removed --verbose --add-header='#{generic_cc}' _deploy/ s3://chrisdown.name"
end

task :deploy => :sync do
  sh "s3cmd modify --recursive #{generic_excludes} --add-header='#{generic_cc}' s3://chrisdown.name"
  sh "s3cmd modify --recursive --exclude '*' #{this_week_post_includes} --add-header='#{this_week_post_cc}' s3://chrisdown.name"
  sh "s3cmd modify --recursive --exclude '*' #{static_includes} --add-header='#{static_cc}' s3://chrisdown.name"
  sh "s3cmd modify --add-header='#{enoent_cc}' s3://chrisdown.name/404.html"
end

task :build do
  sh "JEKYLL_ENV=production jekyll build"
end
