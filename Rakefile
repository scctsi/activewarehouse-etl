require 'bundler'
Bundler::GemHelper.install_tasks
require 'rake'
require 'rake/testtask'
require 'yaml'

def system!(cmd)
  puts cmd
  raise "Command failed!" unless system(cmd)
end

require 'tasks/standalone_migrations'

# experimental tasks to reproduce the Travis behaviour locally
namespace :ci do

  desc "For current RVM, run the tests for one db and one gemfile"
  task :run_one, :db, :gemfile do |t, args|
    Bundler.with_clean_env do
      ENV['BUNDLE_GEMFILE'] = File.expand_path(args[:gemfile] || (File.dirname(__FILE__) + '/test/config/gemfiles/Gemfile.rails-3.2.x'))
      ENV['DB'] = args[:db] || 'mysql2'

      ## Rails 3.2.x
      # system! "bundle install"
      # system! "bundle exec rake db:create"
      # system! "bundle exec rake db:create RAILS_ENV=etl_execution"
      # system! "bundle exec rake db:schema:load"
      # system! "bundle exec rake"

      ## Rails 4.0.x
      # db_config = YAML.load(ERB.new(File.read(File.dirname(__FILE__) + '/test/config/database.yml')).result)
      system! "bundle install"

      # ActiveRecord::Tasks::DatabaseTasks.database_configuration = YAML.load(ERB.new(File.read(File.dirname(__FILE__) + '/test/config/database.yml')).result)
      # ActiveRecord::Base.configurations = YAML.load(ERB.new(File.read(File.dirname(__FILE__) + '/test/config/database.yml')).result)

      system! %Q{bundle exec ruby -e "require %(yaml); require %(active_record); require %(erb); db_config = YAML.load(ERB.new(File.read(File.dirname(__FILE__) + '/test/config/database.yml')).result); ActiveRecord::Tasks::DatabaseTasks.create_current(db_config['postgresql']); ActiveRecord::Tasks::DatabaseTasks.create_current(db_config['operational_database']); ActiveRecord::Tasks::DatabaseTasks.create_current(db_config['data_warehouse']); ActiveRecord::Tasks::DatabaseTasks.create_current(db_config['etl_execution']); ActiveRecord::Tasks::DatabaseTasks.db_dir = 'db'; file = File.join(ActiveRecord::Tasks::DatabaseTasks.db_dir, 'schema.rb'); file_val = File.exist?(file) ? load(file) : nil"}

      

# ActiveRecord::Tasks::DatabaseTasks.migrations_paths ||= ['db/migrate']; ActiveRecord::Migrator.migrations_paths = ActiveRecord::Tasks::DatabaseTasks.migrations_paths

      # ActiveRecord::Tasks::DatabaseTasks.db_dir = 'db'
      # file = File.join(ActiveRecord::Tasks::DatabaseTasks.db_dir, 'schema.rb')
      # if File.exist?(file)
      #   load(file)
      # else
      #   abort %{#{file} doesn't exist yet. Run `rake db:migrate` to create it, then try again. If you do not intend to use a database, you should instead alter #{Rails.root}/config/application.rb to limit the frameworks that will be loaded.}
        # end
      system! "bundle exec rake"
    end
  end

  desc "For current RVM, run the tests for all the combination in travis configuration"
  task :run_matrix do
    require 'cartesian'
    config = YAML.load_file('.travis.yml')
    config['env'].cartesian(config['gemfile']).each do |*x|
      env, gemfile = *x.flatten
      db = env.gsub('DB=', '')
      print [db, gemfile].inspect.ljust(40) + ": "
      cmd = "rake \"ci:run_one[#{db},#{gemfile}]\""
      result = system "#{cmd} > /dev/null 2>&1"
      result = result ? "OK" : "FAILED! - re-run with: #{cmd}"
      puts result
    end
  end

end

task :default => :test

desc 'Test the ETL application.'
Rake::TestTask.new(:test) do |t|
  t.libs << 'lib' << '.'
  t.pattern = 'test/**/*_test.rb'
  t.verbose = true
  # TODO: reset the database
end
