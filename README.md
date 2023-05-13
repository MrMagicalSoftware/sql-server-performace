# sql-server-performace


FORMAT A GIT README :

https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax


1. GETTING STARTED
2. ANALYZING STATEMENTS FOR PERFORMANCE
3. BUILDING INDEXES
4. FINDING BOTTLENECK IN SQL SERVER PERFOMANCE
5. CAPTURE TRACE LOGS OF APPLICATION FROM SQL SERVER
6. APPLYING COMMON BEST PRATICES FOR BETTER PERFOMANCE



# GETTING STARTED
## INTRODUCTION

 ![SQL1](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/a3fac35a-b10e-400b-a548-96930cf9f14f)


## Tools 


SQL SERVER DEVELOPER EDITION.

https://www.microsoft.com/it-it/sql-server/sql-server-downloads


Scaricare SQL Server Management Studio (SSMS)

https://learn.microsoft.com/it-it/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16



## Import the sample database


Import the Students.bak file


Database -> tasto destro -> ripristina database

Selezionare Device ... Add ( aggiungere ) 


## Table Concepts


I dati nel database vengono salvati come CLUSTER INDEX STRUCTURE.


Un indice clusterizzato in SQL Server è un tipo di indice che definisce l'ordine fisico dei dati in una tabella in base ai valori di una o più colonne. Quando una tabella ha un indice clusterizzato, i dati sono organizzati in un ordine specifico in base ai valori della chiave dell'indice clusterizzato.

La struttura di un indice clusterizzato in SQL Server è simile a quella di una struttura ad albero B, dove i valori della chiave dell'indice sono archiviati in ordine ordinato e ogni valore della chiave punta a una riga specifica di dati nella tabella. Tuttavia, a differenza di un indice non clusterizzato, in cui l'indice e i dati sono archiviati separatamente, un indice clusterizzato determina l'ordine fisico dei dati nella tabella stessa.

In un indice clusterizzato di SQL Server, la prima colonna nella chiave dell'indice viene utilizzata per determinare l'ordine fisico dei dati. Se ci sono colonne aggiuntive nella chiave dell'indice, vengono utilizzate come fattori di discriminazione per determinare l'ordine delle righe con lo stesso valore nella prima colonna.

Poiché l'indice clusterizzato determina l'ordine fisico dei dati, può migliorare le prestazioni delle query per le query che richiedono l'accesso sequenziale ai dati. Tuttavia, se la chiave dell'indice clusterizzato non viene scelta con attenzione, può portare a problemi di prestazioni e frammentazione.


** SQL SERVER CLUSTERED INDEX STRUCTURE **

![Schermata del 2023-05-13 16-07-42](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/a067348c-50fd-4370-860a-9636ee5bac89)

I DATI SONO ORGANIZZATI SOTTO FORMA DI ALBERO.
Nel leaf node sono i dati veri e propri,
ognuno di questi leaf node è come se fosse una pagina in sql-server,
i dati quindi vengono organizzati in modo ordinato,
secondo una cluster key della tabella.
Di Default la chiave primaria della tabella corrisponde alla cluser key.

> DEFAULT PRIMA KEY DELLA TABELLA === CLUSTER KEY


VEDIAMO QUESTO ESEMPIO :



![Schermata del 2023-05-13 16-18-22](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/3f3e2144-64ea-4e72-896c-84083d20ed69)



![Schermata del 2023-05-13 16-19-21](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/00d8003b-d80a-4842-8eb9-59cd1b1c8ec4)

LA TABELLA STUDENTE HA COME CHIAVE PRIMARIA STUDENTID DI TIPO INTEGER.

NEL NODI FOGLIA ("LEAF NODE") GLI STUDENTI SONO ORDINATI PER ID.

Per attraversare l'albero mi servono sostanzialmente 3 operazioni.
Ad esempio se voglio "trovare" lo studente con id=202,
prendo la secondo blocco e scendo in DATA-ROWS ( DA 201-300).


Se invece cerco uno studente per "FIRSTName" oppure "LastName" o altro,
vedi composizione della tabella dbo.Students, questo mi crea un problema,
perchè le foglie sono ordinati secondo un criterio per id.
ovvero non esiste una correlazione tra il numero id e il nome

Come fa a trovare il nome ?

Prende come riferimento tutti i leaf node e inizia a cercare.

![Schermata del 2023-05-13 16-28-53](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/eb06493d-5328-4e95-bea0-791a85cdc713)


Questo significa che ho tanti dati la query potrebbe impiegare del tempo.

