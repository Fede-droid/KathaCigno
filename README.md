# How to implement IPtables through Katharà
the steps that you will find in this guide are the following:

1. introduction

2. my network scheme

3. basic device configuration

4. introduction to firewalls and iptables

5. iptables implementation

6. test
## Introduction
Ciao a tutti sono Federico, un oramai dottore della triennale di ingegneria informatica dell'Università degli studi di Pavia.

In questa pagina mostrerò a voi come configurare i device di rete, facendo riferimento alla rete che io ho sviluppato, come implementare le IPtables tramite l'utilizzo di Katharà e come gestire le richieste di accesso alla pagina web di vari server tramite un metodo dinamico. Iniziamo!
## My network scheme
![image](https://github.com/Fede-droid/KathaCigno/assets/80335626/2ea1c5ef-84ec-454a-a6d4-f2728fe114eb)
Qui potete vedere 2 reti pubbliche evidenziate in azzurro ed una rete privata evidenziata in verde.
Queste reti sono evidenziate da una lettera ed articolate da vari device quali:
- client, router e server.
Ogni dispositivo ha affiancato 2 celle con all'interno lettere e numeri.
- la superiore indica l'indirizz ip finale del device;
- l'inferiore la scheda di rete associata al device;

es. prendiamo in considerazione la "public network 2", osserviamo il "client1"; Quest'ultimo ha come indirizzo ip  11.0.0.7.
Stesso discorso per tutti gli altri device della rete. In particolare il router che avra 2 schede di rete e quindi 2 indirizzi ip, visto che nel mio caso unisce 2 domini differenti.
## Basic device configuration
### primo step
Creiamo il file "lab.conf" ed andiamo ad inserire al suo interno tutti i device della rete associati alla scheda di rete ed al dominio a cui sono connessi, in questo modo:
- client1[eth0]=c
Facciamo lo stesso discorso per tutti gli altri device.
### secondo step
Per ogni device andiamo a creare un file di configurazione che denominiamo: 
- [device].startup
ed al suo interno andiamo ad inserire tutti i comandi necessari affinchè alla fine tutti riescano a mandare richieste di ping correttamente.
### terzo step
Visto che io ho utilizzato dei server web apache, questi ultimi hano una pagina html dedicata.

Quindi dobbiamo creare per ogni server le directory ed il file:
- [nameserver]/var/www/html/index.html

## Introduction to firewalls and iptables
Un Firewall è “un dispositivo per la sicurezza della rete che permette di monitorare il traffico in entrata e in uscita utilizzando una serie predefinita di regole di sicurezza per consentire o bloccare gli eventi”. Per dispositivo si intende un elemento hardware o un’applicazione software.

Iptables è uno dei componenti principali per quanto riguarda la sicurezza in un sistema Linux, si tratta di un firewall implementato a livello kernel che ci permette tramite la creazione di regole di avere una protezione per il filtraggio del traffico. Iptables si basa su un meccanismo di regole, regole che determinano il lasciapassare o meno di pacchetti.

proprietà nella mia rete:
- ho una rete privata quindi solo i device all'interno della rete possono inviare pacchetti verso il mondo esterno. Il vceversa, ovvero, un device nella rete pubblica non può ad esempio mandare richieste di ping verso la rete privata.
- inoltre come vedete nell'immagine, la "public network 1" può essere considerata come una server farm. La caratteristica di questa rete è che abbiamo un server principale il "server2" e gli altri 2 server sono di supporto al server principale.
## Iptables implementation
### file r3.startup
Iniziamo con esaminare questo router. Il router che gestisce la nostra rete privata.
> iptables-legacy -A FORWARD -i eth1 -o eth0 -j DROP

questo comando fa in modo che tutto quello che arriva dall'esterno (nella scheda di rete eth1) e viene inoltrato (osserviamo il comando FORWARD) verso la rete privata(verso le schede eth0) viene lasciato cadere (-j DROP).

Di conseguenza un device all interno della rete privata può mandare richieste di ping correttamente senza problemi.
Qual'è il problema? che i pacchetti di ritorno non verranno lasciati passare a causa della regola IPtables appena inserita.
Per risolvere questo problema basta usare il comando qua sotto:
> iptables-legacy -A FORWARD -i eth1 -o eth0 -m state --state ESTABLISHED -j ACCEPT

Questo ci dice che: tutto quello che arriva da eth1 e viene inoltrato verso eth0 ma fa parte di una connessione già stabilita (-m state --state ESTABLISHED) deve essere accettata (-j ACCEPT).

Riassumento abbiamo utilizzato delle regole IPtables che ci permettono di gestire il traffico a livello FORWARDING.
Passiamo ora a livello INPUT.
Stesso discorso per il router stesso che facendo parte di una rete privata non può ricevere pacchetti provenienti dal mondo esterno (reti pubbliche).
Basterà quindi usare queste 2 IPtables:
> iptables-legacy -A INPUT -i eth1 -m state --state ESTABLISHED -j ACCEPT

> iptables-legacy -A INPUT -i eth1 -j DROP

Siamo come ripeto in una rete privata quindi dato che vogliamo fare le cose per bene, dobbiamo fare in modo che i pacchetti che escono dalla rete privata non abbiano l'indirizzo ip del device interno ma del router, quindi ecco qua che introduciamo il NAT:
> iptables-legacy -t nat -A POSTROUTING -s 10.0.0.0/16 -j MASQUERADE

Con questa regola diciamo che tutti quei pacchetti che hanno indirizzo sorgente (-s) 10.0.0.0/16 devono essere mascherati (-j MASQUERADE) con indirizzo ip del router. l'ip viene cambiato automaticamente nel POSTROUTING.
### file r2.startup
Questo router gestisce la server farm.
Nella farm ho voluto fare un ipotesi: abbiamo un server principale SW2 e 2 server di supporto.
In sostanza solo il server2 è visibile alle reti, quindi i device faranno richiesta di accesso alla pagina html del server2.
Io però non voglio sovraccaricare il server di richieste, quindi ho implementato tramite le IPtables un metodo dinamico che mi gestisce le richieste, il cosidetto: Round Robin Load Balancing.
- In sostanza le varie richieste vengono distribuite sui vari server. La prima richiesta viene direttamente gestita dal SW2, la seconda dal SW3, una terza dal SW4, e se ci dovesse essere una quarta richiesta questa torna ad essere gestita dal SW2 ed il ciclo riinizia.



