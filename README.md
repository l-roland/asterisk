# m4205c-asterisk

| @IP Tel1      | @IP Asterisk  | @IP Tel2 |
| ------------- | ------------- | -------- |
| 192.168.0.127 | 192.168.0.198 | 192.168.0.165 |

## Partie 0 - Installation d'Asterisk

- J'installe Asterisk

```
root@m4205c-asterisk : ~
# apt install asterisk
```

- Je met Asterisk en manuel

```
root@m4205c-asterisk : ~
# vim /etc/default/asterisk

RUNASTERISK=no # pour se mettre en manuel
```

- J'applique les modifications en redémarrant le service

```
root@m4205c-asterisk : ~
# /etc/init.d/asterisk restart
[ ok ] Restarting asterisk (via systemctl): asterisk.service.
```

- Je peux regarder les status

```
root@m4205c-asterisk : ~
# /etc/init.d/asterisk status
● asterisk.service - Asterisk PBX
   Loaded: loaded (/lib/systemd/system/asterisk.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2021-01-22 10:53:54 CET; 8s ago
     Docs: man:asterisk(8)
 Main PID: 1819 (asterisk)
    Tasks: 77 (limit: 2359)
   Memory: 37.4M
   CGroup: /system.slice/asterisk.service
           ├─1819 /usr/sbin/asterisk -g -f -p -U asterisk
           └─1820 astcanary /var/run/asterisk/alt.asterisk.canary.tweet.tweet.tweet 1819

janv. 22 10:53:54 m4205c-asterisk asterisk[1819]: [Jan 22 10:53:54] ERROR[1819]: loader.c:2249 load_modules: cel_radius declined to load.
janv. 22 10:53:54 m4205c-asterisk asterisk[1819]: [Jan 22 10:53:54] ERROR[1819]: loader.c:2249 load_modules: cdr_pgsql declined to load.
janv. 22 10:53:54 m4205c-asterisk asterisk[1819]: [Jan 22 10:53:54] ERROR[1819]: loader.c:2249 load_modules: cdr_sqlite3_custom declined to load.
janv. 22 10:53:54 m4205c-asterisk asterisk[1819]: [Jan 22 10:53:54] ERROR[1819]: loader.c:2249 load_modules: cel_sqlite3_custom declined to load.
janv. 22 10:53:54 m4205c-asterisk asterisk[1819]: [Jan 22 10:53:54] ERROR[1819]: loader.c:2249 load_modules: chan_unistim declined to load.
janv. 22 10:53:54 m4205c-asterisk asterisk[1819]: radcli: rc_read_config: rc_read_config: can't open /etc/radiusclient-ng/radiusclient.conf: No such file or directory
janv. 22 10:53:54 m4205c-asterisk asterisk[1819]: [Jan 22 10:53:54] ERROR[1819]: loader.c:2249 load_modules: pbx_dundi declined to load.
janv. 22 10:53:54 m4205c-asterisk asterisk[1819]: [Jan 22 10:53:54] ERROR[1819]: loader.c:2249 load_modules: res_hep_rtcp declined to load.
janv. 22 10:53:54 m4205c-asterisk asterisk[1819]: [Jan 22 10:53:54] ERROR[1819]: loader.c:2249 load_modules: res_hep_pjsip declined to load.
janv. 22 10:53:54 m4205c-asterisk asterisk[1819]: Asterisk Ready
```

## Partie 1 - Prise en main du logiciel avec article GLMF

### Paramètres de sip.conf

```
[user1]
type=friend
host=dynamic
téléphone
username=user1
secret=toto
disallow=all
allow=ulaw
context=example

[user2]
type=friend
host=dynamic
téléphone
username=user2
secret=toto
disallow=all
allow=ulaw
context=example

[user3]
type=friend
host=dynamic
téléphone
username=user3
secret=toto
disallow=all
allow=ulaw
context=example
```

### Paramètres de extension.conf

```
[example]
exten => 555,1,Dial(SIP/user1)     
exten => 556,1,Dial(SIP/user2)
exten => 557,1,Dial(SIP/user3)
```

### Création d'abonnés

```
root@m4205c-asterisk : ~
# /etc/init.d/asterisk restart
[ ok ] Restarting asterisk (via systemctl): asterisk.service.

root@m4205c-asterisk : ~
# asterisk -r -vvv
Asterisk 16.2.1~dfsg-1+deb10u2, Copyright (C) 1999 - 2018, Digium, Inc. and others.
Created by Mark Spencer <markster@digium.com>
Asterisk comes with ABSOLUTELY NO WARRANTY; type 'core show warranty' for details.
This is free software, with components licensed under the GNU General Public
License version 2 and other licenses; you are welcome to redistribute it under
certain conditions. Type 'core show license' for details.
=========================================================================
Connected to Asterisk 16.2.1~dfsg-1+deb10u2 currently running on m4205c-asterisk (pid = 3086)

    -- Registered SIP 'user1' at 192.168.0.127:5060
    -- Registered SIP 'user2' at 192.168.0.127:5060
    -- Registered SIP 'user3' at 192.168.0.127:5060
```

## Partie 2 - Approfondissement sur sip.conf


### Configuration d'Asterisk
```
root@m4205c-asterisk : ~
# vim /etc/asterisk/sip.conf

[toto]
username=toto
type=friend
secret=toto
host=dynamic
context=usersnum #pour la partie 3
```

```
root@m4205c-asterisk : ~
# /etc/init.d/asterisk restart
[ ok ] Restarting asterisk (via systemctl): asterisk.service.
```

### Configuration de Linphone

- Configuration de Linphone - Client

    - Votre nom d'affichage : test
    - Votre nom d'utilisateur : test
    - Votre @SIP : `"test" <sip:test@192.168.0.127:5060>`

- Configuration de Linphone - Compte Proxy
    
    - Identité SIP : `sip:toto@192.168.0.198:5060`
    - @ Proxy SIP : `sip:toto@192.168.0.198:5060`
    - Je rentre `toto` comme mot de passe
    - Un message apparaît dans le chat asterisk disant que l'user toto est ajouté

```
    -- Registered SIP 'toto' at 192.168.0.127:5060
```

- Vérification

