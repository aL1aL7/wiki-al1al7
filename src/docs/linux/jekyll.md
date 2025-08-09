# Jekyll - Install & Basics

## Install Jekyll

```bash title="Install requirements with apt-get"
apt-get install ruby ruby-dev build-essential
```

!!! danger "Attention !!!"
    Do not install gems as root user

Login as standard user and modify bash environment

```bash title="Modify path for ruby gems"
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME=$HOME/gems' >> ~/.bashrc
echo 'export PATH=$HOME/gems/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

```bash title="Install Jekyll and dependencies"
gem install jekyll bundler

# update jekyll or any other gem
gem update jekyll

# update Rubygems
gem update --system
```

## Working with Jekyll

```bash title="Start new website"
jekyll new the_blog
```

```bash title="Build website"
jekyll build
# => The current folder will be generated into ./_site

jekyll build --destination <destination>
# => The current folder will be generated into <destination>

jekyll build --source <source> --destination <destination>
# => The <source> folder will be generated into <destination>

jekyll build --watch
# => The current folder will be generated into ./_site,
#    watched for changes, and regenerated automatically.
```
