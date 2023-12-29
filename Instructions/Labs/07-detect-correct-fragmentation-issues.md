---
lab:
  title: 'Lab 7: Erkennen und Beheben von Fragmentierungsproblemen'
  module: Monitor and optimize operational resources in Azure SQL
---

# Erkennen und Beheben von Fragmentierungsproblemen

**Geschätzte Dauer**: 15 Minuten

Die Kursteilnehmer planen anhand der in den Lektionen gewonnenen Informationen die Leistungen für ein digitales Transformationsprojekt in AdventureWorks. Sie untersuchen das Azure-Portal und andere Tools und bestimmen, wie sie native Tools zur Identifizierung und Lösung von Leistungsproblemen einsetzen können. Schließlich werden die Teilnehmer in der Lage sein, die Fragmentierung innerhalb der Datenbank zu identifizieren und Schritte zu erlernen, um diese angemessen zu beheben.

Sie wurden als Datenbankadministrator eingestellt, um leistungsbezogene Probleme zu ermitteln und geeignete Lösungen zu erarbeiten, um gefundene Probleme zu beheben. AdventureWorks verkauft seit über einem Jahrzehnt Fahrräder und Fahrradteile direkt an Verbraucher und Händler. In letzter Zeit hat das Unternehmen Leistungseinbußen bei seinen Produkten festgestellt, die zur Bearbeitung von Kundenanfragen verwendet werden. Sie müssen SQL-Tools verwenden, um die Leistungsprobleme zu ermitteln und Methoden zu ihrer Lösung vorzuschlagen.

**Hinweis:** In diesen Übungen werden Sie aufgefordert, T-SQL-Code zu kopieren und einzufügen. Überprüfen Sie, ob der Code korrekt kopiert wurde, bevor Sie ihn ausführen.

## Wiederherstellen einer Datenbank

1. Laden Sie die Sicherungsdatei der Datenbank unter **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak** in den Pfad **C:\LabFiles\Monitor and optimize** auf dem virtuellen Lab-Computer herunter. (Erstellen Sie die Ordnerstruktur, falls sie nicht vorhanden ist.)

    ![Abbildung 03](../images/dp-300-module-07-lab-03.png)

1. Wählen Sie die Windows-Starttaste und geben Sie SSMS ein. Wählen Sie **microsoft SQL Server Management Studio 18** aus der Liste aus.  

    ![Abbildung 01](../images/dp-300-module-01-lab-34.png)

1. Beim Öffnen von SSMS wird das Dialogfeld **Mit Server verbinden** vorab mit dem Standardinstanznamen ausgefüllt. Wählen Sie **Verbinden**.

    ![Abbildung 02](../images/dp-300-module-07-lab-01.png)

1. Wählen Sie den Ordner **Datenbanken** und dann **Neue Abfrage** aus.

    ![Abbildung 03](../images/dp-300-module-07-lab-04.png)

