---
title: No VNC Example 
---

Example used to produce the first basic novnc 
based on conversation had with [NiceGUI contributor](https://github.com/zauberzeug/nicegui/discussions/5929).

``` python
from nicegui import ui
import netifaces
import subprocess
import socket

def main():

    noVncPort='8059'
    vncDisplayNum='1'

    if checkPort(noVncPort) == True:
        subprocess.Popen(['startNoVnc',vncDisplayNum,noVncPort])

    # If native is set 
    # Failed experiment 
    # https://github.com/zauberzeug/nicegui/issues/3815?issue=zauberzeug%7Cnicegui%7C1841
    # But remember to do sudo apt-get install qt5-qmake
    # multiprocessing.set_start_method("spawn", force=True)

    # Example using the public ip address of the rasperry pi viz node 
    # ui.html('<iframe src="http://192.168.1.134:8059/vnc.html" allowfullscreen style="width: 1600px; height: 900px; border: none;"></iframe>', sanitize=False)
    # ui.html('<iframe src="http://192.168.1.134:8059/vnc.html" allowfullscreen frameborder="0" marginheight="0" marginwidth="0" width="100%" height="100%" style="display: block; width: 100%; border: none; overflow-y: auto; overflow-x: auto;"></iframe>', sanitize=False)
    # ui.html('<iframe src="http://192.168.1.134:8059/vnc.html" allowfullscreen style="position:fixed; top:0; left:0; bottom:0; right:0; width:100%; height:100%; border:none; margin:0; padding:0; overflow:hidden;"></iframe>', sanitize=False)
    ui.html('<iframe src="http://' + clusterIp() + ':' + noVncPort + '/vnc.html?resize=remote&quality=7" style="position:fixed; top:0; left:0; bottom:0; right:0; width:100%; height:100%; border:none; margin:0; padding:0; overflow:hidden;" allowfullscreen></iframe>', sanitize=False)
    # print(clusterPublicIp)

    # Example where the user portforwards the NoVnc port on the raspberry pi viz node to loacl host
    # ui.html('<iframe src="http://localhost:8059/vnc.html" style="width: 930px; height: 920px; border: none;"></iframe>', sanitize=False)
    # ui.html('<iframe src="http://localhost:8059/vnc.html?resize=remote&quality=7" style="position:fixed; top:0; left:0; bottom:0; right:0; width:100%; height:100%; border:none; margin:0; padding:0; overflow:hidden;" allowfullscreen></iframe>', sanitize=False)

    ui.run()

def checkPort(port):
    # internalIp = socket.gethostbyname (socket.gethostname())  #getting ip-address of host
    # portFree=True
    # try:
    #     serv = socket.socket(socket.AF_INET,socket.SOCK_STREAM) # create a new socket
    #     serv.bind((internalIp,port)) # bind socket with address
    # except:
    #     print('[OPEN] Port open :',port) #print open port number
    #     portFree = False
    
    portFree = True
    nsPorts = subprocess.check_output(['netstat','-antu'],shell=True).decode('utf8')

    if port in nsPorts:
        print('port ' + port + ' busy!')
        portFree = False
    else:
        print('port ' + port + ' Free!')

    return portFree


def clusterIp():
    clusterPublicIp=''
    for iface in netifaces.interfaces():
        if 'wi-fi' in iface.lower() or 'wlan' in iface.lower():
            addrs = netifaces.ifaddresses(iface).get(netifaces.AF_INET)
            if addrs:
                clusterPublicIp=addrs[0]['addr']
                break
    else:
        print("Wireless IP not found.")

    return clusterPublicIp

main()
```

