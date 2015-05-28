# Alaveteli Capistano Test VM

VM to act as target for Capistrano deployments of Alaveteli.

## Usage

    git clone https://github.com/mysociety/alaveteli-capistrano-vm.git
    cd alaveteli-capistrano-vm
    vagrant up

    # On your usual dev machine / VM
    cat > config/deploy.yml <<EOF
    # Site-specific deployment configuration lives in this file
    default: &default
      repository: git://github.com/mysociety/alaveteli.git
      server: 192.168.33.155
      user: alaveteli

    staging:
      <<: *default
      branch: develop
      rails_env: production
      deploy_to: /var/www/alaveteli/alaveteli_staging

    production:
      <<: *default
      branch: master
      rails_env: production
      deploy_to: /var/www/alaveteli/alaveteli
    EOF

    bundle exec cap -S stage=production deploy:setup
    bundle exec cap -S stage=production deploy:update_code
    bundle exec cap -S stage=production deploy

## Gotchas

Somehow `/var/www/alaveteli/alaveteli/current` is created as a directory.
Haven't looked in to it but you need to remove it so that the `current` symlink
can be created.

---

Sometimes thin doesn't start at the end of `cap deploy`:

    ** [out :: 192.168.33.155] Restarting Alaveteli app server:
    *** [err :: 192.168.33.155] /var/www/alaveteli/alaveteli/shared/bundle/ruby/1.9.1/gems/thin-1.5.1/lib/thin/daemonizing.rb:131:in `send_signal': Can't stop process, no PID found in tmp/pids/thin.pid (Thin::PidFileNotFound)

Just manually start it:

    bundle exec cap -S stage=production deploy:start
