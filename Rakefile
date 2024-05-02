require 'time'

AVAILABLE_REVISIONS = %w[major minor patch].freeze

# task :default => [:deploy]

task :command_exists, [:command] do |_, args|
  abort "#{args.command} doesn't exists" if `command -v #{args.command} > /dev/null 2>&1 && echo $?`.chomp.empty?
end

task :has_bumpversion do
  Rake::Task['command_exists'].invoke('bumpversion')
end

task :bump, [:revision] => [:has_bumpversion] do |_, args|
  args.with_defaults(revision: 'patch')
  unless AVAILABLE_REVISIONS.include?(args.revision)
    abort "Please provide valid revision: #{AVAILABLE_REVISIONS.join(',')}"
  end

  system "bumpversion #{args.revision}"
  exit $?.exitstatus unless ENV['RAKE_CONTINUE']
end

desc "deploy"
task :deploy, [:revision] do |_, args|
  args.with_defaults(version: 'patch')

  now = Time.now
  current_month = now.strftime("%B")
  current_day = now.strftime("%e")
  current_year = now.strftime("%Y")

  system %{
    sed -i "" 's/\\(Updated at \\)\\(<span>\\)[^<]*\\(<\\/span>, &copy; \\)\\(<strong>\\)[^<]*\\(<\\/strong>\\)/\\1\\2#{current_month}#{current_day}\\3\\4#{current_year}\\5/' index.html
    sed -i "" 's/\\(Updated at \\)\\(<span>\\)[^<]*\\(<\\/span>, &copy; \\)\\(<strong>\\)[^<]*\\(<\\/strong>\\)/\\1\\2#{current_month}#{current_day}\\3\\4#{current_year}\\5/' resume/index.html
  }

  if $?.exitstatus == 0
    Rake::Task[:bump].invoke(args.version)
  
    system %{
      git checkout gh-pages &&
      git rebase main &&
      git push origin gh-pages &&
      git checkout main
    }
    exit $?.exitstatus
  else
    abort "sed fucked up"
  end
end