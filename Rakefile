require 'time'

AVAILABLE_REVISIONS = %w[major minor patch].freeze

# task :default => [:deploy]

task :command_exists, [:command] do |_, args|
  abort "#{args.command} doesn't exists" if `command -v #{args.command} > /dev/null 2>&1 && echo $?`.chomp.empty?
end

task :has_bumpversion do
  Rake::Task['command_exists'].invoke('bump-my-version')
end

task :bump, [:revision] => [:has_bumpversion] do |_, args|
  args.with_defaults(revision: 'patch')
  unless AVAILABLE_REVISIONS.include?(args.revision)
    abort "Please provide valid revision: #{AVAILABLE_REVISIONS.join(',')}"
  end

  system "bump-my-version bump #{args.revision}"
end

desc "deploy"
task :deploy, [:revision] do |_, args|
  args.with_defaults(version: 'patch')

  now = Time.now
  current_month = now.strftime("%B")
  current_day = now.strftime("%e")
  current_year = now.strftime("%Y")
  current_time = now.strftime("%H:%M")

  system %{
    sed -i "" 's/\\(Updated at \\)\\(<span>\\)[^<]*\\(<\\/span>, &copy; \\)\\(<strong>\\)[^<]*\\(<\\/strong>\\)/\\1\\2#{current_month}#{current_day} at #{current_time}\\3\\4#{current_year}\\5/' index.html
    sed -i "" 's/\\(Updated at \\)\\(<span>\\)[^<]*\\(<\\/span>, &copy; \\)\\(<strong>\\)[^<]*\\(<\\/strong>\\)/\\1\\2#{current_month}#{current_day} at #{current_time}\\3\\4#{current_year}\\5/' resume/index.html
  }

  unless `git status -s | wc -l`.strip.to_i.zero?
    system %{
      git add .
      git commit -m '[UPDATE] - #{current_month} #{current_day}, #{current_year} at #{current_time}'
      git push
    }
    abort "auto add and commit failed" if $?.exitstatus != 0
  end

  if $?.exitstatus == 0
    Rake::Task[:bump].invoke(args.revision)
  
    system %{
      git push
      git push --tags
      git checkout gh-pages &&
      git pull &&
      git rebase main &&
      git push origin gh-pages &&
      git checkout main
    }
    exit $?.exitstatus
  else
    abort "sed fucked up"
  end
end