```
m4205c-asterisk*CLI> sip show peers
Name/username             Host                                    Dyn Forcerport Comedia    ACL Port     Status      Description                      
toto/toto                 192.168.0.127                            D  Auto (No)  No             5060     Unmonitored                                  
1 sip peers [Monitored: 0 online, 0 offline Unmonitored: 1 online, 0 offline]
```

- Quand je me déconnecte d'Asterisk

```
    -- Unregistered SIP 'toto'
```

### Appel du numéro 600 (echo) sur Linphone

Je suis censé m'entendre en écho. Voici les logs d'asterisk lors de l'appel

```
== Using SIP RTP CoS mark 5
    -- Executing [600@public:1] playback("SIP/toto-00000001", "demo-echotest")
    -- <SIP/toto-00000001> Playing 'demo-echotest.gsm' (language 'en')
    -- Executing [600@public:1] echo("SIP/toto-00000001", "")
  == Spawn extension (public, 600, 1) exited non-zero on 'SIP/toto-00000001'
```

### Capture de trames

#### Connexion à Asterisk

```
No.     Time           Source                Destination           Protocol Length Info
      6 3.852448540    192.168.0.127         192.168.0.198         SIP      541    Request: REGISTER sip:192.168.0.198:5060  (1 binding) | 

No.     Time           Source                Destination           Protocol Length Info
      7 3.852641681    192.168.0.198         192.168.0.127         SIP      554    Status: 401 Unauthorized | 

No.     Time           Source                Destination           Protocol Length Info
      8 3.883478158    192.168.0.127         192.168.0.198         SIP      708    Request: REGISTER sip:192.168.0.198:5060  (1 binding) | 

No.     Time           Source                Destination           Protocol Length Info
      9 3.883776359    192.168.0.198         192.168.0.127         SIP      568    Status: 200 OK  (1 binding) | 
```

#### Appel avec le numéro 600

```
No.     Time           Source                Destination           Protocol Length Info
     20 6.262690172    192.168.0.127         192.168.0.198         SIP/SDP  1173   Request: INVITE sip:600@192.168.0.198:5060 | 

No.     Time           Source                Destination           Protocol Length Info
     21 6.262935972    192.168.0.198         192.168.0.127         SIP      551    Status: 401 Unauthorized | 

No.     Time           Source                Destination           Protocol Length Info
     22 6.271261378    192.168.0.127         192.168.0.198         SIP      394    Request: ACK sip:600@192.168.0.198:5060 | 

No.     Time           Source                Destination           Protocol Length Info
     23 6.271754873    192.168.0.127         192.168.0.198         SIP/SDP  1344   Request: INVITE sip:600@192.168.0.198:5060 | 

No.     Time           Source                Destination           Protocol Length Info
     24 6.272579704    192.168.0.198         192.168.0.127         SIP      493    Status: 100 Trying | 

No.     Time           Source                Destination           Protocol Length Info
     25 6.294902581    192.168.0.198         192.168.0.127         SIP/SDP  842    Status: 200 OK | 

No.     Time           Source                Destination           Protocol Length Info
     26 6.331368972    192.168.0.127         192.168.0.198         SIP      466    Request: ACK sip:600@192.168.0.198:5060 | 

No.     Time           Source                Destination           Protocol Length Info
     27 6.332977019    192.168.0.127         192.168.0.198         STUN     62     Binding Request

No.     Time           Source                Destination           Protocol Length Info
     28 6.333030839    192.168.0.198         192.168.0.127         STUN     74     Binding Success Response MAPPED-ADDRESS: 192.168.0.127:7078

No.     Time           Source                Destination           Protocol Length Info
     29 6.333049549    192.168.0.198         192.168.0.127         CLASSIC-STUN 66     Message: Binding Request
```
#### Communication vocale
```
No.     Time           Source                Destination           Protocol Length Info
     30 6.385138654    192.168.0.127         192.168.0.198         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0xA600EA5D, Seq=0, Time=800

No.     Time           Source                Destination           Protocol Length Info
     31 6.385158004    192.168.0.127         192.168.0.198         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0xA600EA5D, Seq=1, Time=960

No.     Time           Source                Destination           Protocol Length Info
     32 6.385159684    192.168.0.127         192.168.0.198         STUN     62     Binding Request

No.     Time           Source                Destination           Protocol Length Info
     33 6.385457594    192.168.0.198         192.168.0.127         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0x5B250C5F, Seq=19732, Time=160, Mark

No.     Time           Source                Destination           Protocol Length Info
     34 6.385501754    192.168.0.198         192.168.0.127         STUN     74     Binding Success Response MAPPED-ADDRESS: 192.168.0.127:7078

No.     Time           Source                Destination           Protocol Length Info
     35 6.385516764    192.168.0.198         192.168.0.127         CLASSIC-STUN 66     Message: Binding Request

No.     Time           Source                Destination           Protocol Length Info
     36 6.405877505    192.168.0.198         192.168.0.127         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0x5B250C5F, Seq=19733, Time=320

No.     Time           Source                Destination           Protocol Length Info
     37 6.425568547    192.168.0.198         192.168.0.127         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0x5B250C5F, Seq=19734, Time=480

No.     Time           Source                Destination           Protocol Length Info
     38 6.426843918    192.168.0.127         192.168.0.198         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0xA600EA5D, Seq=2, Time=1120

No.     Time           Source                Destination           Protocol Length Info
     39 6.426853018    192.168.0.127         192.168.0.198         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0xA600EA5D, Seq=3, Time=1280

No.     Time           Source                Destination           Protocol Length Info
     40 6.445931224    192.168.0.198         192.168.0.127         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0x5B250C5F, Seq=19735, Time=640

No.     Time           Source                Destination           Protocol Length Info
     41 6.465882860    192.168.0.198         192.168.0.127         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0x5B250C5F, Seq=19736, Time=800

No.     Time           Source                Destination           Protocol Length Info
     42 6.469970132    192.168.0.127         192.168.0.198         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0xA600EA5D, Seq=4, Time=1440

No.     Time           Source                Destination           Protocol Length Info
   2580 31.611001104   192.168.0.198         192.168.0.127         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0x5B250C5F, Seq=20992, Time=202400

No.     Time           Source                Destination           Protocol Length Info
   2581 31.611041404   192.168.0.198         192.168.0.127         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0x5B250C5F, Seq=20993, Time=202560
```
#### Le client raccroche
```
No.     Time           Source                Destination           Protocol Length Info
   2582 31.620812050   192.168.0.127         192.168.0.198         SIP      512    Request: BYE sip:600@192.168.0.198:5060 | 

No.     Time           Source                Destination           Protocol Length Info
   2583 31.621098580   192.168.0.198         192.168.0.127         SIP      464    Status: 200 OK | 
```
#### Déconnexion d'Asterisk
```
No.     Time           Source                Destination           Protocol Length Info
   2589 37.496743577   192.168.0.127         192.168.0.198         SIP      705    Request: REGISTER sip:192.168.0.198:5060  (remove 1 binding) | 

No.     Time           Source                Destination           Protocol Length Info
   2590 37.496913848   192.168.0.198         192.168.0.127         SIP      554    Status: 401 Unauthorized | 

No.     Time           Source                Destination           Protocol Length Info
   2591 37.505719242   192.168.0.127         192.168.0.198         SIP      705    Request: REGISTER sip:192.168.0.198:5060  (remove 1 binding) | 

No.     Time           Source                Destination           Protocol Length Info
   2592 37.505990483   192.168.0.198         192.168.0.127         SIP      517    Status: 200 OK  (0 bindings) | 
```


