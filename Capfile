# Load DSL and Setup Up Stages
require 'capistrano/setup'
require 'capistrano/simpledeploy'

set :application,   "bolt-try"
set :deploy_to,     "/var/www/sites/try.bolt.cm"
set :repo_url,      "git@github.com:bolt/try-bolt.git"
set :stage,         "production" ### Default stage

set :composer_roles, :web
set :composer_install_flags, '--no-dev --prefer-dist --no-interaction --quiet'

task :production do
    set :branch,        "master"
    server 'try.bolt.cm', user: 'ross', roles: %w{web}
end

namespace :deploy do

    desc "Updates the code on the remote container"
    task :start do
        on roles :web do |host|
            begin execute "pkill -f 'try-bolt-start.sh'" rescue nil end
            begin execute "pkill -f 'demo-runner'" rescue nil end
            execute "cd #{fetch(:deploy_to)} && chmod +x ./try-bolt-start.sh"
            execute "cd #{fetch(:deploy_to)} && ((nohup ./try-bolt-start.sh &>/dev/null) &)"
        end
    end

    task :secrets do
        on roles :web do
            upload! "config/env.php", "#{fetch(:deploy_to)}/config/env.php"
        end
    end

    task :symlink do
        on roles :all do
            execute "ln -fs production.php #{fetch(:deploy_to)}/config/config.php"
        end
    end

end

namespace :composer do
    task :symlink do
        on roles :web do
           execute "cd #{fetch(:deploy_to)}; ln -fs ~/composer composer.phar"
        end
    end
end

before "deploy", "composer:install_executable"
before "deploy", "composer:symlink"
after "deploy", "deploy:symlink"
after "deploy", "deploy:secrets"
after "deploy", "deploy:start"

