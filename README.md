# sql-server-performace



SQL SERVER 2019 
https://go.microsoft.com/fwlink/?linkid=866662
________________


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



I logical reads sono passati da 12k a 134



![Schermata del 2023-05-13 19-36-23](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/2c51ba60-17b5-4ece-a4dd-e14d7b1521e1)



CONFRONTO FINALE :

![Schermata del 2023-05-13 19-37-37](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/1d361c83-3b8d-49f5-9da2-a8a2a6add511)

VEDIAMO UN'ALTRO ESEMPIO DI OTTIMIZZAZIONE CHE NON RICHIEDE L'AGGIUNTA DI UN INDICE


## Rewriting sql statements for improved performance





```
select co.*
from CourseOfferings co
LEFT JOIN CourseEnrollments ce on co.CourseOfferingId = ce.CourseEnrollmentId
where co.TermCode = 'SP2016' AND ce.CourseOfferingId IS NULL

```


Questa query seleziona tutti i record dalla tabella CourseOfferings (abbreviata in co) dove il campo TermCode è uguale a 'SP2016' e il CourseOfferingId non è presente nella tabella CourseEnrollments (abbreviata in ce). 

In particolare, viene utilizzata una LEFT JOIN tra le due tabelle, in cui la tabella CourseOfferings viene mantenuta come tabella di base e la tabella CourseEnrollments come tabella secondaria. La condizione di join è data dalla corrispondenza tra il campo CourseOfferingId della tabella CourseOfferings e il campo CourseEnrollmentId della tabella CourseEnrollments.

La clausola WHERE specifica ulteriori condizioni: il TermCode della tabella CourseOfferings deve essere uguale a 'SP2016' e il campo CourseOfferingId della tabella CourseEnrollments deve essere nullo (IS NULL).

Il risultato della query sono tutti i record della tabella CourseOfferings che soddisfano le condizioni specificate nella clausola WHERE e che non hanno una corrispondenza nella tabella CourseEnrollments.


> controllare minuto 53

The issue here is that sql server is looking for all the enrollments for each course.
when we really only need to know if there is any student enroll in the course
since we are looking up all the enviroments these operations turns out to be pretty
expensive making our whole query pretty exprensive.
Now in this case the answare is not an index but instead for us to change how we
write this query.
So let's take and alternate approach.


```
select co.*
from CourseOfferings co
Where NOT EXISTS
(SELECT 1 FROM CourseEnrollments ce WHERE co.CourseOfferingId = ce.CourseEnrollmentId)
AND co.TermCode = 'SP2016'

```

Otteniamo lo stesso risultato in termini di ricerca.
ma l'ottimizzazione della querty è notevolmente migliorata.


![Schermata del 2023-05-14 16-19-29](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/144c508e-1d88-4a89-ad97-1a8a368b4fdd)



## Rewriting a statement smetimes improves performance.

**Common Execution Plan Operations**


________________________

Nota pratica :

Subqueries can be rewritten as joins
Joins can be rewritten as subqueries
Exists and not exists are best in some scenarios

_______________________

In SQL Server, quando si eseguono query su grandi tabelle, il motore di database deve trovare rapidamente i dati richiesti. Per fare questo, il motore utilizza gli indici, che sono strutture dati che consentono di accedere ai dati più velocemente rispetto alla scansione dell'intera tabella. Tuttavia, il tipo di accesso all'indice utilizzato dipende dalla struttura dell'indice e dalla query stessa.

Di seguito spiego i diversi tipi di accesso all'indice:

- Clustered Index Scan: questo tipo di accesso all'indice viene utilizzato quando viene richiesto l'intero contenuto di una tabella indicizzata in modo cluster. Questo tipo di accesso all'indice legge tutte le pagine della tabella per trovare i dati richiesti e quindi può essere molto costoso in termini di tempo.

- Table Scan: questo tipo di accesso all'indice viene utilizzato quando una query richiede l'intero contenuto di una tabella che non ha un indice o quando l'indice non può essere utilizzato per una particolare query. Questo tipo di accesso all'indice legge tutte le pagine della tabella e può essere molto costoso in termini di tempo, soprattutto per le tabelle di grandi dimensioni.