## Partie 3 - Approfondissement du extension.conf
- déclaration d'un deuxième postes client "B"

```
root@m4205c-asterisk : ~
# vim /etc/asterisk/sip.conf

[tata]
username=tata
type=friend
secret=tata
host=dynamic
context=usersnum
```

- procéder aux vérifications identiques au §2 pour le poste B

```
    -- Registered SIP 'tata' at 192.168.0.165:5060
  == Using SIP RTP CoS mark 5
    -- Executing [600@public:1] playback("SIP/tata-00000000", "demo-echotest")
    -- <SIP/tata-00000000> Playing 'demo-echotest.gsm' (language 'en')
    -- Executing [600@public:1] echo("SIP/tata-00000000", "")
  == Spawn extension (public, 600, 1) exited non-zero on 'SIP/tata-00000000'
```

```
m4205c-asterisk*CLI> sip show peers
Name/username             Host                                    Dyn Forcerport Comedia    ACL Port     Status      Description                      
tata/tata                 192.168.0.165                            D  Auto (No)  No             5060     Unmonitored                                  
toto/toto                 192.168.0.127                            D  Auto (No)  No             5060     Unmonitored                                  
2 sip peers [Monitored: 0 online, 0 offline Unmonitored: 2 online, 0 offline]
```

- modification de extension.conf pour permettre les appels entre ces deux postes, sous les numéros A=701 et B=702

```
root@m4205c-asterisk : ~
# vim /etc/asterisk/extensions.conf

[usersnum]
exten => 701,1,Dial(SIP/toto)
exten => 702,1,Dial(SIP/tata)
```

- réduction au minimum nécessaire des deux fichiers (sip.conf et extension.conf), renommage stratégique des postes A et B pour utiliser la variable ${EXTEN} pour les appels et forçage du seul codec Alaw.

    - Contenu de /etc/asterisk/extensions.conf

    ```
    [usersnum]
    exten => _70X,1,Dial(SIP/${EXTEN})
    ```

    - Contenu de /etc/asterisk/sip.conf
    ```
	[general]
	context=public
	allowoverlap=no
	udpbindaddr=0.0.0.0
	tcpenable=no
	tcpbindaddr=0.0.0.0
	transport=udp
	srvlookup=yes
	
	[701]
	username=701
	type=friend
	secret=toto
	host=dynamic
	context=usersnum
	allow=!all,ulaw
	
	[702]
	username=702
	type=friend
	secret=tata
	host=dynamic
	context=usersnum
	allow=!all,ulaw
	```




## Partie 4 : Étude des flux
- A appelle B, A raccroche
- A appelle B, B raccroche
- Prise en compte des paramètres "reinvite" et "directmedia" pour le test des différents scénarios, avec capture des flux.

### Sans aucun paramètre

- A appelle B, A raccroche
    
```
No.     Time           Source                Destination           Protocol Length Info
      3 2.653616978    192.168.0.127         192.168.0.198         SIP/SDP  1165   Request: INVITE sip:702@192.168.0.198 | 

No.     Time           Source                Destination           Protocol Length Info
      4 2.653807098    192.168.0.198         192.168.0.127         SIP      550    Status: 401 Unauthorized | 

No.     Time           Source                Destination           Protocol Length Info
      5 2.666947621    192.168.0.127         192.168.0.198         SIP      387    Request: ACK sip:702@192.168.0.198 | 

No.     Time           Source                Destination           Protocol Length Info
      6 2.667169441    192.168.0.127         192.168.0.198         SIP/SDP  1330   Request: INVITE sip:702@192.168.0.198 | 

No.     Time           Source                Destination           Protocol Length Info
      7 2.667643660    192.168.0.198         192.168.0.127         SIP      492    Status: 100 Trying |

No.     Time           Source                Destination           Protocol Length Info
      8 2.668192240    192.168.0.198         192.168.0.165         SIP/SDP  867    Request: INVITE sip:702@192.168.0.165 | 

No.     Time           Source                Destination           Protocol Length Info
      9 2.815169509    192.168.0.165         192.168.0.198         SIP      277    Status: 100 Trying | 

No.     Time           Source                Destination           Protocol Length Info
     10 3.020113741    192.168.0.165         192.168.0.198         SIP      359    Status: 180 Ringing | 

No.     Time           Source                Destination           Protocol Length Info
     11 3.020309541    192.168.0.198         192.168.0.127         SIP      508    Status: 180 Ringing | 

No.     Time           Source                Destination           Protocol Length Info
     13 4.027145465    192.168.0.165         192.168.0.198         SIP/SDP  739    Status: 200 Ok | 

No.     Time           Source                Destination           Protocol Length Info
     14 4.027422034    192.168.0.198         192.168.0.165         SIP      427    Request: ACK sip:702@192.168.0.165 | 

No.     Time           Source                Destination           Protocol Length Info
     15 4.027641204    192.168.0.198         192.168.0.127         SIP/SDP  815    Status: 200 OK | 

No.     Time           Source                Destination           Protocol Length Info
     19 4.055652677    192.168.0.127         192.168.0.198         SIP      459    Request: ACK sip:702@192.168.0.198:5060 |

No.     Time           Source                Destination           Protocol Length Info
    326 5.701969557    192.168.0.127         192.168.0.198         SIP      510    Request: BYE sip:702@192.168.0.198:5060 | 

No.     Time           Source                Destination           Protocol Length Info
    327 5.702293996    192.168.0.198         192.168.0.127         SIP      463    Status: 200 OK | 

No.     Time           Source                Destination           Protocol Length Info
    328 5.703105116    192.168.0.198         192.168.0.165         SIP      461    Request: BYE sip:702@192.168.0.165 | 

No.     Time           Source                Destination           Protocol Length Info
    329 5.716565060    192.168.0.165         192.168.0.198         SIP      351    Status: 200 Ok | 
```
    