![Schermata del 2023-05-13 16-31-25](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/be6050e4-d522-4be1-ab8b-b1a58d03dbdc)

Cosa possiamo fare per mitigare questo problema ?

Creare un'index sulla nostra tabella per aiutare sql a trovare i dati 
in maniera più efficiente.


## INDEX CONCEPTS


Abbiamo detto che bisognerebbe passare da una situazione di questo tipo :


![Schermata del 2023-05-13 16-34-55](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/e342ec98-678d-4e0b-91da-3001736def7a)


Ad organizzare i dati con index che viene "sortato" per lastname e firstname.



![Schermata del 2023-05-13 16-36-11](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/1b3a35bb-8d47-44a2-ac28-216fbc38f83b)

IN QUESTO CASO AVREMO CHE :

INDEX VALUES POINT TO WHERE THRE CORRESPONDING DATA IS IN THE TABLE.
 

![Schermata del 2023-05-13 16-40-42](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/e7cac084-7362-47af-a9a8-d2dca419f68b)



Supporniamo che si voglia cercare uno studente con una select.


```
SELECT FROM STUDENTS
WHERE LASTName='Jack' and FirstName='The ripper'
```
SUCCEDE Che prima viene recuperato con la tabella di destra dove
si trova il dato, una volta recuperato id, si recupera il dato
come visto in precedenza.

![Schermata del 2023-05-13 16-45-07](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/df5cb6d1-cbeb-447f-b8f4-112d80cc95ae)


AVVENGO QUINDI 2 OPERAZIONI , MA QUESTE 2 OPERAZIONI SONO MOLTOOO EFFIENTI ,
RISPETTO A CERCARE SU TUTTE LE ROWS.


Conviene quindi crearsi INDEX MULTI SE PREVEDO CHE DEVO FARE IL RETRIEVE DEI DATI.


![Schermata del 2023-05-13 16-49-01](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/40dae7d4-29f2-48bb-804c-bf0f5a196448)



![Schermata del 2023-05-13 16-50-54](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/fa8741bf-1d65-4ac4-8629-1cfecb605bf7)


# Analyzing SQL Statements for Performance

1. Introduction
2. Understanding How Sql Server Will Execute a Sql statement
3. Reading and interpreting ad Execution Plan for a sql statement
4. Getting Execution statitics for a sql statement
5. Improving statement performance by Adding an index
6. rewriting sql statements for improved performance
7. common execution plan operations



 
Supponiamo di avere delle query che sono lente, oppure vogliamo ottimizzare 
i dati nella maniera più corretta.
devo analizzare correttamente le mie query e capire se sono performanti.
Devo generare quello che si chiama PIANO DI ESECUZIONE.

**PIANO DI ESECUZIONE IN SQL**


Il piano di esecuzione è un componente fondamentale del motore di database di SQL Server che determina come una query viene eseguita e restituisce i risultati. Il piano di esecuzione descrive come il motore di database di SQL Server utilizza gli indici, le tabelle, le viste e gli altri oggetti del database per elaborare una query e restituire i risultati al client.

Il piano di esecuzione è generato dal motore di database di SQL Server per ogni query che viene inviata al server. Il piano di esecuzione può essere visualizzato utilizzando diversi strumenti di monitoraggio e profilazione in SQL Server, come ad esempio l'Activity Monitor, SQL Server Management Studio, SQL Server Profiler e SQL Server Data Tools.

Comprendere il piano di esecuzione può essere utile per identificare le inefficienze e le aree di miglioramento delle prestazioni nelle query. Analizzando il piano di esecuzione, è possibile identificare le operazioni costose e le operazioni che possono essere ottimizzate, come l'utilizzo di indici appropriati, la riduzione della quantità di dati letti dalle tabelle e l'utilizzo di tecniche di join efficienti.

Il piano di esecuzione è uno strumento essenziale per l'ottimizzazione delle prestazioni delle query in SQL Server, che consente di comprendere il modo in cui una query viene elaborata e di identificare le aree di miglioramento delle prestazioni.



## Understanding How Sql Server Will Execute a Sql statement


Supponiamo di voler eseguire questa query 



```
select c.DepartmentCode, c.CourseNumber, c.CourseTitle, c.Credits,ce.Grade
from CourseEnrollments ce
inner join CourseOfferings co on co.CourseOfferingId = ce.CourseOfferingId
inner join Courses c on co.DepartmentCode = c.DepartmentCode AND co.CourseNumber = c.CourseNumber
where ce.StudentId = 29717

```


Per ottenere questa risultati abbiamo fatto il join con 3 tabelle.


