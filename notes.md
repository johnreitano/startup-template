rails new startup --database=postgresql --skip-sprockets --skip-turbolinks --webpack=vue
cd ./startup
bin/rails db:create db:migrate

sed -i '' "s/# system('bin\/yarn')/system('bin\/yarn')/" bin/setup
bin/setup

cat >>config/initializers/content_security_policy.rb <<EOF

Rails.application.config.content_security_policy do |policy|
  if Rails.env.development?
    policy.script_src :self, :https, :unsafe_eval

    policy.connect_src :self, :https, 'http://localhost:3035', 'ws://localhost:3035'
  else
    policy.script_src :self, :https
  end
end
EOF

bin/webpack
bin/rails runner "exit"

sed -i '' "s#<%= javascript_pack_tag 'application' %>#<%= stylesheet_pack_tag 'hello_vue', media: 'all' %> <%= javascript_pack_tag 'hello_vue' %>#" app/views/layouts/application.html.erb

bin/rails generate controller Landing index --no-javascripts --no-stylesheets --no-helper --no-assets --no-fixture