- A appelle B, B raccroche
```
No.     Time           Source                Destination           Protocol Length Info
      1 0.000000000    192.168.0.127         192.168.0.198         SIP/SDP  1163   Request: INVITE sip:702@192.168.0.198 | 

No.     Time           Source                Destination           Protocol Length Info
      2 0.000255230    192.168.0.198         192.168.0.127         SIP      550    Status: 401 Unauthorized | 

No.     Time           Source                Destination           Protocol Length Info
      3 0.012196548    192.168.0.127         192.168.0.198         SIP      387    Request: ACK sip:702@192.168.0.198 | 

No.     Time           Source                Destination           Protocol Length Info
      4 0.012421439    192.168.0.127         192.168.0.198         SIP/SDP  1328   Request: INVITE sip:702@192.168.0.198 | 

No.     Time           Source                Destination           Protocol Length Info
      5 0.012891668    192.168.0.198         192.168.0.127         SIP      492    Status: 100 Trying | 

No.     Time           Source                Destination           Protocol Length Info
      6 0.013362928    192.168.0.198         192.168.0.165         SIP/SDP  865    Request: INVITE sip:702@192.168.0.165 | 

No.     Time           Source                Destination           Protocol Length Info
      7 0.052378361    192.168.0.165         192.168.0.198         SIP      277    Status: 100 Trying | 

No.     Time           Source                Destination           Protocol Length Info
      8 0.230784920    192.168.0.165         192.168.0.198         SIP      359    Status: 180 Ringing | 

No.     Time           Source                Destination           Protocol Length Info
      9 0.230966249    192.168.0.198         192.168.0.127         SIP      508    Status: 180 Ringing | 

No.     Time           Source                Destination           Protocol Length Info
     10 1.164321157    192.168.0.165         192.168.0.198         SIP/SDP  739    Status: 200 Ok | 

No.     Time           Source                Destination           Protocol Length Info
     11 1.164529037    192.168.0.198         192.168.0.165         SIP      427    Request: ACK sip:702@192.168.0.165 | 

No.     Time           Source                Destination           Protocol Length Info
     12 1.164729017    192.168.0.198         192.168.0.127         SIP/SDP  817    Status: 200 OK | 

No.     Time           Source                Destination           Protocol Length Info
     16 1.189633675    192.168.0.127         192.168.0.198         SIP      459    Request: ACK sip:702@192.168.0.198:5060 | 

No.     Time           Source                Destination           Protocol Length Info
    232 2.334467447    192.168.0.165         192.168.0.198         SIP      380    Request: BYE sip:701@192.168.0.198:5060 | 

No.     Time           Source                Destination           Protocol Length Info
    233 2.334715007    192.168.0.198         192.168.0.165         SIP      503    Status: 200 OK | 

No.     Time           Source                Destination           Protocol Length Info
    234 2.335166186    192.168.0.198         192.168.0.127         SIP      591    Request: BYE sip:701@192.168.0.127 | 

No.     Time           Source                Destination           Protocol Length Info
    235 2.356466317    192.168.0.127         192.168.0.198         SIP      318    Status: 200 Ok | 
```


### Avec canreinvite=yes

- A appelle B, A raccroche
```
No.     Time           Source                Destination           Protocol Length Info
      2 2.528127116    192.168.0.127         192.168.0.198         SIP/SDP  1165   Request: INVITE sip:702@192.168.0.198 |

No.     Time           Source                Destination           Protocol Length Info
      3 2.528336056    192.168.0.198         192.168.0.127         SIP      550    Status: 401 Unauthorized | 

No.     Time           Source                Destination           Protocol Length Info
      4 2.559549171    192.168.0.127         192.168.0.198         SIP      387    Request: ACK sip:702@192.168.0.198 | 

No.     Time           Source                Destination           Protocol Length Info
      5 2.559789981    192.168.0.127         192.168.0.198         SIP/SDP  1330   Request: INVITE sip:702@192.168.0.198 | 

No.     Time           Source                Destination           Protocol Length Info
      6 2.560253120    192.168.0.198         192.168.0.127         SIP      492    Status: 100 Trying |

No.     Time           Source                Destination           Protocol Length Info
      7 2.560762710    192.168.0.198         192.168.0.165         SIP/SDP  865    Request: INVITE sip:702@192.168.0.165 | 

No.     Time           Source                Destination           Protocol Length Info
      8 2.689620242    192.168.0.165         192.168.0.198         SIP      277    Status: 100 Trying | 

No.     Time           Source                Destination           Protocol Length Info
     10 2.957361412    192.168.0.165         192.168.0.198         SIP      359    Status: 180 Ringing | 

No.     Time           Source                Destination           Protocol Length Info
     11 2.957577712    192.168.0.198         192.168.0.127         SIP      508    Status: 180 Ringing | 

No.     Time           Source                Destination           Protocol Length Info
     12 3.630700666    192.168.0.165         192.168.0.198         SIP/SDP  740    Status: 200 Ok | 

No.     Time           Source                Destination           Protocol Length Info
     13 3.630920095    192.168.0.198         192.168.0.165         SIP      427    Request: ACK sip:702@192.168.0.165 | 

No.     Time           Source                Destination           Protocol Length Info
     14 3.631101886    192.168.0.198         192.168.0.127         SIP/SDP  815    Status: 200 OK | 

No.     Time           Source                Destination           Protocol Length Info
     15 3.631449945    192.168.0.198         192.168.0.165         SIP/SDP  839    Request: INVITE sip:702@192.168.0.165, in-dialog | 

No.     Time           Source                Destination           Protocol Length Info
     19 3.647206139    192.168.0.127         192.168.0.198         SIP      459    Request: ACK sip:702@192.168.0.198:5060 | 

No.     Time           Source                Destination           Protocol Length Info
     20 3.647358279    192.168.0.198         192.168.0.127         SIP/SDP  804    Request: INVITE sip:701@192.168.0.127, in-dialog | 

No.     Time           Source                Destination           Protocol Length Info
     24 3.692322258    192.168.0.127         192.168.0.198         SIP      258    Status: 100 Trying | 

No.     Time           Source                Destination           Protocol Length Info
     25 3.693442267    192.168.0.127         192.168.0.198         SIP/SDP  707    Status: 200 Ok | 

No.     Time           Source                Destination           Protocol Length Info
     26 3.693711117    192.168.0.198         192.168.0.127         SIP      392    Request: ACK sip:701@192.168.0.127 |

No.     Time           Source                Destination           Protocol Length Info
     70 4.051834525    192.168.0.165         192.168.0.198         SIP      291    Status: 100 Trying | 

No.     Time           Source                Destination           Protocol Length Info
     71 4.053587884    192.168.0.165         192.168.0.198         SIP/SDP  740    Status: 200 Ok | 

No.     Time           Source                Destination           Protocol Length Info
     72 4.053716154    192.168.0.198         192.168.0.165         SIP      427    Request: ACK sip:702@192.168.0.165 |

No.     Time           Source                Destination           Protocol Length Info
     73 5.301566489    192.168.0.127         192.168.0.198         SIP      5
```
    
