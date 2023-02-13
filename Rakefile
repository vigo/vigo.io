desc "deploy"
task :deploy do
  system %{
    git checkout gh-pages &&
    git rebase main &&
    git push origin gh-pages &&
    git checkout main
  }
end