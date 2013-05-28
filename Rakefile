desc "Watch the site and regenerate when it changes"
task :watch do
  Dir.chdir '_source'
  puts "Starting to watch source with Jekyll."
  jekyllPid = Process.spawn("jekyll server --watch")

  trap("INT") {
    [jekyllPid].each { |pid| Process.kill(9, pid) rescue Errno::ESRCH }
    exit 0
  }

  [jekyllPid].each { |pid| Process.wait(pid) }
end
