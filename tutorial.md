# Tutorial: Windows Azure, Ruby on Rails, Capistrano 3 & PostgreSQL

This tutorial will show you how to create and deploy a basic Ruby on Rails app onto your own [Windows Azure](http://windowsazure.com) Linux Virtual Machine using [Capistrano 3](http://capistranorb.com) to manage the deployment tasks including database migrations and versioning. Be sure to follow closely and don't skip any steps, missing just one can result in lots of frustration (trust me, I know!).

## Time

![Time][clock] Approximately 40 - 60 minutes

## Assumptions

* You have a subscription ([or free trial](http://www.windowsazure.com/en-us/pricing/free-trial/)) with [Windows Azure](http://windowsazure.com)
* You have a [GitHub account](http://github.com)
* You are using OSX (though any linux distro should be fine)
* You have Ruby &amp; Ruby on Rails installed (options include [rubyonrails.org](http://rubyonrails.org/), [rbenv](https://github.com/sstephenson/rbenv), [bitnami](http://bitnami.com/stack/ruby) etc.)
* You are comfortable with [terminal](http://en.wikipedia.org/wiki/Terminal_(OS_X)) &amp; [ssh](http://en.wikipedia.org/wiki/Secure_Shell)
* You know basic commands for [nano](https://wiki.gentoo.org/wiki/Nano/Basics_Guide) or [vi](http://www.cs.colostate.edu/helpdocs/vi.html)

## Getting Started

This tutorial is based on this Windows Azure article [Deploy a Ruby on Rails Web application to a Windows Azure VM using Capistrano](http://www.windowsazure.com/en-us/develop/ruby/tutorials/web-app-with-capistrano/) and [Capistrano 3 Tutorial](http://www.talkingquickly.co.uk/2014/01/deploying-rails-apps-to-a-vps-with-capistrano-v3/) by [@TalkingQuickly](http://twitter.com/TalkingQuickly).

### Create your SSH Key
![Time][clock] 1 minute

You **will need to upload a Windows Azure compatible SSH key**. To create one, open `Terminal` & run the following in a folder where you wish to store your keys. I recommend using the `~/.ssh` folder. [Jeff Wilcox](http://www.jeff.wilcox.name/2013/06/secure-linux-vms-with-ssh-certificates/) has a great post on creating SSH Keys which I've borrowed from.

You'll be prompted for information like country, state or province, organization etc. You can put whatever you want in there, or leave it blank, up to you.

> **Don't** try and copy+paste this entire script in one shot, it won't work. Run each command separately.

<script src="https://gist.github.com/m-gagne/7afdcff597099e8b5fd1.js"></script>

### Create a Virtual Machine

![Time][clock] 5 minutes

Virtual Machine, VM, Virtual Private Server, VPS or server, basically it all means a "machine" in the cloud for you to setup.

The Windows Azure team has a [more detailed article](http://www.windowsazure.com/en-us/manage/linux/tutorials/virtual-machine-from-gallery/) here that I've used, but the basics are:

* log into [portal.windowsazure.com](http://portal.windowsazure.com)
* select New
* select Compute
* select Virtual Machine
* select From Gallery

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2013/11/new.png)

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2013/11/vm-add-from-gallery-620.png)

Select your **linux distro** (I'm partial to Ubuntu). Not sure which version? LTS stands for Long Term Support so that's a good choice. Once selected click **Next**

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2013/11/vm-gallery-linux-620.png)

Enter a unique **virtual machine name**, choose a **size**, enter your **username** (default is 'azureuser') and upload your SSH .pem file (`azure.pem`).

> Don't forget to **upload the public SSH key** (azure.pem)> While in the file upload dialog **to navigate to your .ssh folder** simply start typing `~/.ssh` and hit enter.

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2013/11/create-vm-ssh-key-620.png)

You can simply** use the defaults on step 3** and move on to step 4

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2013/11/create-vm-step-3-620.png)

In step 4, be sure to **add the HTTP endpoint** so you can access your app. Then click the `OK` icon in the bottom right to create your server.

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2013/11/create-vm-http-endpoint-620.png)

### Once your server is ready

![Time][clock] 5 minutes

It'll take about 4-5 minutes or so to create your server. Once it's ready open `Terminal` and connect to your server by running

<script src="https://gist.github.com/m-gagne/b0066b6a91db4068e1c0.js"></script>

Be sure to replace
* `<azureuser>` with your username that you specified during creation (default is azureuser)
* `<vmname>` with the  virtual machine name (or DNS name) provider during creation.

> When promoted be sure to agree to add your server to the known hosts.

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2014/03/azure-ror-tutorial/ssh_to_vm.png)

### Install Git, Ruby &amp; Nginx

![Time][clock] 12 minutes

Once connected to your server install [git](http://git-scm.com/) and clone the following file into `~/setup` and run the setup script. This script ([which you can see here](https://gist.github.com/m-gagne/9234940))  will

*   Install build tools
*   Install Node.js
*   Install Nginx
*   Setup a local (user) deployment of rbenv
*   Install (using [rbenv](https://github.com/sstephenson/rbenv)) Ruby 2.0.0-p451
*   Install the Bundler gem



<script src="https://gist.github.com/m-gagne/2ddfdaf2ca3cc1223035.js"></script>

### Install PostgreSQL

![Time][clock] 3 minutes

Time to install a database server, although this tutorial uses [PostgreSQL](http://www.postgresql.org/) you could just as easily use [MySQL](http://www.mysql.com/) or even [Windows Azure SQL](http://www.windowsazure.com/en-us/services/sql-database/).

<script src="https://gist.github.com/m-gagne/9259868.js"></script>

### Create a user & database

![Time][clock] 2 minutes

You'll need to edit PostgreSQL config to change from [Peer to MD5 authentication type to allow for password-based authentication](http://www.postgresql.org/docs/9.1/static/auth-methods.html).

<script src="https://gist.github.com/m-gagne/80c1d08ca47448860c8b.js"></script>

The above command opens the config file to line 90, all you need to do is change from peer (user) to md5 (user+pass) for authentication.

<script src="https://gist.github.com/m-gagne/8dba324a9c45ee1a741c.js"></script>

to

<script src="https://gist.github.com/m-gagne/5b60a831eac45664d11e.js"></script>

Now reload PostgreSQL configuration

<script src="https://gist.github.com/m-gagne/0f397bf81bb267a2d26c.js"></script>

Time to create our user & database. Don't forget to change

*   `my_username` to your desired username
*   `my_database` to your desired database name

<script src="https://gist.github.com/m-gagne/e2815b6a678b60a92f69.js"></script>

Verify it works by connecting to it

<script src="https://gist.github.com/m-gagne/d9bc3968e93fd89a5068.js"></script>

To quite simply type `\q <enter>`

### Kill the default nginx website

We are going to let Capistrano create a new nginx configuration for us that points to our app, but first we need to delete the default website.

<script src="https://gist.github.com/m-gagne/2c02d1ccc5b169673ba0.js"></script>

## Setting up your Ruby on Rails App

We will now setup your (local) machine & app for deployment.

### Capistrano

[Capistrano](http://capistranorb.com/) is a  remote server automation and deployment tool written in Ruby. We will use it to manage setting up and deploying our app to our server(s).

### Sample App

I recommend cloning [this sample app](https://github.com/m-gagne/ror-azure-demo) for two reasons. First if you don't yet have a Ruby on Rails app you can simply deploy this one. Second, it will make it easier to copy configuration files into your own application once you know how it works. GitHub repo: [https://github.com/m-gagne/ror-azure-demo](https://github.com/m-gagne/ror-azure-demo).

### Step by step instructions on integrating Capistrano 3

For detailed step by step instructions please reference this post [Capistrano 3 Tutorial](http://www.talkingquickly.co.uk/2014/01/deploying-rails-apps-to-a-vps-with-capistrano-v3/) by [@TalkingQuickly](http://twitter.com/TalkingQuickly).

### Starting with the sample app (Recommend approach)

![Time][clock] 4 minutes

Open `Terminal` on your local machine &  clone the sample into your code directory (for example `~/Code`)

<script src="https://gist.github.com/m-gagne/f2c49bdb873985e94cfd.js"></script>

#### database.yml

The sample app ships with an example database config file `database.example.yml`. Copy it to `database.yml` and configure it as needed. The sample uses PostgreSQL but you can use Mysql, SQLlite and more.

> Don't forget to `cd` into the app folder `cd ror-azure-demo`

<script src="https://gist.github.com/m-gagne/d43ad4ef89d8eab6d80f.js"></script>

You will want to edit the default values in `config/database.yml` to match your development environment

    development:
      adapter: sqlite3
      database: db/development.sqlite3
      pool: 5
      timeout: 5000

#### Gemfile

Gems needed for Capistrano deployment. These are already included in the sample apps `Gemfile` an are only included here for reference if you are adapting this to your own app.

<script src="https://gist.github.com/m-gagne/c449c7e62cd2af374ab0.js"></script>

Run the following to install any missing gems

<script src="https://gist.github.com/m-gagne/a79c7fb6d6fa49a2ed49.js"></script>

#### Capistrano Files
The following is the folders & files created by Capistrano if you were to simply run `cap install`. The sample app is already setup and this is not required.

<pre>
├── Capfile
├── config
│   ├── deploy
│   │   ├── production.rb
│   │   └── staging.rb
│   └── deploy.rb
└── lib
    └── capistrano
            └── tasks
</pre>

> The sample app also includes a number of configuration files to instruct Capistrano to setup nginx, unicorn and more. To better understand how that works see the files in `config/deploy/shared` and read this [tutorial](http://www.talkingquickly.co.uk/2014/01/deploying-rails-apps-to-a-vps-with-capistrano-v3/).

#### config/deploy.rb

This is the basis for your deployment configuration in Capistrano. The settings to change in this file are:

* `:application`
    * Set this to a name for your application
    * It will be used in the deployment for things like the deployment folder
* `:deploy_user`
    * Set this to the username you selected when you created your server
    * The default when setting up a server on Windows Azure is `azureuser`
* `:repo_url`
    * Set this to the URL for your applications GitHub repository

<script src="https://gist.github.com/m-gagne/2ef3fd15492164e1a0ab.js"></script>

#### config/deploy/production.rb

Capistrano allows you to create multiple stages (dev, test, production for example). In this tutorial we'll focus on production.

The idea is the `config/deploy.rb` we configured above contains the common configuration (independent of the stage) and the stage specific files contain the rest.

In `config/deploy/production.rb` the main line to edit is

<script src="https://gist.github.com/m-gagne/4a1c5b5af160d51b3887.js"></script>

You will need to change

* `yourserver.cloudapp.net`
    * change this to your your servers DNS address (you should just need to change yourserver to the name of the Virtual Machine you recently created.)
* `azureuser`
    * change this to the username you specific when creating your server. By default on Windows Azure this is `azureuser`

### Time to deploy

#### Setting up your apps configuration

![Time][clock] 2 minutes

The first step is tell Capistrano to deploy the basic configuration of your application to the desired stage. For this tutorial we'll deploy to `production` by running the following command

<script src="https://gist.github.com/m-gagne/4e65ed551c06ead966e5.js"></script>

This will create the following files & folders on your servers home (`~/`) directory:

    ~/apps/
    └── ror-azure-demo_production
      └── shared
          └── config
              ├── application.yml
              ├── database.example.yml
              ├── log_rotation
              ├── monit
              ├── nginx.conf
              ├── unicorn_init.sh
              └── unicorn.rb

So what did Capistrano do exactly?

*   Created an `apps` folder in your home directory
*   Created a folder for your app using the structure `appname_stage`
*   Created a `shared/config` folder for configuration that each release/deployment of your app will reference

You might have noticed that you have a `database.example.yml` file. That's because it's a bad idea to store your production credentials in your code. So you'll need to create a database.yml file on your production serer. For that you can simply copy the example file and edit as needed.

`ssh` to your server and run the following to create & edit a `database.yml` file

<script src="https://gist.github.com/m-gagne/9c6fc35c7ff5194ed625.js"></script>

This will create and open the following `database.yml` file

<script src="https://gist.github.com/m-gagne/5e4c85a8ef7381b2c692.js"></script>

You will want to edit it as appropriate, however if you are using PostgreSQL then all you'll need to edit is

* `username:`
* `password:`
* `database:`

#### Restart Nginx

While SSH'd into your server you will want to restart Nginx to pick up the new virtualhost that was added when you ran `cap production deploy:setup_config`

<script src="https://gist.github.com/m-gagne/5d2ec8c9a0f4b48367e2.js"></script>

#### Deploy your application

![Time][clock] 3 minutes

Time to deploy your application to your server! On your local machine in your app folder run

<script src="https://gist.github.com/m-gagne/697e37f7d2be2d1b32e7.js"></script>

This will take a few minutes as part of the initial (or 'cold') deployment is to compile and install all the gems.

### Fingers Crossed

If all went well you should now be able to point your browser to your server and see your app deployed in production on your very own Windows Azure Linux Virtual Machine!

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2013/11/achievement-unlocked-rails-azure-first-app.png)

### Learn More

Want to learn more? Hit up [windowsazure.com](http://windowsazure.com) for scenarios, documentation and tutorials, also why not check out [Microsoft Virtual Academy](http://www.microsoftvirtualacademy.com/product-training/windows-azure) which includes courses for Windows Azure.

Follow me for more on Windows Azure [@marc_gagne](http://twitter.com/marc_gagne)


[clock]: http://mediafiles.w00t.ms/Images/Articles/uploads/2014/03/azure-ror-tutorial/clock_28.png
