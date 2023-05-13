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


























































