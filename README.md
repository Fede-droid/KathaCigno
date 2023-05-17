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
### file server2.startup
Iniziamo con esaminare questo router. Il router che gestisce la nostra rete privata.
>
>