- A appelle B, B raccroche
```
No.     Time           Source                Destination           Protocol Length Info
      1 0.000000000    192.168.0.127         192.168.0.198         SIP/SDP  1164   Request: INVITE sip:702@192.168.0.198 |

No.     Time           Source                Destination           Protocol Length Info
      2 0.000208780    192.168.0.198         192.168.0.127         SIP      550    Status: 401 Unauthorized |

No.     Time           Source                Destination           Protocol Length Info
      3 0.030062742    192.168.0.127         192.168.0.198         SIP      387    Request: ACK sip:702@192.168.0.198 |

No.     Time           Source                Destination           Protocol Length Info
      4 0.030272732    192.168.0.127         192.168.0.198         SIP/SDP  1329   Request: INVITE sip:702@192.168.0.198 |

No.     Time           Source                Destination           Protocol Length Info
      5 0.030794652    192.168.0.198         192.168.0.127         SIP      492    Status: 100 Trying |

No.     Time           Source                Destination           Protocol Length Info
      6 0.031325861    192.168.0.198         192.168.0.165         SIP/SDP  867    Request: INVITE sip:702@192.168.0.165 |

No.     Time           Source                Destination           Protocol Length Info
      7 0.074833646    192.168.0.165         192.168.0.198         SIP      277    Status: 100 Trying |

No.     Time           Source                Destination           Protocol Length Info
      8 0.247361678    192.168.0.165         192.168.0.198         SIP      359    Status: 180 Ringing |

No.     Time           Source                Destination           Protocol Length Info
      9 0.247557628    192.168.0.198         192.168.0.127         SIP      508    Status: 180 Ringing |

No.     Time           Source                Destination           Protocol Length Info
     11 1.967442968    192.168.0.165         192.168.0.198         SIP/SDP  740    Status: 200 Ok |

No.     Time           Source                Destination           Protocol Length Info
     12 1.967649608    192.168.0.198         192.168.0.165         SIP      427    Request: ACK sip:702@192.168.0.165 |

No.     Time           Source                Destination           Protocol Length Info
     13 1.967834108    192.168.0.198         192.168.0.127         SIP/SDP  815    Status: 200 OK |

No.     Time           Source                Destination           Protocol Length Info
     14 1.968169978    192.168.0.198         192.168.0.165         SIP/SDP  841    Request: INVITE sip:702@192.168.0.165, in-dialog |

No.     Time           Source                Destination           Protocol Length Info
     18 1.989225414    192.168.0.127         192.168.0.198         SIP      459    Request: ACK sip:702@192.168.0.198:5060 |

No.     Time           Source                Destination           Protocol Length Info
     19 1.989379074    192.168.0.198         192.168.0.127         SIP/SDP  804    Request: INVITE sip:701@192.168.0.127, in-dialog |

No.     Time           Source                Destination           Protocol Length Info
     23 2.045017569    192.168.0.127         192.168.0.198         SIP      258    Status: 100 Trying |

No.     Time           Source                Destination           Protocol Length Info
     24 2.046063259    192.168.0.127         192.168.0.198         SIP/SDP  706    Status: 200 Ok | 

No.     Time           Source                Destination           Protocol Length Info
     25 2.046219148    192.168.0.198         192.168.0.127         SIP      392    Request: ACK sip:701@192.168.0.127 |

No.     Time           Source                Destination           Protocol Length Info
     49 2.260650196    192.168.0.165         192.168.0.198         SIP      291    Status: 100 Trying |

No.     Time           Source                Destination           Protocol Length Info
     50 2.261997035    192.168.0.165         192.168.0.198         SIP/SDP  740    Status: 200 Ok | 

No.     Time           Source                Destination           Protocol Length Info
     51 2.262139405    192.168.0.198         192.168.0.165         SIP      427    Request: ACK sip:702@192.168.0.165 | 

No.     Time           Source                Destination           Protocol Length Info
     53 3.632838044    192.168.0.165         192.168.0.198         SIP      380    Request: BYE sip:701@192.168.0.198:5060 | 

No.     Time           Source                Destination           Protocol Length Info
     54 3.633108774    192.168.0.198         192.168.0.165         SIP      503    Status: 200 OK | 

No.     Time           Source                Destination           Protocol Length Info
     55 3.633344764    192.168.0.198         192.168.0.127         SIP/SDP  805    Request: INVITE sip:701@192.168.0.127, in-dialog | 

No.     Time           Source                Destination           Protocol Length Info
     56 3.644656419    192.168.0.127         192.168.0.198         SIP      258    Status: 100 Trying | 

No.     Time           Source                Destination           Protocol Length Info
     57 3.645874784    192.168.0.127         192.168.0.198         SIP/SDP  706    Status: 200 Ok | 

No.     Time           Source                Destination           Protocol Length Info
     58 3.646039364    192.168.0.198         192.168.0.127         SIP      392    Request: ACK sip:701@192.168.0.127 |

No.     Time           Source                Destination           Protocol Length Info
     59 3.646077464    192.168.0.198         192.168.0.127         SIP      591    Request: BYE sip:701@192.168.0.127 |

No.     Time           Source                Destination           Protocol Length Info
     64 3.704753661    192.168.0.127         192.168.0.198         SIP      318    Status: 200 Ok | 
```