- Clustered Index Seek: questo tipo di accesso all'indice viene utilizzato quando viene richiesto un sottoinsieme dei dati di una tabella indicizzata in modo cluster. In questo tipo di accesso all'indice, il motore di database utilizza l'indice cluster per trovare la posizione della pagina di dati richiesta e quindi recupera solo i dati richiesti. Questo tipo di accesso all'indice è generalmente molto efficiente.

- Index Scan: questo tipo di accesso all'indice viene utilizzato quando una query richiede l'intero contenuto di una tabella che ha un indice non cluster o quando l'indice non può essere utilizzato per una particolare query. Questo tipo di accesso all'indice legge tutte le pagine dell'indice e quindi accede alle pagine di dati richieste. Può essere efficiente se l'indice è piccolo o se la maggior parte dei dati deve essere recuperata.

- Index Seek: questo tipo di accesso all'indice viene utilizzato quando viene richiesto un sottoinsieme dei dati di una tabella indicizzata non cluster. In questo tipo di accesso all'indice, il motore di database utilizza l'indice non cluster per trovare la posizione della pagina di dati richiesta e quindi recupera solo i dati richiesti. Questo tipo di accesso all'indice è generalmente molto efficiente.

L'utilizzo di un indice può migliorare significativamente le prestazioni delle query, ma il tipo di accesso all'indice utilizzato dipende dalla struttura dell'indice e dalla query stessa.

_________________________________________

**SCAN OPERATION VS SEEK OPERATION**

> SCAN OPERATION MEANS THAT SQL SERVER IS READING THE ENTIRE DATA STRUCTURE


Le differenze principali tra le operazioni di scan e seek in SQL Server riguardano la modalità di accesso ai dati e l'efficienza della query.

Le operazioni di **scan** implicano la scansione di tutte le righe di una tabella o di un indice, mentre le operazioni di seek utilizzano un indice per cercare e accedere alle righe specifiche richieste dalla query. In altre parole, le operazioni di scan richiedono di leggere tutte le righe della tabella o dell'indice, mentre le operazioni di seek cercano solo le righe necessarie e recuperano solo quelle.

Le operazioni di scan sono generalmente meno efficienti delle operazioni di seek in termini di prestazioni perché richiedono più tempo per elaborare e recuperare i dati. Tuttavia, le operazioni di scan possono essere più efficienti in alcune situazioni in cui la maggior parte o tutte le righe della tabella devono essere lette, ad esempio quando si esegue un backup completo di un database.

Le operazioni di seek sono generalmente più efficienti delle operazioni di scan in termini di prestazioni perché richiedono meno tempo per recuperare i dati richiesti dalla query. Tuttavia, le operazioni di seek richiedono che la tabella abbia un indice appropriato per supportare la query, altrimenti il motore di database dovrà utilizzare le operazioni di scan per recuperare i dati richiesti.

**NOTA :**

Le operazioni di scan sono utilizzate quando è necessario accedere a molte o tutte le righe di una tabella, mentre le operazioni di seek sono utilizzate quando è necessario accedere solo a un sottoinsieme specifico di righe. L'uso corretto di queste operazioni dipende dalle esigenze specifiche della query e dalla struttura dell'indice e della tabella.


L'idea di fondo è quella di dire che quando nel piano di esecuzione vediamo delle operazioni di scan
dobbiamo capire perchè vengono fatte in questo modo, e magari modificarle in seek operation.


________________________________________________________________________________________



![Schermata del 2023-05-14 17-46-03](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/8e15c79a-5d88-45be-8384-00f2d61ad71a)





Le join operations sono uno dei concetti fondamentali in SQL Server per combinare i dati di più tabelle in una singola query. Ci sono diverse tecniche di join disponibili, tra cui Nested Loops Join e Merge Join.

