# OpenShift Rails 3.2 Example Application 

```bash
oldname=railsapp
newname=rails_app

if [ ! -d "$newname" ]; then
  # Create OpenShift app
  rhc app create -a $oldname -t ruby-1.9
  rhc app cartridge add -a $oldname -c mysql-5.1

  # Rename it to an underscore name so Rails makes a nice camel case app name
  mv $oldname $newname
fi

# Move into Rails directory
pushd $newname > /dev/null

# Make sure to reset to the original Ruby cart head
git reset --hard a14fc9eed7e9de71c243acc506c1bf303072c40a

# Clean up exiting template stuff
rm -rf *
git commit -am "Removed OpenShift template code"

# Create new Rails app here
rails new . -f
bundle install
git add .
git commit -F- <<EOM
Adding initial Rails code:
rails new .

Created Gemfile.lock:
bundle install
EOM

# Add OpenShift gem mirror
sed -i "1i source 'http://mirror1.prod.rhcloud.com/mirror/ruby/'" Gemfile
bundle check
git commit -am "Added OpenShift gem mirror"

# Move action_hook
pushd .openshift/action_hooks/ > /dev/null
curl -O https://raw.github.com/openshift/rails-example/master/.openshift/action_hooks/deploy
vim -c 'gg=G' -c ':wq' deploy
git commit -am "Modified deploy action hook to run database migrations"
popd > /dev/null

# Forcing clean build
touch .openshift/markers/force_clean_build
git add .openshift/markers
git commit -am "Forcing clean build"

popd > /dev/null
```
