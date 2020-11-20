# Blog Template and Deployment

This project has been created in support of Assignment 7.5, centering on deployment.  Further, this has been extended to assist with your final project.

A significant part of this tutorial is based on [How To Set Up Django with Postgres, Nginx, and Gunicorn on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-20-04), written by [Erin Glass](https://www.digitalocean.com/community/users/eglass).  There are also elements from Antonio's Chapter 14 that are useful as well.

In this project, I show some steps regarding:
* How to select a [Bootstrap 4](https://getbootstrap.com/) [template](https://startbootstrap.com/templates/blog-home)
* Deployment to our VPS

## Preparing for Deployment

For our purposes here, the first three steps in Erin's tutorial should be followed again for a new database and database user:

* [Installing the Packages from the Ubuntu Repositories](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-20-04#installing-the-packages-from-the-ubuntu-repositories)
* [Creating the PostgreSQL Database and User](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-20-04#creating-the-postgresql-database-and-user)
* [Creating a Python Virtual Environment for your Project](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-20-04#creating-a-python-virtual-environment-for-your-project)

However, we do not need to start a new django project as we have one in source control.  Also, it would be best if the project were located outside of a combined repository.  Either way will work, however.

## Ensuring our domain name is associated with our servers

In Digital Ocean's cloud dashboard, you go to networking and associate your domain with your droplet.  At your domain registrar, you provide the nameserver information. This is necessary for SSL to work and a requirement of the deployed final project.

## Ensuring Nginx is set for TLS/SSL

We also need to ensure that Nginx is set to go with Let's Encrypt prior to the Gunicorn work with Django:

* [How To Install Nginx on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04)
* [How To Secure Nginx with Let's Encrypt on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04)

## Finding a Bootstrap Blog Template

[Bootstrap 4](https://getbootstrap.com/) is a convenient CSS framework that assists you in creating responsive and professional web frontends and pages easily.

Acuity with web frontends is a course (or three) of its own, so I'll demonstrate finding a template that we can use.  For this, I have used the [Start Bootstrap](https://startbootstrap.com/) website, but there are many, many other sources for paid and free Boostrap 4 templates.  An alternative is [materialize](https://materializecss.com/), but there are many other CSS frameworks beyond these.

Another approach is to use a tool like [Bootstrap Studio](https://www.bootstrapstudio.io/), but the template approach is even faster.

In the video, I demonstrated these two:

* [Blog Home Page](https://startbootstrap.com/templates/blog-home/)
* [Blog Post Page](https://startbootstrap.com/templates/blog-post/)

## Using Git for Deployment

I refer to [this guide](http://rogerdudler.github.io/git-guide/) in the video. It covers the basics likes:

* starting your repo
* adding and commiting code
* pulling code
* versioning with branches

### Steps when you get started

* Make a new repo on github
* here are some of the suggested steps when you first make that repo:
* these work both local and on vps:
    * `git init` - only needed once 
    * `git add -A` - do this everytime you want to push to github
    * `git commit -m "some sensible and descriptive commit message describing why the codebase has changed"` - do this everytime you want to push to github
    * `git remote add origin https://github.com/<your-account>/<your-repo>.git` - only needed once
    * `git push -u origin master` - do this everytime you want to push to github

You would do the init and remote parts on the VPS.

Once your code is up on github, then you can use the `requirements.txt` to grab and install the required python packages.

If you have updated your virtual environment with new or updated packages, ensure that you export them to `requirements.txt`:

`pip freeze > requirements.txt`

## Managing settings

Antonio covers the need to manage settings differently depending on what stage of development your application is at.  As such, we establish different settings files to be used conditionally depending on the current environment:

* Development/Local
* Production (deployed to VPS)

take note that when you use this settings approach, you can always use the `--settings` switch for the `manage.py` command:

`manage.py runserver --settings=educa.settings.local`

manage.py runserver --settings=blog_deploy.settings.local

Or, you are expected to configure the [DJANGO_SETTINGS_MODULE environment variable](https://docs.djangoproject.com/en/3.1/topics/settings/#designating-the-settings).  You can stick to the original approach and just use a single `settings.py` file as well.

### Things to do on the server

Now, to get the code runing on the VPS, we'll simply pull the code:

* `git pull origin master`
* run any migrations if you've updated them
* run `python manage.py collectstatic`
* Restart the gunicorn and nginx services

### Nginx to serve and facilitate /media/

The best solution for media files, which is where user uploads will go, is to follow Erin Glass' lead and use Nginx.  Here is a part of my site config file in `/etc/nginx/sites-available`:

```
location /media {
        autoindex on;
        alias /home/jeff/buffteksdir/media/;
}
```
This directive specifies that nginx should serve files from the `media` folder.

This should go right below the `server_name` directive in the file.

The `alias` should actually go to where you have your application installed.  

### DATABASE_URL

Another way to provide connection information for Postgres is using a database url:

`DATABASE_URL=postgres://DB_USER:DB_PASS@DB_HOST:DB_PORT/DB_NAME`

Above is the variable you put into .env and the elements of authentication that you place into each part.

### WeasyPrint

The PDF handling in WeasyPrint requires [additional package installations on Ubuntu](https://weasyprint.readthedocs.io/en/stable/install.html#debian-ubuntu).

### Whitenoise

Bear in mind that I use [Whitenoise](http://whitenoise.evans.io/en/stable/) for static file serving rather than what Antonio and Erin show.

### Serving Media Files (uploads)

Despite the availability of [Whitenoise](http://whitenoise.evans.io/en/stable/), we still need to use nginx or [Django Storages](https://django-storages.readthedocs.io/en/latest/) to go that last mile of serving and accepting media files.  I have mentioned this previously.

### REMEMBER TO CREATE A .ENV FILE ON THE SERVER

This is what I used in this project:

![Blog Deploy ENV](https://i.imgur.com/v9WE8Lg.png)

### Extras

* I used the [favicon.io](https://favicon.io/favicon-generator/) site to generate a favicon.ico.
* I referred to [this tutorial](https://learndjango.com/tutorials/django-favicon-tutorial) regarding the use of favicons.
* I use [django-crispy-forms](https://django-crispy-forms.readthedocs.io/en/latest/) to help with including [Bootstrap 4](https://getbootstrap.com/).
* I use the built-in [humanize](https://docs.djangoproject.com/en/3.1/ref/contrib/humanize/) filter to make numbers more readable.

# The Heroku Alternative

While I felt it important to show you a fully controlled and **manual** way to get deployed on a VPS that you fully control, it is also possible to use a [Platform as a Service](https://en.wikipedia.org/wiki/Platform_as_a_service) provided for deployment, such as [Heroku](https://devcenter.heroku.com/start), [AWS](https://aws.amazon.com/), or [PythonAnywhere](https://www.pythonanywhere.com/).

I will show steps for Heroku.  

**NOTE**: If, after you review these steps and utilize this deployment strategy, I will withold some of the **VPS** points allocated for the project as that was an explicit requirements.  However, it may be that this PaaS route becomes your only viable solution.

I will generally follow the steps in [this resource](https://devcenter.heroku.com/articles/getting-started-with-python), although it seems to be a slight bit out of date.

## Pre-Requisites

You need to have the following in place:

* [Obtain a Heroku account](https://signup.heroku.com/dc). You start at a free tier.
* Have python installed locally
* Have Postgres installed locally

## Heroku CLI

We'll want to use the [Heroku CLI utility](https://devcenter.heroku.com/articles/getting-started-with-python#set-up) for heroku deployment.

To get going, you type: `heroku login`

At the terminal.

## Getting Started

Heroku provides a barebones [Django starter app](https://github.com/heroku/python-getting-started) to work with, but we already have an app.  As such, we'll follow their steps without actually cloning their app.  That said, it might be a good idea to work through their tutorial entirely as a dry run.

## Before you Create the App

The Heroku tutorial jumps straight into deploying their demo app, but that is a very simple example.  There are some things we'll want to do first:

* First, we'll want to double-check that, in addition to Django, and all the other packages we've used with Antonio, that we have the following packages:
    * `pip install dj-database-url gunicorn whitenoise`
* Also, ensure that we have an up-to-date `requirements.txt` file: `pip freeze > requirements.txt` (this file should exist alongside `manage.py` in your project folder)
* Heroku also needs a `Procfile` to help instruct Heroku on what to do with your application.  Put the following in that file:
    * `web: gunicorn <name-of-your-project>.wsgi --log-file -`
    * for those of us on windows, also include a `Profile.windows` file that contains this: `web: python manage.py runserver 0.0.0.0:5000`
* `Runtime.txt` file is used to specify the version of Python you want running: `python-3.8.6` for instance
* Modifying your project's `settings.py` file:
    * `DEBUG=False`
    * Change allowed hosts: `ALLOWED_HOSTS = ['127.0.0.1', '.herokuapp.com']`
    * Disable Django’s static file handling, allowing WhiteNoise to handle static files:
    ```
    INSTALLED_APPS = [
        'whitenoise.runserver_nostatic',
        'django.contrib.staticfiles',
        # ...
    ]
    ```
    * Optionally, update database settings to use `dj_database_url`

## Config

Either though the web dashboard, or through the CLI, you won't be placing a .env file on heroku like you would your VPS.  Rather, you set `config vars` (really just environment variables) for Heroku to read.  This part of the Heroku documentation discusses this further: [Configuration and Config Vars](https://devcenter.heroku.com/articles/config-vars).

**NOTE!!!**: Most of your config variables must be in place prior to attempting a deployment.  Without variables like the secret key, you will not be able to successfully deploy.

## Deployment

The steps to deploy are actually very straightforward with Heroku.  All of the necessary steps are accomplished with the Heroku CLI.

* Make sure you commit and push to Github first.
* Create app container: `heroku create <name-of-app>`
    * The heroku tutorial has you draw a random name, but you can specify a name.  
    * Your app will be deployed to a url like this: `https://<name-of-app>.herokuapp.com`

## Running Locally

What is nice about the Heroku CLI is that you can emulate how your app will (or won't) work on Heroku byy using the Heroku CLI's `local web` option:

On Windows: `heroku local web -f Profile.windows`
On Mac/Linux: `heroku local web`

### Media Files and Heroku

While Whitenoise will handle static files, you'll still have an issue with the `media` files (user uploads) and this is a painful achilles heel of Django.  

#### Ephemeral FileSystem

Initially, you might think that your uploads work, but you'll notice that the entire filesystem that hosts your Heroku app will be wiped out daily - moreso if you remain on free levels of service.  

Heroku recommends using [AWS S3](https://aws.amazon.com/s3/), but you could also use [Digital Ocean](https://www.digitalocean.com/products/spaces/) or [Linode](https://www.linode.com/products/object-storage/).  Of course this means maintaining separate accounts with multiple services, but that too is a fact of life in application deployment.  Honestly, [AWS](https://aws.amazon.com/) is something you want to learn eventually, but it is its own ball of wax.

There are also static files addons available, you are again getting into more paid territory.  Much as the case with ecosystems like [WordPress](https://wordpress.org/), you'll find that the deeper you go, the more there is to pay for. This is why I started you out on the VPS route: if you are willing to learn the details, you can stay at the $5/month level with a DIY approach.

### Pushing Changes

Now, you'll be pushing changes to two repositories:

* `git push -u origin master` to go to Github
* `git push heroku master` for deployment to Heroku

**NOTE**: There is a effort underway to change nomenclature of the default root branch in git from **master** to **main**.  As you encounter tutorials and articles, consider these to inchangeable from a functional perspective.  While ultimately, there are no code-functional differences, you can change the name of the **master** branch to **main**. Steps on how to do so, and the background as to why this phenomenon has arisen, are covered in [this article](https://www.kapwing.com/blog/how-to-rename-your-master-branch-to-main-in-git/).

### Console Access

What is very handy is your ability to quickly start a console sesion using the Heroku CLI.  This could be useful to run `python manage.py shell` or any other `manage.py` task.  

`heroku run python manage.py shell`

Further, you can get a `bash` shell going.

`heroku run bash`

### Working with the Database

Postgres is the preferred RDBMS for Django and your deployment would have created the database for you.  However, Heroku provides options to work with the database using the `heroku pg` command.  Further, your database is accessible via `https://<your-app-name>.herokuapp.com/db`. 

It will be neccessary to use remote console access to run things like migrations: `heroku run python manage.py migrate`

Lastly, you can get a database prompt directly using `heroku pg:psql` in order to issue queries and otherwise check on things directly.  Take care that you don't mangle the schemas or your migrations will be out of sync and your app and/or data will be broken.

## Application Error

Before you celebrate, you'll often see "Application Error" when, despite your app working locally, your app fails to run in deployment.

Fortunately, you'll get the correct advise on the error page in the browser.  That advice is to run `heroku logs --tail`