Nested Loops Join è una tecnica di join che utilizza un loop nidificato per confrontare ogni riga di una tabella con ogni riga di un'altra tabella. Questa tecnica di join è adatta per le tabelle di piccole o medie dimensioni e quando le tabelle coinvolte hanno un indice adeguato per la colonna di join. Nested Loops Join utilizza l'indice su una delle tabelle e la scansione completa dell'altra tabella. L'algoritmo esamina ogni riga della prima tabella e cerca le corrispondenti righe nella seconda tabella, utilizzando l'indice, una alla volta. Questa tecnica può essere molto inefficiente se le tabelle coinvolte sono molto grandi o se l'indice non è ottimizzato.

Esempio:
SELECT *
FROM tabella1
INNER JOIN tabella2
ON tabella1.colonna = tabella2.colonna;

Merge Join è una tecnica di join che ordina le tabelle di input in base alla colonna di join e quindi scansiona le tabelle simultaneamente. Questa tecnica di join è adatta per le tabelle di grandi dimensioni e quando le tabelle hanno già un indice sulla colonna di join. Merge Join è particolarmente efficiente quando le tabelle sono ordinate sulla colonna di join. L'algoritmo utilizza due buffer per mantenere le righe della prima e della seconda tabella, mentre le scansiona in modo sincronizzato. Questa tecnica richiede un po' più di tempo per eseguire la preparazione dei dati, ma una volta che i dati sono ordinati, la join stessa è molto veloce.

Esempio:
SELECT *
FROM tabella1
INNER JOIN tabella2
ON tabella1.colonna = tabella2.colonna
ORDER BY tabella1.colonna, tabella2.colonna;

In sintesi, Nested Loops Join è una buona scelta per tabelle di piccole o medie dimensioni, mentre Merge Join è più adatto per tabelle di grandi dimensioni. La scelta della tecnica di join corretta dipende dalle esigenze specifiche della query e dalla struttura delle tabelle coinvolte. Inoltre, l'utilizzo degli indici corretti può migliorare significativamente le prestazioni delle operazioni di join.



Hash Join è una tecnica di join utilizzata in SQL Server per combinare grandi tabelle di dati. A differenza di Nested Loops Join e Merge Join, Hash Join non utilizza l'ordinamento o l'indice per eseguire la join. Invece, Hash Join crea una tabella hash temporanea che contiene le colonne di join delle tabelle di input.

Il processo di Hash Join può essere suddiviso in tre fasi principali:

1. Hash Table Build: per la prima tabella di input, SQL Server crea una tabella hash temporanea, che contiene la colonna di join e il valore della colonna associata per ogni riga. Il valore della colonna di join viene utilizzato come chiave per l'hashing.

2. Hash Table Probe: per la seconda tabella di input, SQL Server confronta ogni riga con la tabella hash temporanea creata nella fase precedente. Per ogni riga nella seconda tabella, SQL Server cerca la corrispondente riga nella tabella hash temporanea utilizzando l'hashing sulla colonna di join.

3. Join Results: se SQL Server trova una corrispondenza nella tabella hash temporanea, unisce le righe di input e restituisce il risultato della join. Questo processo continua finché tutte le righe nella seconda tabella di input sono state confrontate con la tabella hash temporanea.

Hash Join è particolarmente efficiente quando le tabelle di input sono grandi e non sono ordinate sulla colonna di join. Inoltre, Hash Join richiede una quantità limitata di memoria temporanea per creare la tabella hash. Tuttavia, Hash Join può essere meno efficiente se la colonna di join non ha valori unici o se la tabella hash temporanea diventa troppo grande per essere mantenuta in memoria.

Esempio:
SELECT *
FROM tabella1
INNER HASH JOIN tabella2
ON tabella1.colonna = tabella2.colonna;

In sintesi, Hash Join è una tecnica di join potente e flessibile che può essere utilizzata per combinare grandi tabelle di dati in SQL Server. Tuttavia, come per tutte le tecniche di join, la scelta della tecnica corretta dipende dalle esigenze specifiche della query e dalla struttura delle tabelle coinvolte.