### Avec directrtpsetup=yes et canreinvite=yes

- A appelle B, A raccroche
```
No.     Time           Source                Destination           Protocol Length Info
      8 1.978623685    192.168.0.127         192.168.0.198         SIP/SDP  1163   Request: INVITE sip:702@192.168.0.198 | 

No.     Time           Source                Destination           Protocol Length Info
      9 1.978827455    192.168.0.198         192.168.0.127         SIP      550    Status: 401 Unauthorized | 

No.     Time           Source                Destination           Protocol Length Info
     10 1.990100629    192.168.0.127         192.168.0.198         SIP      387    Request: ACK sip:702@192.168.0.198 |

No.     Time           Source                Destination           Protocol Length Info
     11 1.990348319    192.168.0.127         192.168.0.198         SIP/SDP  1328   Request: INVITE sip:702@192.168.0.198 | 

No.     Time           Source                Destination           Protocol Length Info
     12 1.990816739    192.168.0.198         192.168.0.127         SIP      492    Status: 100 Trying | 

No.     Time           Source                Destination           Protocol Length Info
     13 1.991445309    192.168.0.198         192.168.0.165         SIP/SDP  866    Request: INVITE sip:702@192.168.0.165 |

No.     Time           Source                Destination           Protocol Length Info
     14 2.024961773    192.168.0.165         192.168.0.198         SIP      277    Status: 100 Trying |

No.     Time           Source                Destination           Protocol Length Info
     15 2.142980400    192.168.0.165         192.168.0.198         SIP      359    Status: 180 Ringing |

No.     Time           Source                Destination           Protocol Length Info
     16 2.143182930    192.168.0.198         192.168.0.127         SIP      508    Status: 180 Ringing |

No.     Time           Source                Destination           Protocol Length Info
     17 2.935327051    192.168.0.165         192.168.0.198         SIP/SDP  740    Status: 200 Ok |

No.     Time           Source                Destination           Protocol Length Info
     18 2.935545401    192.168.0.198         192.168.0.165         SIP      427    Request: ACK sip:702@192.168.0.165 |

No.     Time           Source                Destination           Protocol Length Info
     19 2.935736441    192.168.0.198         192.168.0.127         SIP/SDP  814    Status: 200 OK | 

No.     Time           Source                Destination           Protocol Length Info
     20 2.956543391    192.168.0.127         192.168.0.198         SIP      459    Request: ACK sip:702@192.168.0.198:5060 |                                 ....

No.     Time           Source                Destination           Protocol Length Info
     22 3.849802167    192.168.0.127         192.168.0.198         SIP      510    Request: BYE sip:702@192.168.0.198:5060 |

No.     Time           Source                Destination           Protocol Length Info
     23 3.849991816    192.168.0.198         192.168.0.127         SIP      463    Status: 200 OK |

No.     Time           Source                Destination           Protocol Length Info
     24 3.850184086    192.168.0.198         192.168.0.165         SIP/SDP  842    Request: INVITE sip:702@192.168.0.165, in-dialog |

No.     Time           Source                Destination           Protocol Length Info
     25 3.883771341    192.168.0.165         192.168.0.198         SIP      291    Status: 100 Trying |

No.     Time           Source                Destination           Protocol Length Info
     26 3.886227400    192.168.0.165         192.168.0.198         SIP/SDP  740    Status: 200 Ok |

No.     Time           Source                Destination           Protocol Length Info
     27 3.886347740    192.168.0.198         192.168.0.165         SIP      427    Request: ACK sip:702@192.168.0.165 |

No.     Time           Source                Destination           Protocol Length Info
     28 3.886377040    192.168.0.198         192.168.0.165         SIP      461    Request: BYE sip:702@192.168.0.165 |

No.     Time           Source                Destination           Protocol Length Info
     29 3.917252186    192.168.0.165         192.168.0.198         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0xD47BE8D1, Seq=40, Time=7440

No.     Time           Source                Destination           Protocol Length Info
     31 3.941177475    192.168.0.165         192.168.0.198         SIP      351    Status: 200 Ok | 

```
    
