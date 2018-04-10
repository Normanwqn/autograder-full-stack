This repository contains Docker and other configuration files needed to run and deploy the autograder system.

# Setup

This tutorial will walk you through installing and running the autograder on your local machine.

## System Requirements
**Supported Operating Systems:**
- Ubuntu 16.04
- OSX 10.11.6 or later

It _might_ be possible to run the development stack on Windows using Docker CE for Windows. If you decide to try this, you're on your own.

Environments that will **NOT** work:
- Linux Subsystem for Windows (Do NOT attempt!)
- Cygwin (No way)
- CAEN Linux (Plz no)

Supported Browsers:
- Google Chrome 65 or later
- Mozilla Firefox 54 or later

## Install Docker Community Edition
Ubuntu: https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce-1

OSX: https://docs.docker.com/docker-for-mac/install/

## Clone and checkout
```
git clone --recursive git@github.com:eecs-autograder/autograder-full-stack.git
git checkout develop

cd autograder-server && git checkout develop && cd ..
cd autograder-website && git checkout develop && cd ..
```
If you accidentally left out the `--recursive` flag, you can get the same effect by running this command in the autograder-full-stack directory:
```
git submodule init --update
```

## Run the development stack
```
docker-compose -f docker-compose-dev.yml build
docker-compose -f docker-compose-dev.yml up
```
This will start the development stack in the foreground.
Note that you _usually_ won't need to rebuild the development stack.
If you run into a situation where changes aren't automatically detected,
kill the stack with Ctrl+C and rerun the two commands above.

Times when a rebuild is required:
- Changing the requirements.txt file in autograder-server
- Changing the package.json file in autograder-website

## Finish setting up the database
Run these commands in a _new_ terminal window.

Apply Django migrations:
```
docker exec -it ag-dev-django python3 manage.py migrate
```
Start a Python shell inside the ag-dev-django container:
```
docker exec -it ag-dev-django python3 manage.py shell
```
In the Python shell, create a course and add yourself as an administrator:
```
from autograder.core.models import Course
from django.contrib.auth.models import User

me = User.objects.get_or_create(username='jameslp@umich.edu')[0]
course = Course.objects.validate_and_create(name='My Course')
course.administrators.add(me)
```
Note, you can use a different username if you like. If you do, keep track of what you chose, as you'll need it when we visit the autograder in a browser.

## Load the web page
Navigate to `localhost` in your browser.

If you want to switch ports, you'll need to edit the file `autograder-full-stack/docker-compose-dev.yml`.
Under the `nginx` block, change `"80:80"` to `<port>:80`. For example, to change the port to 9001:
```
services:
  nginx:
    ...
    ports:
      - "9001:80"
    ...
```
Then, navigate to `localhost:9001` in your browser.

## "Authenticate"
The development stack allows users to manually specify the user they wish to log in as.
In order to specify your desired username, you'll need a browser plugin that lets you edit cookies, such as the
[EditThisCookie](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg?hl=en) for Google Chrome.

Using the plugin, set a cookie with key "username" and value "<desired username>".
If no cookie is set, it will authenticate you as jameslp@umich.edu by default.
If you specified a different username when setting up the database, use that username
as the value for the cookie you set.