![Schermata del 2023-05-14 17-53-24](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/cfa7e0c9-9887-4011-9fb5-89a017a2faac)



**RIASSUNTO**


![Schermata del 2023-05-14 18-07-22](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/90c9d302-4c24-4c73-aad6-e53df41d8a9c)

NOTA : QUANDO SI INCONTRANO LE OPERAZIONI DI JOIN SI LEGGONO BOTTOM.TOP

![Schermata del 2023-05-14 18-09-03](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/0c3d87a7-d604-4873-8019-20ae476a5786)


NOTA : RICORDARSI DI FOCALIZZARE LA NOSTRA ATTENZIONE
SUI COSTI MAGGIORI.


![Schermata del 2023-05-14 18-10-36](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/773439c3-8ca3-4454-b3b2-661093ce05e0)


NOTA : SULLA PARTE PIU' A SINISTRA PER AVERE UNA STIMA DEL COSTO TOTALE GUARDO QUESTO :



![Schermata del 2023-05-14 18-12-01](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/cb2b0f6e-f952-4aa7-9ad6-c1beed90ae58)



FOCALIZZARE LA NOSTRA ATTENZIONE SUI LOGICAL READS :


![Schermata del 2023-05-14 18-13-28](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/8025f172-8bf7-4151-9036-9f035da66649)



**TUNING OPERATIONS**

![Schermata del 2023-05-14 18-15-56](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/2f0e8b10-0181-4f1f-9e1e-098c3cd9f182)

![Schermata del 2023-05-14 18-17-04](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/056a8d9b-f240-4aca-ac5c-d3bffa63e63e)


MATERIALE GRATUITO DI APPROFONDIMENTO :

https://www.red-gate.com/simple-talk/books/sql-server-execution-plans-third-edition-by-grant-fritchey/




# Building EFFECTIVE Indexes


1. Index Terminology refresher
2. What columns should i index?
3. Why column order matters in an index
4. Index selectivity
5. Include columns and covering indexes
6. Functions in the where clause and indexes.
7. Over Indexing
8. Sql server's index recommendations




### INDEX TERMINOLOGY REFRESHER


**CLURESTED VS NON-CLUSTERED INDEXES**


In SQL Server, gli indici vengono utilizzati per migliorare le prestazioni delle query, consentendo al motore di database di accedere rapidamente ai dati richiesti. Esistono due tipi principali di indici in SQL Server: clustered e non-clustered.

Un clustered index ordina fisicamente i dati della tabella in base alla colonna di indice selezionata. Ogni tabella può avere solo un clustered index, e di solito viene utilizzato per la colonna di chiave primaria della tabella. Quando si esegue una query che fa riferimento alla colonna di indice del clustered index, SQL Server può accedere direttamente alle righe di dati in base all'ordine della colonna di indice, rendendo le ricerche più efficienti. Tuttavia, gli aggiornamenti frequenti dei dati nella tabella possono causare una sovrapposizione e una frammentazione dei dati, il che può causare un impatto negativo sulle prestazioni.

Un non-clustered index, d'altra parte, non riordina fisicamente i dati della tabella. Invece, crea una struttura di indice separata che fa riferimento alla posizione fisica dei dati nella tabella. Un non-clustered index può essere creato su qualsiasi colonna della tabella e una singola tabella può avere più di un non-clustered index. Quando si esegue una query che fa riferimento alla colonna di indice di un non-clustered index, SQL Server utilizza l'indice per trovare la posizione fisica dei dati nella tabella, rendendo le ricerche più efficienti. Tuttavia, poiché i dati della tabella non sono riordinati, l'accesso ai dati stessi potrebbe essere più lento rispetto a un clustered index.