- A appelle B, B raccroche
```
No.     Time           Source                Destination           Protocol Length Info
      5 1.808373691    192.168.0.127         192.168.0.198         SIP/SDP  1165   Request: INVITE sip:702@192.168.0.198 |

No.     Time           Source                Destination           Protocol Length Info
      6 1.808565020    192.168.0.198         192.168.0.127         SIP      550    Status: 401 Unauthorized |

No.     Time           Source                Destination           Protocol Length Info
      7 1.819199165    192.168.0.127         192.168.0.198         SIP      387    Request: ACK sip:702@192.168.0.198 |

No.     Time           Source                Destination           Protocol Length Info
      8 1.819443265    192.168.0.127         192.168.0.198         SIP/SDP  1330   Request: INVITE sip:702@192.168.0.198 |

No.     Time           Source                Destination           Protocol Length Info
      9 1.819907164    192.168.0.198         192.168.0.127         SIP      492    Status: 100 Trying |

No.     Time           Source                Destination           Protocol Length Info
     10 1.820466224    192.168.0.198         192.168.0.165         SIP/SDP  866    Request: INVITE sip:702@192.168.0.165 |

No.     Time           Source                Destination           Protocol Length Info
     11 1.850605301    192.168.0.165         192.168.0.198         SIP      277    Status: 100 Trying |                                     ....

No.     Time           Source                Destination           Protocol Length Info
     13 1.982537631    192.168.0.165         192.168.0.198         SIP      359    Status: 180 Ringing |

No.     Time           Source                Destination           Protocol Length Info
     14 1.982762471    192.168.0.198         192.168.0.127         SIP      508    Status: 180 Ringing |

No.     Time           Source                Destination           Protocol Length Info
     16 2.999828830    192.168.0.165         192.168.0.198         SIP/SDP  739    Status: 200 Ok |

No.     Time           Source                Destination           Protocol Length Info
     17 2.999998550    192.168.0.198         192.168.0.165         SIP      427    Request: ACK sip:702@192.168.0.165 |

No.     Time           Source                Destination           Protocol Length Info
     18 3.000176650    192.168.0.198         192.168.0.127         SIP/SDP  816    Status: 200 OK |

No.     Time           Source                Destination           Protocol Length Info
     19 3.030049156    192.168.0.127         192.168.0.198         SIP      459    Request: ACK sip:702@192.168.0.198:5060 |

No.     Time           Source                Destination           Protocol Length Info
     20 3.989999992    192.168.0.165         192.168.0.198         SIP      380    Request: BYE sip:701@192.168.0.198:5060 |

No.     Time           Source                Destination           Protocol Length Info
     21 3.990181331    192.168.0.198         192.168.0.165         SIP      503    Status: 200 OK |

No.     Time           Source                Destination           Protocol Length Info
     22 3.990366881    192.168.0.198         192.168.0.127         SIP/SDP  807    Request: INVITE sip:701@192.168.0.127, in-dialog |

No.     Time           Source                Destination           Protocol Length Info
     23 4.016236809    192.168.0.127         192.168.0.198         SIP      258    Status: 100 Trying |

No.     Time           Source                Destination           Protocol Length Info
     24 4.017658819    192.168.0.127         192.168.0.198         SIP/SDP  707    Status: 200 Ok |

No.     Time           Source                Destination           Protocol Length Info
     25 4.017777079    192.168.0.198         192.168.0.127         SIP      392    Request: ACK sip:701@192.168.0.127 |

No.     Time           Source                Destination           Protocol Length Info
     26 4.017813329    192.168.0.198         192.168.0.127         SIP      591    Request: BYE sip:701@192.168.0.127 |

No.     Time           Source                Destination           Protocol Length Info
     27 4.020930578    192.168.0.127         192.168.0.198         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0x2BE47149, Seq=46, Time=17520

No.     Time           Source                Destination           Protocol Length Info
     28 4.020940187    192.168.0.127         192.168.0.198         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0x2BE47149, Seq=47, Time=17680

No.     Time           Source                Destination           Protocol Length Info
     29 4.063936068    192.168.0.127         192.168.0.198         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0x2BE47149, Seq=48, Time=17840

No.     Time           Source                Destination           Protocol Length Info
     30 4.063948628    192.168.0.127         192.168.0.198         RTP      214    PT=ITU-T G.711 PCMU, SSRC=0x2BE47149, Seq=49, Time=18000

No.     Time           Source                Destination           Protocol Length Info
     31 4.077281323    192.168.0.127         192.168.0.198         SIP      318    Status: 200 Ok | 
```

A partir de là, on remet à zéro le contenu de extension.conf qui ne contiendra que les nouvelles fonctionnalités, en plus de l'appel des postes entre eux.

```
root@m4205c-asterisk : ~
# echo "" > /etc/asterisk/extensions.conf
```

## Partie 5 - Mettre en place la fonction "echo" au numéro 600
- Quand il appelle le correspondant doit s'entendre.
- Prendre exemple sur le fichiers extensions.conf d'origine

```
root@m4205c-asterisk : ~
# vim /etc/asterisk/extensions.conf

[usersnum]
exten => 600,1,Playback(demo-echotest)  ; Let them know what's going on
exten => 600,n,Echo()                   ; Do the echo test
exten => 600,n,Playback(demo-echodone)  ; Let them know it's over
exten => 600,n,Goto(s,6)                ; Start over

root@m4205c-asterisk : ~
# /etc/init.d/asterisk restart
[ ok ] Restarting asterisk (via systemctl): asterisk.service.
```

https://github.com/asterisk/asterisk/tree/master/configs/samples


## Partie 6 - Mettre en place une horloge parlante au numéro 611
- Elle doit donner la date et l'heure à la seconde près
- Elle doit parler en français

```
exten => 611,1,Answer()
exten => 611,2,SayUnixTime(,Europe/Paris,AdBY T)
exten => 611,3,HangUp()
```

```
root@m4205c-asterisk : ~
# apt install asterisk-core-sounds-fr
```

```
root@m4205c-asterisk : ~
# vim /etc/asterisk/asterisk.conf 

[options]
...
defaultlanguage = fr            ; Default language
documentation_language = fr_FR  ; Set the language you want documentation
                                ; displayed in. Value is in the same format as
                                ; locale names.
```

```
root@m4205c-asterisk : ~
# vim /etc/asterisk/sip.con

[general]
...
language=fr
```

```
root@m4205c-asterisk : ~
# /etc/init.d/asterisk restart
[ ok ] Restarting asterisk (via systemctl): asterisk.service.
```

https://siocours.lycees.nouvelle-aquitaine.pro/doku.php/reseau/asterisk/configserveur/installasterisk

## Partie 7 - Mettre en place la lecture de la météo au numéro 622
- Utiliser festival ou espeak pour la synthèse vocale
- Utiliser un site web comme source de la météo

### Utilisation de metar

```
root@m4205c-asterisk : ~
# wget http://ftp.br.debian.org/debian/pool/main/m/metar/metar_20190227.1-1+b1_amd64.deb
```

```
root@m4205c-asterisk : ~
# dpkg -i metar_20190227.1-1+b1_amd64.deb
```

```
root@m4205c-asterisk : ~
# metar -d LFMU
LFMU 101400Z AUTO 34012KT 310V010 9999 SCT042 BKN048 BKN056 13/07 Q1006 TEMPO 30015G25KT
Station       : LFMU
Day           : 10
Time          : 14:00 UTC
Wind direction: 340 (NNW)
Wind speed    : 12 KT
Wind gust     : 12 KT
Visibility    : 9999 M
Temperature   : 13 C
Dewpoint      : 7 C
Pressure      : 1006 hPa
Clouds        : SCT at 4200 ft
                BKN at 4800 ft
                BKN at 5600 ft
Phenomena     :
```

### Script metar.sh afin d'extraire la température et la pression

```
#!/bin/bash
metar=`metar -d LFMU`
echo "$metar"
echo "--------------------------------------------"
metar -d LFMU | grep Temperature | awk '{print $3}' > /home/degre.txt
metar -d LFMU | grep Pressure | awk '{print $3}' > /home/pression.txt
```

