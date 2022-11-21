## Indice

1. [Introduzione](#introduzione)

   - [Informazioni sul progetto](#informazioni-sul-progetto)
   
   - [Scopo](#scopo)

2. [Analisi](#analisi)

    - [Analisi e specifica dei requisiti](#analisi-e-specifica-dei-requisiti)

3. [Implementazione](#implementazione)

4. [Test](#test)

   - [Protocollo di test](#protocollo-di-test)

   - [Risultati test](#risultati-test)

1. [Conclusioni](#conclusioni)

    - [Considerazioni personali](#considerazioni-personali)

2. [Sitografia](#sitografia)

<br>

## Introduzione

### Informazioni sul progetto

 - **Titolo**: Questionario patenti 

 - **Allievi coinvolti nel progetto**:
  
   - Samuel Banfi, <a href="mailto:samuel.banfi@samtrevano.ch">samuelbanfi@samtrevano.ch</a>
  
   - Dennis Donofrio, <a href="mailto:dennis.donofrio@samtrevano.ch">dennis.donofrio@samtrevano.ch</a>

 - **Classe**: I4AC, Scuola Arti e Mestieri Trevano, sezione Informatica

 - **Committente**: Massimo Sartori

 - **Data d'inizio**: 24.10.2022

 - **Data di fine**: 21.11.2022

<br>

### Scopo

Lo scopo del progetto `IP Tables` è quello di utilizzare un server Debian che fa da router e gestsice tutte le richieste in entrata e in uscita. Il Server Debian serve anche per dividere tutto in 2 reti Una interna e una DMZ. Una con indrizzo di rete `192.168.238.0/24` e una con indirizzo di rete `192.168.198.0/24`. Quest'ultima viene utilizzata come DMZ dove mettere il server WEB.

<br>

<div style="text-align:center">
  <img src="schema_di_rete.png" alt="schema_di_rete">
</div>

<br>

## Analisi

### Analisi e specifica dei requisiti

<br>

| ID | REQ-001 |
| -------- | - |
| **Nome** | Configurazione VM NET 3 |
| **Priorità** | 1 |
| **Versione** | 1.0 |
| **Note** | Bisogna avere una rete privata con indirizzo di rete `192.168.238.0/24`. <br> Tutti i computer nella rete possono accedere ad internet. <br> Non bisogna accedere dall'esterno. <br> Bisogna disabilitare il ping verso la rete interna. |

<br>

| ID | REQ-002 |
| -------- | - |
| **Nome** | Configurazione Server Apache |
| **Priorità** | 1 |
| **Versione** | 1.0 |
| **Note** | Bisognerà avere un server WEB con installato Apache. <br> Il servizio di Apache dovrà ascoltare sulla porta `443` (HTTPS) e `8080` (interno). <br> Bisognerà poter accedere alla pagina. |

<br>

| ID | REQ-003 |
| -------- | - |
| **Nome** | Configurazione VM NET 4 |
| **Priorità** | 1 |
| **Versione** | 1.0 |
| **Note** | Bisogna avere una rete DMZ con indirizzo di rete `192.168.198.0/24`. <br> Bisogna avere un server WEB al suo interno (REQ-002). <br> Bisogna aprire le connessioni in uscita. <br> Bisogna disabilitare le connessioni in entrata. (Escluso Apache). <br> Il server deve essere raggiungibile dall'esterno solo tramite l'IP del router. <br>  |

<br>

| ID | REQ-004 |
| -------- | - |
| **Nome** | Ping disabilitato |
| **Priorità** | 1 |
| **Versione** | 1.0 |
| **Note** | Il `ping` verso le reti interne deve essere impossibile.<br> Bisogna impostare le schede di rete su `Host-Only`|

<br>

| ID | REQ-005 |
| -------- | - |
| **Nome** | Accesso tramite SSH |
| **Priorità** | 1 |
| **Versione** | 1.0 |
| **Note** | Bisogna rendere possibile l'accesso al router tramite `SSH` per la configurazione. |

<br>

| ID | REQ-006 |
| -------- | - |
| **Nome** | Configurazione IP forwarding |
| **Priorità** | 1 |
| **Versione** | 1.0 |
| **Note** | Bisogna abilitare `ip_forwarding` |

<br>

| ID | REQ-007 |
| -------- | - |
| **Nome** | Impostare regole di default per IP Tables |
| **Priorità** | 1 |
| **Versione** | 1.0 |
| **Note** | Bisogna impostare le regole di default in modo da avere `INPUT` e `FORWARD` bloccati. Invece `OUTPUT` deve essere aperto. |

<br>

| ID | REQ-009 |
| -------- | - |
| **Nome** | Salvataggio regole IP Tables |
| **Priorità** | 1 |
| **Versione** | 1.0 |
| **Note** | Bisogna salvare le regole di `iptables`. |

<br>

| ID | REQ-010 |
| -------- | - |
| **Nome** | Caricamento automatico regole IP Tables ad ogni riavvio |
| **Priorità** | 1 |
| **Versione** | 1.0 |
| **Note** | Bisogna fare in modo che le regole di `iptables` vengano caricate automaticamente ad ogni riavvio. |

<br>

**Spiegazione elementi tabella dei requisiti:**

**ID**: identificativo univoco del requisito

**Nome**: breve descrizione del requisito

**Priorità**: indica l’importanza di un requisito nell’insieme del
progetto, definita assieme al committente.

**Versione**: indica la versione del requisito. Ogni modifica del
requisito avrà una versione aggiornata.

Sulla documentazione apparirà solamente l’ultima versione, mentre le
vecchie dovranno essere inserite nei diari.

**Note**: eventuali osservazioni importanti o riferimenti ad altri
requisiti.

<br>

## Implementazione

### Configurazione macchine virtuali

#### Router

Il router è configurato nel seguente modo:

- CPU: 2 core
- RAM: 2 GB
- Rete:
  - `NAT network`
  - `Internal network` lan (192.168.238.1/24)
  - `Internal network` dmz (192.168.198.1/24)
  - `Host-Only`
- Sistema operativo: `Debian`

<br>

#### PC rete interna

Il PC della rete interna è configurato nel seguente modo:

- CPU: 2 core
- RAM: 2 GB
- Rete: 
  - `Internal network` lan (192.168.238.10/24, 192.168.238.1)
  - `Host-Only` 
- Sistema operativo: `Debian`

<br>

#### Webserver

Il webserver è configurato nel seguente modo:

- CPU: 2 core
- RAM: 2 GB
- Rete:
  - `Internal network` dmz (192.168.198.10/24, 192.168.198.1)
  - `Host-Only` 
- Sistema operativo: `Debian`

<br>

#### Impostazione proxy

Durante l'installazione di tutte le macchine virtuali bisogna configurare il proxy impostando l'indirizzo `10.0.4.2:5865`. Il proxy serve solamente all'inizio su tutte le macchine per fare la configurazione iniziale e installare tutti gli aggiornamenti necessari.

<br>

#### Impostazione indirizzo ip

Essendo che stiamo lavorando su `Debian` per modificare l'indirizzo ip delle macchine bisogna modificare il file `/etc/network/interfaces` aggiungendo le varie schede di rete e impostando gli indirizzi ip (`address`), le subnet mask (`netmask`) e i `gateway`.

```bash
auto <interface>
iface <interface> inet static
    address <ip_address>
    netmask <ip_subnet>
    gateway <ip_gateway>
```

<br>

### Installazione e configurazione Apache

#### Installazione

Per installare `Apache 2.4` bisogna eseguire l'`update` per aggiornare tutte le librerie. In seguito bisogna eseguire il comando `sudo apt install apache2 -y` per installare Apache. Attenzione, per installare i pacchetti bisogna eseguire le operazioni come `sudo`, ovvero come `superuser`.

```bash
sudo apt update
sudo apt install apache2 -y
```
<br>

#### Configurazione porte in ascolto

Per configurare le porte in ascolto da Apache sul server bisogna modificare il file `etc/apache2/ports.conf` e aggiungere un `Listen` per la porta `8080`. Non serve aggiungerlo per la porta `443` perché è già presente di default come per la porta 80.

```bash
sudo nano /etc/apache2/ports.conf
```
```bash
Listen 8080
```

<br>

### Installazione e configurazione IP Tables

#### Installazione

Di default `iptables` non è presente su Debian. Quindi va installato tramite `apt`. Una volta installato tutte le regole di default vengono impostate su `accept`.

```bash
sudo apt install iptables -y
```

<br>

#### Mostrare tutte le regole attive

Per mostrare tutte le regole attive con IP Tables bisogna eseguire il comando `sudo iptables` aggiungendo il parametro `-S` per listare tutte le regole. Usando `iptables` vengono mostrate tutte le regole per l'`IPv4`. Se si volessero vedere le regole per `IPv6` bisogna usare il comando `sudo ip6tables -S`

```bash
sudo iptables -S
```

<br>

#### 

## Test

### Protocollo di test

 | Test Case       | TC-001                               |
 | --------------- |--------------------------------------|
 | **Nome**        |  |
 | **Riferimento** |  |
 | **Descrizione** |  |
 | **Prerequisiti** | - |
 | **Procedura** |  |
 | **Risultati attesi** |  |

### Risultati test

 | Test Case | TC-013 |
 | --------- | ------ |
 | Funzionamento |  |
 | Commento |  |
 | Data |  |

## Conclusioni

### Considerazioni personali

## Sitografia

 - sito<br>Data ultima visita: data