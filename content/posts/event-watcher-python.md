---
title: "Event Watcher Manager Python3"
date: 2017-07-30
tags: ["python"]
draft: false
---

For one specific request, I had to build an automated program that reacts to SFTP user updates.  
The main technical challenge was that the SFTP protocol has no logging system of its own.

I had heard about [the pyinotify library](https://github.com/seb-m/pyinotify/blob/master/python3/pyinotify.py), so I started working with it.  
**This post covers the project's core mechanism**; for more detail, I invite you to read the source code.  

#### Technical requirements for the project

**Context:**  
The program needs to monitor SFTP user actions. Each user's home directory is an NFS mount.  
Users can upload anything to their account, but the program must detect video uploads (in specific formats) and then trigger a series of actions.  

**The workflow should be as follows:**  
For each detected video, check that its format is allowed, convert it to .flv, generate its metadata, create a screenshot (video thumbnail), and send an email to the SFTP user's address.  

#### Preparation

A few specific constraints had to be handled up front, the main one being how to associate a user's account with their email address once a video upload is detected.  
That's why I chose to start with a static file, since the list of users is known in advance.  

```python
# Création de la classe
class Person:
    def __init__(self, name, surname, login, homedir, email, realpath):
        self.name = name
        self.surname = surname
        self.login = login
        self.homedir = homedir
        self.email = email
        self.realpath = realpath
#
# Et instanciation des utilisateurs
#
user_test = Person(
        'Test',
        'test',
        'test',
        '/test/test/test.fr',
        'test@test.fr',
        '/test.fr/')
```

Next, I worked on the _core_ of the program: first reading the information from the user file, then grouping it into lists.  

```python
# Définition des listes
user_list = []
user_path = []
user_mail = []
user_realpath = []
# On appelle le fichier utilisateur pour implémenter les listes
### LOGIN ###
for users.obj in gc.get_objects():
    if isinstance(users.obj, users.Person):
        user_list.append(users.obj.login)
### DOCUMENT_ROOT ###
for users.obj in gc.get_objects():
    if isinstance(users.obj, users.Person):
        user_path.append(users.obj.homedir

### EMAIL ###
for users.obj in gc.get_objects():
    if isinstance(users.obj, users.Person):
        user_mail.append(users.obj.email)
### REALPATH ###
for users.obj in gc.get_objects():
    if isinstance(users.obj, users.Person):
        user_realpath.append(users.obj.realpath)
```

The first function defined handles the associative lookup, using an ID-based position system.  

```python
def owner_func():
    for filename in multiple_file_types('*.avi', '*.mov', '*.mp4', '*.mpg', '*.wmv'):
        # On affiche l'utilisateur
        owner = pwd.getpwuid(os.stat(filename).st_uid).pw_name
        # On vérifie que ce dernier est bien présent dans notr liste
        if owner in user_list:
            print('The owner of file is :   ' + owner)
            position = user_list.index(owner)
            print('Owner_login_path :' + user_path[position])
            mail = user_mail[position]
            mailp = print(mail)
            realpath = user_realpath[position]
            return(realpath)
        else:
            print('The owner was not found.')
```

#### Introduction to Pyinotify

Pyinotify provides functions related to data creation and deletion (IN_CREATE and IN_DELETE).  
Once the basic setup was done, I used these definitions to implement the **owner_func()** function.  

```python
class EventHandler(pyinotify.ProcessEvent):
    def process_IN_CREATE(self, event):
        print('\n\n===========================')
        print(time.strftime("%d/%m/%Y %H:%M:%S"))
        evp = print('Path complet :' + '\'' + event.pathname + '\'')
        evn = print('Objet crée : ' + '\'' + event.name + '\'')
        Ouser = event.pathname.split('/')[2]
        print('Utilisateur : ' + Ouser)
        Oplace = [i for i,x in enumerate(user_list) if x == Ouser][0]
        Orelatif = user_realpath[Oplace]
        Omail = user_mail[Oplace]
        print('Chroot Sftp : ' + Orelatif)
        print('Mail : ' + Omail)
        command = 'convert.bash '+str(event.pathname)
        previous_size=0
        upload_finished = False
        try:
            while True:
                time.sleep(1)
                size=os.stat(event.pathname).st_size
                print(previous_size)
                print(size)
                if size == previous_size:
                    break
                else:
                    previous_size = size
        except:
            return False
        print(command)

        os.rename(event.pathname, event.pathname.replace(" ", "_"))
        neventpathname = event.pathname.replace(' ', '_')
        neventname = event.name.replace(' ', '_')
        print(neventpathname + neventname)
        p1 = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
        print('convert.bash', neventpathname)
        print(p1.stdout.read())
        p1.wait()
        p2 = subprocess.Popen(['generatemetadata.bash', neventpathname])
        p2.wait()
        p3 = subprocess.Popen(['extractpng.sh', neventpathname])
        p3.wait()
        if neventname.endswith('.html') :
            p5 = subprocess.Popen(['mail.bash', Omail, Orelatif,  neventpathname])
            print('MAIL SENT')
        print('===========================')
    def process_IN_DELETE(self, event):
        print('\n\n===========================')
        print(time.strftime("%d/%m/%Y %H:%M:%S") + "    Deleting:", event.pathname)
        print('===========================')
```

**Explanation:**  
Pyinotify's principle is to trigger an automatic action on a given event. Let's take the upload of a video in the right format as our event.  
The **IN_CREATE** function is then triggered and gives us the initial information: the creation date, the full path of the event, the relative path (the SFTP user's home directory), the user, their email, and so on.  

In a second step, we convert the file to the right format (with **ffmpeg**). But first, we need to make sure the video has finished uploading. That's handled by this block:  

```python
        command = 'convert.bash '+str(event.pathname)
        previous_size=0
        upload_finished = False
        try:
            while True:
                time.sleep(1)
                size=os.stat(event.pathname).st_size
                print(previous_size)
                print(size)
                if size == previous_size:
                    break
                else:
                    previous_size = size
        except:
            return False
        print(command)
```

Once that's confirmed, we can run the bash scripts as subprocesses.  

#### Benefits

There are several benefits here.  
The first is obvious: it's now possible to log activity on a service that didn't natively support it. The second is that setting up a logging and PID system becomes very simple.  

```python
notifier.loop(daemonize=True, callback=on_loop_func, pid_file='logs/pyinotify.pid', stdout='logs/%s.log' % timestr)
```

This opens up an interesting avenue for event automation, using a Python subroutine that builds directly on work the kernel already does out of the box.  

I've been thinking about building a kind of API to make this type of operation scalable across different environments.  
This project, still sitting on my famous **ToDoList**, would end up close to a master/slave system, essentially an imitation of what already exists with Puppet, but in Python.  

Wait and see what comes of it!