```
root@m4205c-asterisk : ~
# sh /home/metar.sh && cat /home/degre.txt /home/pression.txt

LFMU 121930Z AUTO VRB03KT 3400 -DZ BR OVC008 07/07 Q1016 TEMPO BKN010 FEW030TCU
Station       : LFMU
Day           : 12
Time          : 19:30 UTC
Wind direction: Variable
Wind speed    : 3 KT
Wind gust     : 3 KT
Visibility    : 3400 M
Temperature   : 7 C
Dewpoint      : 7 C
Pressure      : 1016 hPa
Clouds        : OVC at 800 ft
                BKN at 1000 ft
Phenomena     : Light Drizzle
                Mist
--------------------------------------------
7
1016
```

```
root@m4205c-asterisk : /home
# chmod 0777 /home/degre.txt /home/pression.txt
```



### Numéro 622 avec Espeak

```
root@m4205c-asterisk : ~
# apt install espeak asterisk-espeak
```


```
exten => 622,1,Answer()
exten => 622,n,System(/home/metar.sh)
exten => 622,n,Set(METEO=${FILE(/home/degre.txt)})
exten => 622,n,Set(PRESSION=${FILE(/home/pression.txt)})
exten => 622,n,Espeak("Température ${METEO} degrés",any,fr)
exten => 622,n,Espeak("Pression ${PRESSION} hectopascal",any,fr)
```

```
  == Using SIP RTP CoS mark 5
    -- Executing [622@usersnum:1] Answer("SIP/701-00000002", "") in new stack
    -- Executing [622@usersnum:2] System("SIP/701-00000002", "/home/metar.sh") in new stack
    -- Executing [622@usersnum:3] Set("SIP/701-00000002", "METEO=7
    -- ") in new stack
    -- Executing [622@usersnum:4] Set("SIP/701-00000002", "PRESSION=1016
    -- ") in new stack
    -- Executing [622@usersnum:5] eSpeak("SIP/701-00000002", ""Température 7
    --  degrés",any,fr") in new stack
    -- <SIP/701-00000002> Playing '/tmp/espk_7JKqvb.slin' (language 'fr')
    -- Executing [622@usersnum:6] eSpeak("SIP/701-00000002", ""Pression 1016
    --  hectopascal",any,fr") in new stack
    -- <SIP/701-00000002> Playing '/tmp/espk_8OWNyH.slin' (language 'fr')
    -- Auto fallthrough, channel 'SIP/701-00000002' status is 'UNKNOWN'
```

### Numéro 623 avec MP3Player et Espeak

- Température
-- Record Audacity : "Température"
-- Synthèse vocal Espeak : "valeur"
-- Record Audacity : "degrés"

- Pression
-- Record Audacity : "Pression"
-- Synthèse vocal Espeak : "valeur"
-- Record Audacity : "hectopascal"

```
exten => 623,1,Answer()
exten => 623,n,System(/home/metar.sh)
exten => 623,n,Set(METEO=${FILE(/home/degre.txt)})
exten => 623,n,Set(PRESSION=${FILE(/home/pression.txt)})
exten => 623,n,MP3Player(/home/temp.mp3)
exten => 623,n,Espeak("${METEO}",any,fr)
exten => 623,n,MP3Player(/home/temp2.mp3)
exten => 623,n,MP3Player(/home/press1.mp3)
exten => 623,n,Espeak("${PRESSION}",any,fr)
exten => 623,n,MP3Player(/home/press2.mp3)
```

```
  == Using SIP RTP CoS mark 5
    -- Executing [623@usersnum:1] Answer("SIP/701-00000001", "") in new stack
    -- Executing [623@usersnum:2] System("SIP/701-00000001", "/home/metar.sh") in new stack
    -- Executing [623@usersnum:3] Set("SIP/701-00000001", "METEO=7
    -- ") in new stack
    -- Executing [623@usersnum:4] Set("SIP/701-00000001", "PRESSION=1016
    -- ") in new stack
    -- Executing [623@usersnum:5] MP3Player("SIP/701-00000001", "/home/temp.mp3") in new stack
    -- Executing [623@usersnum:6] eSpeak("SIP/701-00000001", ""7
    -- ",any,fr") in new stack
    -- <SIP/701-00000001> Playing '/tmp/espk_5ZJeKG.slin' (language 'fr')
    -- Executing [623@usersnum:7] MP3Player("SIP/701-00000001", "/home/temp2.mp3") in new stack
    -- Executing [623@usersnum:8] MP3Player("SIP/701-00000001", "/home/press1.mp3") in new stack
    -- Executing [623@usersnum:9] eSpeak("SIP/701-00000001", ""1016
    -- ",any,fr") in new stack
    -- <SIP/701-00000001> Playing '/tmp/espk_sE3tAL.slin' (language 'fr')
    -- Executing [623@usersnum:10] MP3Player("SIP/701-00000001", "/home/press2.mp3") in new stack
    -- Auto fallthrough, channel 'SIP/701-00000001' status is 'UNKNOWN'

```


## Partie 8 - Jeu "trouver un nombre entre 1 et 99" au numéro 633
- Utiliser les fonctions IVR d'asterisk
- vous enregistrerez vous même les messages vocaux nécéssaires ou vous utiliserez la synthèse vocale pour guider les joueurs

## Partie 9 - Écouter France Culture au numéro 644
- Le flux radio est disponible sur internet, mais c'est pas simple. Il va vous falloir espionner votre navigateur web pour trouver la "vraie" source.

```
root@m4205c-asterisk : ~
# apt install mpg123
```

```
exten => 644,1,Answer();
exten => 644,n,MP3Player(http://direct.franceculture.fr/live/franceculture-midfi.mp3)
```

```
  == Using SIP RTP CoS mark 5
    -- Executing [644@usersnum:1] Answer("SIP/701-00000003", "") in new stack
    -- Executing [644@usersnum:2] MP3Player("SIP/701-00000003", "http://direct.franceculture.fr/live/franceculture-midfi.mp3") in new stack
  == Spawn extension (usersnum, 644, 2) exited non-zero on 'SIP/701-00000003'
```

À partir de là, il faut être en mesure de relier son IPBX à un opérateur via un trunk IAX:

TO BE DONE !

X : Trunk IAX

Y : transfert d'appel appeler avec le poste A le service radio chez l'opérateur, et transférer la communication vers le poste B

Z : idem, mais en rajoutant une MOH (France Culture) 