Questa query SQL seleziona il codice del dipartimento, il numero del corso, il titolo del corso, i crediti e il voto di tutti gli studenti che hanno effettuato l'iscrizione a un corso specifico, identificato dallo studente con ID 29717.

La query utilizza tre tabelle: CourseEnrollments, CourseOfferings e Courses. La tabella CourseEnrollments contiene informazioni sugli studenti iscritti ai corsi, mentre la tabella CourseOfferings contiene informazioni sulle offerte dei corsi e la tabella Courses contiene informazioni sui corsi stessi.

La clausola INNER JOIN viene utilizzata per unire le tre tabelle sulla base di specifiche condizioni di join. In particolare, la prima JOIN unisce le tabelle CourseEnrollments e CourseOfferings sulla base dell'ID dell'offerta del corso, mentre la seconda JOIN unisce le tabelle CourseOfferings e Courses sulla base del codice del dipartimento e del numero del corso.

La clausola WHERE viene utilizzata per filtrare i risultati in base all'ID dello studente. Solo i record che corrispondono allo studente con ID 29717 vengono restituiti.

In sintesi, la query seleziona le informazioni sui corsi e le informazioni sugli studenti che hanno effettuato l'iscrizione a un corso specifico, identificato dall'ID dello studente. Le informazioni sui corsi includono il codice del dipartimento, il numero del corso, il titolo del corso, i crediti e il voto dell'iscrizione al corso.



Per visualizzare il **piano di esecuzione** posso premere la combinazione di tasti CTRL + L

![Schermata 2023-05-13 17:28:52](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/67822bbd-2a3a-4154-a0ba-1a7598311267)


![Schermata del 2023-05-13 17-31-24](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/0830c9d5-73aa-4b83-a0b4-d11e18715117)


Ora bisogna capire come interpretare questo piano di esecuzione.



## Reading and interpreting an Execution Plan for a Sql Statment


Un piano di esecuzione in SQL Server rappresenta il metodo che il motore di database utilizza per elaborare una query SQL e restituire i risultati richiesti. Questo piano è una rappresentazione grafica delle operazioni che vengono eseguite dal server durante l'esecuzione della query, nonché delle risorse utilizzate dal server, come CPU, memoria, I/O e quantità di dati scambiati.

Per interpretare un piano di esecuzione in SQL Server, è possibile seguire i seguenti passaggi:

1. Analizzare la struttura del piano di esecuzione: un piano di esecuzione può essere visualizzato utilizzando diversi strumenti di monitoraggio e profilazione in SQL Server, come ad esempio SQL Server Management Studio. Una volta ottenuto il piano di esecuzione, analizzare la struttura del piano per capire le diverse operazioni che vengono eseguite dal server e l'ordine in cui vengono eseguite.

2. Identificare le operazioni costose: esaminare il piano di esecuzione per identificare le operazioni che richiedono molte risorse, come CPU, I/O e memoria. Queste operazioni costose possono indicare potenziali problemi di prestazioni della query e possono richiedere l'ottimizzazione della query.

3. Esaminare le proprietà delle operazioni: ogni operazione del piano di esecuzione ha una serie di proprietà che possono essere esaminate per comprendere meglio il funzionamento della query. Ad esempio, la proprietà "Actual Number of Rows" indica il numero di righe effettivamente restituite dalla query, mentre la proprietà "Estimated I/O Cost" indica il costo stimato in termini di I/O delle operazioni.

4. Verificare l'utilizzo degli indici: esaminare il piano di esecuzione per verificare l'utilizzo degli indici nella query. In particolare, verificare che gli indici vengano utilizzati correttamente e che siano presenti gli indici necessari per le operazioni di join e di ricerca.

5. Verificare l'utilizzo dei filtri: esaminare il piano di esecuzione per verificare l'utilizzo dei filtri nella query. I filtri sono utilizzati per limitare il numero di righe restituite dalla query e possono migliorare le prestazioni.

Pper interpretare un piano di esecuzione in SQL Server, è necessario analizzare la struttura del piano, identificare le operazioni costose e le proprietà delle operazioni, verificare l'utilizzo degli indici e dei filtri, e comprendere come la query viene elaborata dal motore di database. Queste informazioni possono essere utilizzate per identificare le aree di miglioramento delle prestazioni della query.


Per leggere il piano di esecuzione lo leggo da destra verso sinistra, e ( top-bottom).

Quindi (right -left ) e  ( top - bottom ).

Ad esempio la parte più a sinistra è l'operazione finale che viene eseguita
per ultima.