In generale, si consiglia di utilizzare un clustered index sulla colonna di chiave primaria della tabella, poiché questo offre il miglior supporto per le operazioni di ricerca e ordinamento. Tuttavia, se si desidera creare un indice su una colonna che non sia la chiave primaria, un non-clustered index potrebbe essere la scelta migliore. Inoltre, se la tabella subisce frequenti operazioni di aggiornamento, può essere preferibile utilizzare un non-clustered index per ridurre l'impatto sulle prestazioni delle operazioni di aggiornamento. In ogni caso, la scelta tra un clustered index e un non-clustered index dipende dalle esigenze specifiche della query e dalle caratteristiche della tabella coinvolta.



![Schermata del 2023-05-14 19-00-54](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/235338b6-97f5-450a-b3f6-63cffaeeb396)


Nota :

In  genere la cluster index è costituita sulla chiave primaria della tabella.
tutti gli altri index che creiamo sulla tabella vengono chiamati "non cluster index"

Non cluster index is built over one or more columuns of the table witch define the index
key and the index also stored a row pointer where the table the matching rows for that index
key are located 



![Schermata del 2023-05-14 19-45-39](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/87309f49-2bb7-47df-9384-a4e4fff34f01)


Sulla chiave posso fare 2 tipi di operazione :

**OPERAZIONI SULLA CHIVE SCAN VS Seek Operation**

![Schermata del 2023-05-14 19-48-08](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/b7f7fd40-ad41-459d-9c3c-725a6aebfd66)



![Schermata del 2023-05-14 19-47-50](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/fd8d4a9a-0b08-48a5-aaf1-52b0a1247e83)

### What should i index in my database ?


![Schermata del 2023-05-14 19-59-25](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/7cc8fadc-33db-41c1-8dd4-2b960b2b5ec6)


______________________________________________________


La scelta degli indici giusti per il proprio database dipende dalle esigenze specifiche della propria applicazione e dal modo in cui vengono effettuate le query sui dati. Gli indici possono migliorare notevolmente le prestazioni delle query, ma un uso scorretto degli stessi può anche avere un impatto negativo sulle prestazioni generali del database.

In linea generale, gli indici dovrebbero essere creati per le colonne che vengono spesso utilizzate nelle clausole WHERE e JOIN delle query. In questo modo, il motore di database può utilizzare l'indice per trovare rapidamente le righe di dati corrispondenti alla query.

Tuttavia, la creazione di troppi indici può causare problemi di prestazioni, poiché ogni indice aggiuntivo comporta un costo in termini di spazio su disco e tempo di aggiornamento dei dati. Inoltre, gli indici possono diventare inutili o persino dannosi se vengono creati su colonne che non vengono utilizzate frequentemente nelle query o se le query coinvolgono molte tabelle.

Per determinare quali indici creare, è utile monitorare le query eseguite sul database e analizzare i piani di esecuzione delle query per individuare eventuali aree che potrebbero beneficiare di un indice. Inoltre, è possibile utilizzare gli strumenti di analisi delle prestazioni forniti da SQL Server per identificare le query che richiedono più tempo e risorse del necessario.

La scelta degli indici giusti dipende dalle specifiche esigenze del database e delle query eseguite. Un uso ponderato degli indici può migliorare significativamente le prestazioni del database, ma è importante evitare di creare indici superflui o dannosi.


**Example**

Add Indexes for where clause Criteria


![Schermata del 2023-05-14 20-04-17](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/4e6791e2-18a5-4d38-90b1-3949f47e7c43)




![Schermata del 2023-05-14 20-05-38](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/2d531bcf-1283-436b-baee-58cb15f3228e)

![Schermata del 2023-05-14 20-08-23](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/1cff96c3-0bf6-4087-b083-f531f95d109c)


Seconda categoria di colonne di cui vogliamo essere sicuri che che abbiamo l'index
sono le foreign key columns nella nostra tabella.
Ci sono 2 motivazioni per questo :

1. Quando facciamo il join di 2 tabelle, sql server guarda le righe in una tabella e poi trova la corrispondenza delle righe nella seconda tabella

2.Quando facciamo una query sulla nostra applicazione,  utilizziamo la foreign per caricare i dati


![Schermata del 2023-05-14 20-15-21](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/5b35f29d-d876-4141-bd3b-1044a0332870)

