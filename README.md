Deploy Django App on Heroku Take-Aways.
=======================================================

## Install heroku CLI
[Sign up](https://signup.heroku.com/) to Heroku.

Then install the [Heroku Toolbelt](https://toolbelt.heroku.com/). It is a command line tool to manage your Heroku apps

After installing the Heroku Toolbelt, open a terminal and login to your account:
```bash
$ heroku login
Enter your Heroku credentials.
Email: newtonkiragu@herokudeploying.com
Password (typing will be hidden):
Authentication successful.
```

# Preparing the Application
In this tutorial I will deploy an existing project, [moringa-tribune-hosting](https://github.com/newtonkiragu/mtribune-hosting).
It's an very simple open-source Django project, that shows news posted.
Its available on [github](https://github.com/newtonkiragu/mtribune-hosting) so you can actually clone the repository and follow along or try it on your own existing django project.

## Startup
* create a folder, cd into it
* create, activate virtual env & install pip
    - Run `python3.9 -m venv --without-pip virtual` <br>
    - Run `source virtual/bin/activate` to activate and `.../deactivate` to deactivate virtual env.<br>
    - Run `curl https://bootstrap.pypa.io/get-pip.py | python` to install pip in virtual env.<br>
    - Run `python3 -m pip install --upgrade pip`to upgrade pip
* `python3 -m pip install django` to install django 
* in virtual `django-admin startproject <projectname> .` to create project
* in virtual `django-admin startapp <appname>` to create app
* in virtual `python3.9 manage.py runserver` to run server 
* in virtual `python manage.py createsuperuser` to create Admin

## Install all dependiencies at once
* cloudinary
* django-bootstrap4
* django-crispy-forms 
* django-heroku 
* gunicorn 
* Pillow 
* pyscopg2 
* python-decouple 
* typing-extensions 
* djangorestframework

```bash
pip install django-bootstrap4 cloudinary django-crispy-forms django-heroku gunicorn Pillow python-decouple typing-extensions djangorestframework
```
###### This will come with whitenoise, pyscopg2 and dj-database-url dependencies

## Db & Migrations
 1. Db
     - After installing psycopg2
     - run `psql` in terminal
     - `CREATE BATABASE <dbname>;`
     - `\c <dbname>` To connect to db.
     - `\password` To set your db password
 2. Migrations
     - `python3.9 manage.py check` To check models
     - `python3.9 manage.py makemigrations <app>`  OR  `python3.9 manage.py makemigrations`  To make migrations
     - `python3.9 manage.py sqlmigrate <app> 0001` To view migrations 
     - `python3.9 manage.py migrate` To run migrations

## Assumptions
* Your familiar with the basics of django e.g concept of apps, settings, urls, basics of databases 
* You have django application that you want to deploy to heroku
* You are familiar with virtual environments - not a must but the knowledge would be a plus
* Your deployment db is postgres

We need to add the following to our project, we will cover each of them in detail in the below section

* Create a `Procfile` in the project root;
* Create `requirements.txt` file ;
* `pip freeze > requirements.txt` to add all your installed dependencies in requirements.txt file.
   ##### ONLY If you have cloned/copied dependencies requirements.txt file
   - `pip install -r requirents.txt` To install dependencies written in requirements.txt file
* Install django-heroku `pip install django-heroku`
* Install gunicorn`pip install gunicorn==20.1.0`
* Add `Gunicorn` to `requirements.txt` 
* A `runtime.txt` to specify the correct Python version in the project root;
* Configure `whitenoise` to serve static files.

## Procfile
Heroku apps include a `Procfile` that specifies the commands that are executed by the app’s dynos. 

For more information read on the [heroku documentation](https://devcenter.heroku.com/articles/procfile).

Create a file named `Procfile` in the project root with the following content:
```
web: gunicorn your_project_name.wsgi --log-file -
```

## Runtime.txt
This file contains the python version you are using for heroku to use, create `runtime.txt` in your project root and add your python version in the following format
```
python-3.9.2
```
List of [Heroku Python Runtimes](https://devcenter.heroku.com/articles/python-runtimes).

## .gitignore
gitignore file should have the following:
```bash
__pycache__/
virtual/
.vscode/
.env
*.pyc

*.sh   #for start.sh file, used in flask
```


## Whitenoise: Django Static Files settings
Lets first configure static related parameter in `settings.py`
```python 
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/1.9/howto/static-files/
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATIC_URL = '/static/'

# Extra places for collectstatic to find static files.
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'static'),
)
```
Turns out django does not support serving static files in production. However, WhiteNoise project can integrate into your Django application, and was designed with exactly this purpose in mind.

Lets first install Whitenoise   `pip install whitenoise`

Install `WhiteNoise` into your Django application. This is done in `settings.py’s middleware section` (at the top):
```python 
MIDDLEWARE_CLASSES = (
    # Simplified static file serving.
    # https://warehouse.python.org/project/whitenoise/
    'whitenoise.middleware.WhiteNoiseMiddleware',
    ...
```
Add the following setting to `settings.py` in the static files section to enable gzip functionality.
```python
# Simplified static file serving.
# https://warehouse.python.org/project/whitenoise/

STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

# configuring the location for media
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# Configure Django App for Heroku.
django_heroku.settings(locals())
```
## Installed Apps
* Add your installed apps on the installed app list in seettings.py file
* Bootstrap `pip install python-bootstrap4`
* crispy forms `pip install django-crispy-forms`
* Cloudinary `pip install cloudinary`

 ### Claudinary configurations 
 settings.py file
 import cloudinary 
 
 ```bash
   cloudinary.config( 
       cloud_name = config('CLOUDINARY_NAME'), 
       api_key = config('CLOUDINARY_API_KEY'), 
       api_secret = config('CLOUDINARY_API_SECRET') 
     ) 
     
   DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.MediaCloudinaryStorage'
   # You can uncomment this cloudinary_storage if you have default file storage in your local machine
   # Usually this STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
 
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.* messages',
    'django.contrib.staticfiles',
    'insta',
    'bootstrap4',
    'crispy_forms',
    'cloudinary',
    'rest_framework',
    'rest_framework.authtoken',
]
```
models.py
```bash
import cloudinary
from cloudinary.models import CloudinaryField

class Classname(models.Model):
    prof_pic=CloudinaryField('image')
```

## Requirements.txt
 'cloudinary',If your under a virtual environment run the command below to generate the requirements.txt file which heroku will use to install python package dependencies 

`pip freeze > requirements.txt`
Make sure you have the following packages if not install the using pip then run the command above again
```
config==0.3.9
cloudinary==1.29.0
dj-database-url==0.5.0
Django==4.0.5       
django-bootstrap4==22.
django-crispy-forms==1.14.0
django-heroku==0.3.1
django-registration==3.1.2
gunicorn==19.9.0
Pillow==9.1.1
psycopg2==2.9.3
python-decouple==3.6
pytz==2018.5
whitenoise==6.1.0
```
* Remove pkg-resources from the requirements.txt file as it might lead to some errors.<br>
If you are following along with the mtribune app you should use the provided `requirements.txt` as you need to install more python packages, for any app just make sure you have the above packages as a plus.

## Optional but very helpfull settings

### python-decouple and dj-database-url
Python Decouple is a must have app if you are developing with Django. It’s important to keep your application credentials like API Keys, Amazon S3, email parameters, database parameters safe, specially if it’s an open source repository. Also no more development_settings.py and production_settings.py, use just one settings.py for your whole project.

Install it via `pip install python-decouple`

dj-database-url is a simple Django utility allows you to utilize the 12factor inspired `DATABASE_URL` environment variable to configure your Django application.

Install it via `pip install dj-database-url`

Firts create a `.env` file and add it to `.gitignore` so you don’t commit any sensitive data to your remote repository.
below is an example of configurations you can add to the `.env` file.

```python
#just an example, dont share your .env settings
SECRET_KEY='342s(s(!hsjd998sde8$=o4$3m!(o+kce2^97kp6#ujhi'
DEBUG=True #set to false in production
# POSTGRESQL DB
DB_NAME='tribune'
DB_USER='user'
DB_PASSWORD='password'
DB_HOST='127.0.0.1'
# STORE IMGS IN CLOUDINARY
CLOUDINARY_NAME='abcdef'
CLOUDINARY_API_KEY='468hgtcbjk'
CLOUDINARY_API_SECRET='fhjm8765dvhjkkllgg58sdf'
MODE='dev' #set to 'prod' in production
ALLOWED_HOSTS='<herokuAppName>.herokuapp.com'  ###### OR ALLOWED_HOSTS='*' If you are not hosting
DISABLE_COLLECTSTATIC=1
EMAIL_USE_TLS =True
EMAIL_HOST='smtp.gmail.com'
EMAIL_PORT=587
EMAIL_HOST_USER='youremailaddress@gmail.com'
EMAIL_HOST_PASSWORD='youremailpassword'
```

We then edit `settings.py` to enable decouple to use the `.env` configurations.

 ```python
import os
import django_heroku
import dj_database_url
from decouple import config,Csv
import cloudinary

# Email configurations remember to install python-decouple
# We use the config function to connect to the environment variables
EMAIL_USE_TLS = config('EMAIL_USE_TLS')
EMAIL_HOST = config('EMAIL_HOST')
EMAIL_PORT = config('EMAIL_PORT')
EMAIL_HOST_USER = config('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = config('EMAIL_HOST_PASSWORD')

MODE=config("MODE", default="dev")
SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
 # development
if config('MODE')=="dev":
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',  # OR 'ENGINE': 'django.db.backends.sqlite3', For sqlite DB
            'NAME': config('DB_NAME'),
            'USER': config('DB_USER'),
            'PASSWORD': config('DB_PASSWORD'),
            'HOST': config('DB_HOST'),
            'PORT': '',
        }
        
    }
# production
else:
    DATABASES = {
        'default': dj_database_url.config(
            default=config('DATABASE_URL')
        )
    }

db_from_env = dj_database_url.config(conn_max_age=500)
DATABASES['default'].update(db_from_env)

ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv()) 
    OR
ALLOWED_HOSTS = 'herokuAppName.herokuapp.com'

#for api token
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.TokenAuthentication',
    )
}
```
#### DB settings remains the same if you are using sqlite
```
python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / '<sqlitedbname>',
    }
}
```
* Install sqlite3 `sudo apt install sqlite3`
* Install SQLitebrowser `sudo apt install sqlitebrowser`
* Uninstall SQLitebrowser `sudo apt --purge remove sqlitebrowser` 
##### CONNECT to slite3 DATABASE, type this in terminal: 
* `sqlite3`
* sqlite>`.database` to view db available    
* sqlite>`.open <dbname>` to open the db 
* sqlite>`.tables` to view all the available tables OR sqlite>`.fullschema` 
* sqlite>`.schema <tablename>` to view specified table

### Django-Heroku
We then install Django-Heroku which will automatically configure `DATABASE_URL`, `ALLOWED_HOSTS`, `WhiteNoise` (for static assets), `Logging`, and `Heroku CI` for your application.
```bash
pip install django-heroku 
```
**NB:** Remember to remove `pkg-resources` from `requirements.txt` for easy deployment.      
    
# Lets deploy now
First make sure you are in the root directory of the repository you want to deploy
```bash
pip freeze > requirements.txt
```
```bash
heroku login
```
Next create the heroku app
```bash
heroku create <herokuappname>
```
Create a postgres addon to your heroku app
```bash
heroku addons:create heroku-postgresql:hobby-dev
```
Next we log in to [Heroku dashboard](https://dashboard.heroku.com) to access our app and configure it

![Imgur](https://i.imgur.com/dbDQxlJ.png)

To add all your configurations in `.env` file directly to heroku<br>
Remember to first set `DEBUG` to false in .env file and confirm that you have added all the confuguration variables needed.<br>
```bash
heroku config:set $(cat .env | sed '/^$/d; /#[[:print:]]*$/d')
```
 
![Imgur](https://i.imgur.com/D0s6BkV.png?1)

Click on the Settings menu and then on the button Reveal Config Vars <br>
Next add all the environment vaiables, if you dint add your .env variables to heroku.<br> 
By default you should have `DATABASE_URI` configuration created after installing postgres to heroku.

## Heroku Conf Variables
* Set debug=True
* Remove Single/Doube Quotes in Heroku Config Variables

## Pushing to heroku
confirm that your application is running as expected before pushing, runtime errors will cause deployment to fail so make sure you have no bugs, you have all the following `Procfile`, `requirements.txt` with all required packages and  `runtime.txt` .
```bash
git add . && git commit -m "message"
```
```bash
git push heroku master
```
If you did everything correctly then the deployment should be done after a while with an output like this

```bash
Enumerating objects: 94, done.
Counting objects: 100% (94/94), done.
Delta compression using up to 8 threads.
Compressing objects: 100% (84/84), done.
Writing objects: 100% (94/94), 3.35 MiB | 630.00 KiB/s, done.
Total 94 (delta 24), reused 0 (delta 0)
remote: Compressing source files... done.

#.......

remote: -----> Compressing...
remote:        Done: 56.3M
remote: -----> Launching...
remote:        Released v6
remote:        https://mtr1bune.herokuapp.com/ deployed to Heroku
remote: 
remote: Verifying deploy... done.
To https://git.heroku.com/mtr1bune.git
 * [new branch]      master -> master

 ```

 ## Run db migrations
If you instead wish to push your postgres database data to heroku then run
##### Option 1
- `heroku pg:psql` -Allows you to see the psql DATABASE_URL 
- `heroku pg:reset` -If the psql DATABASE_URL is empty run this command 
- `heroku pg:push <The name of the db in the local psql> <DATABASE_URL> --app <heroku-appname>`
- This will go with django admin 
##### Option 2
- `heroku pg:reset` -If the psql DATABASE_URL is empty run this command 
- `heroku pg:push <local psql dbname> DATABASE_URL`
- This will go with django admin 

## configure Heroku Django admin
##### Done ONLY when heroku django admin brings Password Error!
`heroku run python manage.py createsuperuser` to create Heroku django admin <br>
 - This prompts one to enter username, email, password and confirm password <br>
Then `git push heroku master` & `heroku run python manage.py migrate` 

## Comment
This process was a lot and you can easily mess up as I did, I suggest analyzing the part where you went wrong and going back to read on what you are supposed to do. I also highly recommend going through official documentations about deploying python projects to heroku as you will get a lot information that can help you debug effectively. I will provide some links in the resources section.

Remember heroku does not offer support for media files in the free tier subscription so find some where else to store those e.g Cloudinary Amazon s3.

#esources
## heroku Docs
* https://devcenter.heroku.com/articles/heroku-postgresql
* https://devcenter.heroku.com/articles/django-assets
* https://devcenter.heroku.com/articles/python-runtimes
* https://devcenter.heroku.com/articles/procfile
* https://devcenter.heroku.com/articles/getting-started-with-python#introduction
* https://devcenter.heroku.com/articles/deploying-python
* https://devcenter.heroku.com/articles/django-app-configuration
* https://devcenter.heroku.com/articles/python-gunicorn

## very helpfull articles
* https://simpleisbetterthancomplex.com/tutorial/2016/08/09/how-to-deploy-django-applications-on-heroku.html
* https://simpleisbetterthancomplex.com/2015/11/26/package-of-the-week-python-decouple.html

