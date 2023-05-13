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




