Nota : in realtà molte operazioni avvengono in parallelo, ma per semplicità
pensiamo alla disposizione spaziale destra versa sinistra.

Ad esempio noto che la prima operazione ha un costo del 91 % 

Se mi avvicino con il mouse vicino alle operazioni posso notare tutti i costi
che le riguardano.

ad esempio il tipo di operazione della prima operazione è di tipo **CUSTERED INDEX SCAN**

_______________________________

Un clustered index scan in SQL Server è un'operazione di scansione di un indice cluster. Un indice cluster è un tipo di indice in cui i dati sono fisicamente ordinati sulla base del valore della chiave di indice. Ciò significa che i dati della tabella sono organizzati in modo da corrispondere all'ordine della chiave di indice.

Durante un'operazione di clustered index scan, il motore di database SQL Server legge tutti i dati della tabella nell'ordine definito dalla chiave di indice cluster. Ciò significa che i dati vengono letti in sequenza dalla posizione di memoria fisica in cui sono archiviati.

Questa operazione è particolarmente utile per le query che richiedono l'accesso a un ampio numero di righe, poiché il motore di database può accedere alle righe in modo rapido e in sequenza.

Tuttavia, se la query richiede solo alcune righe specifiche, una scansione di un indice cluster potrebbe non essere efficiente. In questo caso, sarebbe più efficiente utilizzare un'operazione di ricerca tramite un indice non cluster, in cui solo le righe specifiche vengono lette.

In generale, un'operazione di clustered index scan può essere un'operazione efficiente per le query che richiedono l'accesso a un ampio numero di righe, ma potrebbe non essere la scelta migliore per le query che richiedono solo alcune righe specifiche.

__________________________________

Quindi per grandi tabelle questa operazione è molto costosa e richiede molto tempo.


Vediamo alcuni dettagli :



1. Physical operation: questa sezione del piano di esecuzione descrive l'operazione fisica che il motore di database SQL Server deve eseguire per recuperare i dati richiesti dalla query. Ad esempio, l'operazione potrebbe essere una scansione di indice, un join o una aggregazione.

2. Logical operation: questa sezione del piano di esecuzione descrive l'operazione logica che il motore di database SQL Server deve eseguire per recuperare i dati richiesti dalla query. Ad esempio, l'operazione potrebbe essere una selezione, un progetto o un'aggregazione.

3. Estimated execution mode: questa sezione del piano di esecuzione mostra come il motore di database SQL Server prevede di eseguire la query. Ad esempio, potrebbe prevedere di utilizzare un'operazione di join nidificata piuttosto che un join di hash.

4. Actual execution mode: questa sezione del piano di esecuzione mostra come la query viene effettivamente eseguita dal motore di database SQL Server. A volte, la modalità di esecuzione reale può differire dalla modalità prevista a causa di fattori come la dimensione dei dati, la distribuzione dei dati o la configurazione del server.

5. Estimated number of rows: questa sezione del piano di esecuzione mostra il numero stimato di righe che saranno restituite da ogni operazione nella query.

6. Actual number of rows: questa sezione del piano di esecuzione mostra il numero effettivo di righe restituite da ogni operazione nella query durante l'esecuzione.


![Schermata del 2023-05-13 17-56-20](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/0c241640-92b3-4219-8f8d-37cb9f4e6563)


Se mi sposto sul nodo più a sinistra noto a quanto ammonta il costo 
totale dell'operazione.

![Schermata del 2023-05-13 18-57-40](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/39892fef-72b7-4606-9a67-b3f243fc1bf4)

Cosa sono le operazioni svolte con parallelismo ?

Le operazioni svolte con parallelismo si intende dividere
un task in sotto-task, ogni task ha la propria esecuzione,
e alla fine avviene un'operazione di merge per ottenere la soluzione finale.
Il parallelismo consente di eseguire le query in maniera più veloce.



> TIPS : CONVIENE FOCALIZZARSI SULLE OPERAZIONI AD ALTO COSTO



## Analyzing Sql statements for performance.

Posso anche usare dei comandi di sql server.





```

set STATISTICS IO ON
set STATISTICS TIME ON


select c.DepartmentCode, c.CourseNumber, c.CourseTitle, c.Credits,ce.Grade
from CourseEnrollments ce
inner join CourseOfferings co on co.CourseOfferingId = ce.CourseOfferingId
inner join Courses c on co.DepartmentCode = c.DepartmentCode AND co.CourseNumber = c.CourseNumber
where ce.StudentId = 29717

```

Nella tabella  dei messaggi ( vicino a Risultati ) ho ad esempio :


