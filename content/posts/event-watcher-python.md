---
title: "Event Watcher Manager Python3"
date: 2017-07-30
tags: ["python"]
draft: false
---

On one specific request, I had to work on the elaboration of an automating program reacting on SFTP users updates.  
The main technical issue of this request is that the SFTP protocol does not have a logging system.

I had heard about [the pyinotify library](https://github.com/seb-m/pyinotify/blob/master/python3/pyinotify.py) so I started working on it.  
**The project is presented in its primary mechanism**, for more details, I invite you to read the sources.  

#### Technical requests concerning the project

**Details of the context of realization:**  
The program must monitor SFTP user actions. These users have their HomeDir which are NFS mounts.  
The user can upload anything to his account, but the program must detect video uploads (in some formats) and then perform a series of successive actions.  

**The operation should be as follows:**  
For each detected video repository, the video must check the allowed format, then it must be converted to .flv, have metadata, have a scree nshot (video thumbnail) and an email must be sent to the SFTP user's email address.  

#### Preparation

Several specific details are binding to begin with. The main one being the association of the account user's mail to his own mail with the detection of the video repository.  
That's why I chose to start on a static file, knowing in advance the list of users.  

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

Then I worked on the _core_ of the program. First of all to get the information from my user file and to make a grouping by list.  

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

The definition of a first function whose purpose will be the associative call with an ID position system.  

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

#### Pyinotifier Introduction

Pyinotifier has functions related to the creation and deletion of data (IN_CREATE and IN_DELETE).  
Once the basic setup was done, I used the basic definition to implement with the **owner_func()** function.  

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

**Explanations :**  
The principle of Pyinotify is to create an automatic action at a given event. Let's take the deposit of a video corresponding to the right format as the event.  
The **IN_CREATE** function is then triggered and will send us first information including: the creation date, the full path of the event, the relative path (HomeDir of the SFTP user), the user, his email...  

In a second step we apply the conversion to the right format (with **ffmpeg**). However, we have to make sure that the video is complete and submitted. This corresponds to the block :  

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

Then we will be able to run the bash scripts in subprocess.  

#### Interests

The interests are multiple!  
The first one is obvious since it is now possible to perform a logging on a service that was not natively available. The second one is that it is possible to set up a logging and PID system very simply.  

```python
notifier.loop(daemonize=True, callback=on_loop_func, pid_file='logs/pyinotify.pid', stdout='logs/%s.log' % timestr)
```

This opens a very interesting new door on event automation by a Python subroutine that would take the work done by the kernel out of the box.  

I was just thinking of making a kind of API calling this type of operation scalable on different environments.  
This project being in my famous **ToDoList** would be very close to a Master-Slaving system with certainly an imitation of what already exists with Puppet ... but in Python.  

Wait and see what will be done !
