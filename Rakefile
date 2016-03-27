require 'html/proofer'
require 'rspec/core/rake_task'

desc 'Run specs'
RSpec::Core::RakeTask.new do |t|
  t.pattern = 'spec/**/*_spec.rb'
  t.rspec_opts = ['--order', 'rand', '--color']
end

task :test do
  sh 'bundle exec jekyll build'
  Rake::Task['spec'].invoke
  HTML::Proofer.new('./_site', check_html: true,
                               validation: { ignore_script_embeds: true },
                               href_swap: { %r{http://espass.it} => '' }).run
end
