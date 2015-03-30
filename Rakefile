require 'rake'

# load our config
load '_rake_config.rb'

task :default => :help

desc "help information"
task :help do
  puts "Tasks:"
  puts "======"
  system "rake --tasks"
end

desc "create a new post"
task :post, :title, :date do |t,args|
  if args.title == nil or
    (args.date and args.date.match(/^[0-9]{4}-[0-9]{2}-[0-9]{2}$/) == nil) then
    exit 1
  end

  post_title = args.title
  post_date = args.date || Time.new.strftime("%Y-%m-%d %H:%M:%S")


  # remove the time from post_date (the filename does not support it)
  filename = post_date[0..9] + "-" + post_title.gsub(' ', '_') + ".markdown"

  # generate a unique filename appending a number
  i = 1
  while File.exists?($post_dir + filename) do
    filename = post_date[0..9] + "-" + post_title.gsub(' ', '_') + "-" +
      i.to_s + ".markdown"
    i += 1
  end

  # setup our post and start editing it
  File.open($post_dir + filename, 'w') do |f|
    f.puts "---"
    f.puts "title: \"#{post_title}\""
    f.puts "layout: post"
    f.puts "date: #{post_date}"
    f.puts "comments: true"
    f.puts "tags: [tag1,tag2]"
    f.puts "share: true"
    f.puts "---"
    f.puts args.content if args.content != nil
  end

  puts "Post created under \"#{$post_dir}#{filename}\""

  sh "vi \"#{$post_dir}#{filename}\"" if args.content == nil
end

task :serve do
  sh 'jekyll serve -w'
end