1. Kopieren Sie im Fenster „Neue Abfrage“ den folgenden T-SQL, und fügen Sie ihn ein. Führen Sie die Abfrage aus, um die Datenbank wiederherzustellen.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\Monitor and optimize\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\Monitor and optimize\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\Monitor and optimize\AdventureWorks2017_log.ldf';
    ```

    **Hinweis:** Der Name und der Pfad der Datenbanksicherungsdatei sollten mit der in Schritt 1 heruntergeladenen Datei übereinstimmen, andernfalls wird der Befehl fehlschlagen.

1. Nach beendeter Wiederherstellung sollte eine Erfolgsmeldung angezeigt werden.

    ![Abbildung 03](../images/dp-300-module-07-lab-05.png)

## Untersuchen der Indexfragmentierung

1. Wählen Sie **Neue Abfrage** aus. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

    ```sql
    USE AdventureWorks2017
    GO
    
    SELECT i.name Index_Name
     , avg_fragmentation_in_percent
     , db_name(database_id)
     , i.object_id
     , i.index_id
     , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
     INNER JOIN sys.indexes i ON ps.object_id = i.object_id 
     AND ps.index_id = i.index_id
    WHERE avg_fragmentation_in_percent > 50 -- find indexes where fragmentation is greater than 50%
    ```

    Diese Abfrage meldet alle Indizes, die eine Fragmentierung von mehr als **50 %** aufweisen. Die Abfrage sollte kein Ergebnis liefern.

1. Wählen Sie **Neue Abfrage** aus. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

    ```sql
    USE AdventureWorks2017
    GO
        
    INSERT INTO [Person].[Address]
        ([AddressLine1]
        ,[AddressLine2]
        ,[City]
        ,[StateProvinceID]
        ,[PostalCode]
        ,[SpatialLocation]
        ,[rowguid]
        ,[ModifiedDate])
        
    SELECT AddressLine1,
        AddressLine2, 
        'Amsterdam',
        StateProvinceID, 
        PostalCode, 
        SpatialLocation, 
        newid(), 
        getdate()
    FROM Person.Address;
    
    GO
    ```

    Diese Abfrage erhöht die Fragmentierungsebene der Tabelle „Person.Address“ und ihrer Indizes, indem eine große Anzahl neuer Datensätze hinzugefügt wird.

1. Führen Sie die erste Abfrage noch einmal aus. Jetzt sollten Sie vier stark fragmentierte Indizes sehen können.

    ![Abbildung 03](../images/dp-300-module-07-lab-06.png)

1. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

    ```sql
    SET STATISTICS IO,TIME ON
    GO
        
    USE AdventureWorks2017
    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

    Klicken Sie im Ergebnisbereich von SQL Server Management Studio auf die Registerkarte **Meldungen**. Notieren Sie sich die Anzahl der logischen Lesevorgänge, die von der Abfrage durchgeführt wurden.

    ![Abbildung 03](../images/dp-300-module-07-lab-07.png)

## Neuerstellung fragmentierter Indizes

1. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

    ```sql
    USE AdventureWorks2017
    GO
    
    ALTER INDEX [IX_Address_StateProvinceID] ON [Person].[Address] REBUILD PARTITION = ALL 
    WITH (PAD_INDEX = OFF, 
        STATISTICS_NORECOMPUTE = OFF, 
        SORT_IN_TEMPDB = OFF, 
        IGNORE_DUP_KEY = OFF, 
        ONLINE = OFF, 
        ALLOW_ROW_LOCKS = ON, 
        ALLOW_PAGE_LOCKS = ON)
    ```

1. Führen Sie folgende Abfrage aus, um zu bestätigen, dass der Index **IX_Address_StateProvinceID** keine Fragmentierung von mehr als 50 Prozent mehr aufweist.

    ```sql
    USE AdventureWorks2017
    GO
        
    SELECT DISTINCT i.name Index_Name
        , avg_fragmentation_in_percent
        , db_name(database_id)
        , i.object_id
        , i.index_id
        , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
        INNER JOIN sys.indexes i ON (ps.object_id = i.object_id AND ps.index_id = i.index_id)
    WHERE i.name = 'IX_Address_StateProvinceID'
    ```

    Der Vergleich der Ergebnisse zeigt, dass die Fragmentierung von 81 % auf 0 gesunken ist.

1. Führen Sie die Select-Anweisung aus dem vorherigen Abschnitt erneut aus. Notieren Sie sich die logischen Lesevorgänge auf der Registerkarte **Meldungen** im Bereich **Ergebnisse** von Management Studio. Hat sich die Anzahl der logischen Lesevorgänge im Vergleich zu der Zeit vor der Indexneuerstellung geändert?

    ```sql
    SET STATISTICS IO,TIME ON
    GO
        
    USE AdventureWorks2017
    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

Da der Index neu erstellt wurde, sollte er nun möglichst effizient sein, und die Anzahl der logischen Lesevorgänge sollte kleiner ausfallen. Sie haben nun gesehen, dass sich die Indexwartung auf die Abfrageleistung auswirken kann.

In dieser Übung haben Sie gelernt, wie man einen Index neu aufbaut und logische Lesevorgänge analysiert, um die Abfrageleistung zu verbessern.