![Schermata del 2023-05-14 20-15-41](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/edc2add3-39cb-4cfd-9c6b-991f4b189a69)


_______________________________________________________________________________________________________



## Why index column order matters ?


Un esempio di creazione è il seguente di index è il seguente.

```
CREATE INDEX IX_Applicants_FirstNameLastName
    ON Applicants(FirstName, LastName , State).

```

La query che vogliamo lanciare è la seguente :


```
SELECT * FROM Applicats WHERE LastName='Davis' AND State='CO';
```

Le colonne sono inserite per ordine , FirstName, LastName,State

Vediamo il piano di esecuzione che fa sql.


![Schermata del 2023-05-17 19-06-59](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/6bce9d57-f50a-489c-b134-2403c71a5150)

Si ha un'operazione di scan sull'indice e questa operazione **SCAN OPERATION** legge l'intero index,
quindi non usa la struttura ad albero dell'index  **TREE STRUCTURE OF THE INDEX** , per trovare la corrispondenza
ma invece raggiunge ogni chiave dell'index per trovare il valore che fa corrispondenza.
Noi invece vogliamo una seek operation perchè significa che sql sta usando tree-strucure, e l'indice è organizzato per trovare la chiave corrispondente, e questo è molto più efficiente.
La ragione per cui sql server usa un'operazione di scan piuttosto che seek operation è perchè la clausula where NON INCLUDE LA PRIMA COLONNA DELL'INDEX, che nel nostro caso è FirstNameColumn,
se non si include la prima colonna dell'index nella clausula where SQL -SERVER , non sarà in grado di usare l'index e eseguirà un'operazione di san sull'intero index


![Schermata del 2023-05-17 19-24-55](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/4c6495e2-eae0-42dd-a083-2c5cdd89f349)


![Schermata del 2023-05-17 19-25-57](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/219505c8-c31d-4472-9148-23fa7ad2284e)

> FIX IT

Per sistemare dobbiamo cambiare l'ordine delle colonne nel nostro index.
Il primo passaggio consiste nel fare il drop dell'indice corrente.

```
DROP INDEX IX_Applicants_FirstNameLastName
   ON Applicants;
```

Ricreiamo nuovamente l'indice.


```
CREATE INDEX IX_Applicants_FirstNameLastName
    ON Applicants( LastName,FirstName , State).

```

Ripropongo la stessa query  :


```
SELECT * FROM Applicats WHERE LastName='Davis' AND State='CO';
```

Possiamo vedere ora che il tipo di operazione che viene svolta è di tipo Seek :


![Schermata del 2023-05-17 19-31-20](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/b8db939f-9cdd-4ae5-a520-992a26d09543)

Questo ha un'efficiente maggiore rispetto a prima.


![Schermata del 2023-05-17 19-32-39](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/e8872790-aa4f-41b9-843b-a0e458de023d)


Estimated SubTree Costo  = 1.42808.

Mentre se attivo le statistiche si ottiene che :



![Schermata del 2023-05-17 19-34-11](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/b1c5f1fa-61df-4c56-af3c-e06fa57ac6b3)


Per creare un indice in maniera corretta devo pensare a come i dati saranno ricercati .


![Schermata del 2023-05-17 19-35-51](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/45284c39-115d-4c8d-b61a-b3c8de87e907)


![Schermata del 2023-05-17 19-36-13](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/3827ce34-3574-4268-a2f9-ba8f83389334)


![Schermata del 2023-05-17 19-37-16](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/19f62f6f-7733-4bb5-9ab6-97c11a0eeac9)



![Schermata del 2023-05-17 19-39-36](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/87d5b78a-e5ae-4000-bb0c-d8fdf732809b)


![Schermata del 2023-05-17 19-39-59](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/65d66331-08d2-4281-8dc0-b40cfc965c55)



![Schermata del 2023-05-17 19-40-22](https://github.com/MrMagicalSoftware/sql-server-performace/assets/98833112/25d1fbfe-4387-48a4-8848-e560b143b935)


## Index Selectivity Explained