______________________________________

(40 righe interessate)
Tabella 'CourseEnrollments'. Conteggio analisi 5, letture logiche 12197, letture fisiche 1, letture server di pagine 0, letture read-ahead 12021, letture read-ahead server di pagine 0, letture logiche LOB 0, letture fisiche LOB 0, letture LOB server di pagine 0, letture LOB read-ahead 0, letture read-ahead LOB server di pagine 0.
Tabella 'Courses'. Conteggio analisi 0, letture logiche 80, letture fisiche 3, letture server di pagine 0, letture read-ahead 0, letture read-ahead server di pagine 0, letture logiche LOB 0, letture fisiche LOB 0, letture LOB server di pagine 0, letture LOB read-ahead 0, letture read-ahead LOB server di pagine 0.
Tabella 'CourseOfferings'. Conteggio analisi 0, letture logiche 133, letture fisiche 2, letture server di pagine 0, letture read-ahead 104, letture read-ahead server di pagine 0, letture logiche LOB 0, letture fisiche LOB 0, letture LOB server di pagine 0, letture LOB read-ahead 0, letture read-ahead LOB server di pagine 0.
Tabella 'Worktable'. Conteggio analisi 0, letture logiche 0, letture fisiche 0, letture server di pagine 0, letture read-ahead 0, letture read-ahead server di pagine 0, letture logiche LOB 0, letture fisiche LOB 0, letture LOB server di pagine 0, letture LOB read-ahead 0, letture read-ahead LOB server di pagine 0.

Tempo di esecuzione SQL Server: 
 tempo di CPU = 375 ms, tempo trascorso = 279 ms.

Ora di completamento: 2023-05-13T19:08:41.5559494+02:00


______________________________________


Questo risultato rappresenta le statistiche di accesso alle tabelle coinvolte nella query. In particolare, per ogni tabella viene riportato il numero di analisi eseguite, le letture logiche (cioè il numero di volte in cui è stata acceduta la cache dei dati in memoria), le letture fisiche (cioè il numero di volte in cui sono state lette le pagine dal disco), le letture server di pagine (cioè il numero di letture richieste al server), le letture read-ahead (cioè il numero di letture eseguite in previsione di future richieste di dati), le letture logiche LOB (cioè il numero di letture di oggetti di grandi dimensioni, come ad esempio immagini o documenti), le letture fisiche LOB, le letture LOB server di pagine, le letture LOB read-ahead e le letture read-ahead LOB server di pagine. Inoltre, viene riportato il tempo di esecuzione totale della query, suddiviso tra tempo di CPU (cioè il tempo di elaborazione trascorso sulla CPU) e tempo trascorso (cioè il tempo trascorso per l'esecuzione della query, inclusi i tempi di attesa per l'accesso al disco). Il tempo di completamento indica il momento in cui la query è stata completata.


> PRINCIPIO GENERALE : TANTO PIU' E' GRANDE IL "LOGICAL READS" TANTO PIU' E' INEFFICIENTE.


VEDIAMO COME MIGLIORARE TUTTA LA SITUAZIONE.


## IMPROVING STATEMENT PERFORMANCE BY ADDING AN INDEX.



ABBIAMO DETTO CHE QUELLO CHE DOBBIAMO MIGLIORARE E' COST 91 % 


![Schermata del 2023-05-13 19-18-10](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/4cca8762-c81d-4763-9b07-631e61655150)


lettura di 12000 pages dalla tabella solo per trovare
una manciata di righe di cui abbiamo bisogno per uno studente

sql mi da un suggerimento in verde di cosa posso andare a migliorare.

![Schermata del 2023-05-13 19-23-56](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/c21b4ed3-00d1-4d67-ac33-2705ffee5bcf)

Tasto destro e seleziono MISSING INDEX DETAILS :


![Schermata del 2023-05-13 19-25-06](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/0ac5c87b-ee67-4237-b73e-1a563eb01f77)


TOLGO I COMMENTI E DO UN NOME A QUESTO INDICE :


```
USE [Students]
GO
CREATE NONCLUSTERED INDEX IX_CourseEnrollments_StudentID
ON [dbo].[CourseEnrollments] ([StudentId])

GO

```

> EXECUTE THE COMMAND


Vediamo ora che impatto ha avuto questa operazione.



![Schermata del 2023-05-13 19-31-21](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/9122bc6a-8603-40f2-90f6-800f11125303)

Siamo passati da 10.92 a 0.003


Il costo totale per eseguire tutto è di 0.207 contro i 12 di prima










































