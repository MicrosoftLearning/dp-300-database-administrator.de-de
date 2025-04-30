---
lab:
  title: 'Lab 7: Erkennen und Beheben von Fragmentierungsproblemen'
  module: Monitor and optimize operational resources in Azure SQL
---

# Erkennen und Beheben von Fragmentierungsproblemen

**Geschätzte Dauer**: 20 Minuten

Die Kursteilnehmer planen anhand der in den Lektionen gewonnenen Informationen die Leistungen für ein digitales Transformationsprojekt in AdventureWorks. Sie untersuchen das Azure-Portal und andere Tools und bestimmen, wie sie native Tools zur Identifizierung und Lösung von Leistungsproblemen einsetzen können. Schließlich werden die Teilnehmer in der Lage sein, die Fragmentierung innerhalb der Datenbank zu identifizieren und Schritte zu erlernen, um diese angemessen zu beheben.

Sie wurden als Datenbankadministrator eingestellt, um leistungsbezogene Probleme zu ermitteln und geeignete Lösungen zu erarbeiten, um gefundene Probleme zu beheben. AdventureWorks verkauft seit über einem Jahrzehnt Fahrräder und Fahrradteile direkt an Verbraucher und Händler. In letzter Zeit hat das Unternehmen Leistungseinbußen bei seinen Produkten festgestellt, die zur Bearbeitung von Kundenanfragen verwendet werden. Sie müssen SQL-Tools verwenden, um die Leistungsprobleme zu ermitteln und Methoden zu ihrer Lösung vorzuschlagen.

> &#128221; In diesen Übungen werden Sie aufgefordert, T-SQL-Code zu kopieren und einzufügen. Überprüfen Sie, ob der Code korrekt kopiert wurde, bevor Sie ihn ausführen.

## Umgebung einrichten

Wenn Ihr virtueller Computer für das Lab bereitgestellt und vorkonfiguriert wurde, sollten Sie die Lab-Dateien im Ordner **C:\LabFiles** finden. *Nehmen Sie sich einen Moment Zeit, um zu überprüfen, ob die Dateien bereits vorhanden sind. Überspringen Sie diesen Abschnitt.* Wenn Sie jedoch Ihren eigenen Computer verwenden oder die Lab-Dateien fehlen, müssen Sie sie von *GitHub* klonen, um fortzufahren.

1. Starten Sie auf dem virtuellen Lab-Computer oder dem lokalen Computer, wenn kein Computer bereitgestellt wurde, eine Visual Studio Code-Sitzung.

1. Öffnen Sie die Befehlspalette (Strg+Umschalt+P) und geben Sie **Git: Clone** ein. Wählen Sie die Option **Git: Clone** aus.

1. Fügen Sie die folgende URL in das Feld **Repository URL** ein und wählen Sie **Eingabe**.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. Speichern Sie das Repository im Ordner **C:\LabFiles** auf dem virtuellen Lab-Computer oder auf Ihrem lokalen Computer, falls kein virtueller Lab-Computer bereitgestellt wurde (erstellen Sie den Ordner, falls er nicht vorhanden ist).

---

## Wiederherstellen einer Datenbank

Wenn Sie die **AdventureWorks2017-Datenbank** bereits wiederhergestellt haben, können Sie diesen Abschnitt überspringen.

1. Starten Sie auf dem virtuellen Lab-Computer oder dem lokalen Computer, wenn kein Computer bereitgestellt wurde, eine SQL Server Management Studio-Sitzung (SSMS).

1. Wenn SSMS geöffnet wird, erscheint standardmäßig das Dialogfeld **Mit Server verbinden**. Wählen Sie die Standardinstanz und dann **Verbinden** aus. Möglicherweise müssen Sie das Kontrollkästchen **Serverzertifikat vertrauen** aktivieren.

    > &#128221; Wenn Sie Ihre eigene SQL Server-Instanz verwenden, müssen Sie mithilfe des entsprechenden Serverinstanznamens und der entsprechenden Anmeldeinformationen eine Verbindung damit herstellen.

1. Wählen Sie den Ordner **Datenbanken**, und dann **Neue Abfrage**.

1. Kopieren Sie im neuen Abfragefenster die folgende T-SQL und fügen Sie sie ein. Führen Sie die Abfrage aus, um die Datenbank wiederherzustellen.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\Shared\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\AdventureWorks2017_log.ldf';
    ```

    > &#128221; Sie müssen einen Ordner namens **C:\LabFiles** haben. Wenn Sie diesen Ordner nicht haben, erstellen Sie ihn, oder geben Sie einen anderen Speicherort für die Datenbank- und Sicherungsdateien an.

1. Auf der Registerkarte **Meldungen** sollten Sie eine Meldung sehen, die besagt, dass die Datenbank erfolgreich wiederhergestellt wurde.

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

1. Die Indexfragmentierung kann durch eine Reihe von Faktoren verursacht werden, einschließlich der folgenden:

    - Häufige Aktualisierungen der Tabelle oder des Indexes.
    - Häufiges Einfügen oder Löschen in der Tabelle oder im Index.
    - Seitenteilungen.

    Um den Fragmentierungsgrad der Tabelle „Person.Address“ und ihrer Indizes zu erhöhen, fügen Sie eine große Anzahl von Datensätzen ein und löschen diese wieder. Dazu führen Sie die folgende Abfrage aus.

    Wählen Sie **Neue Abfrage** aus. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

    ```sql
    USE AdventureWorks2017

    GO
    
    -- Insert 60000 records into the Address table    

    INSERT INTO [Person].[Address] 
        ([AddressLine1], [AddressLine2], [City], [StateProvinceID], [PostalCode], [SpatialLocation], [rowguid], [ModifiedDate])
    SELECT 
        'Split Avenue ' + CAST(v1.number AS VARCHAR(10)), 
        'Apt ' + CAST(v2.number AS VARCHAR(10)), 
        'PageSplitTown', 
        100 + (v1.number % 60),  -- 60 different StateProvinceIDs (100-159)
        '88' + RIGHT('000' + CAST(v2.number AS VARCHAR(3)), 3), -- Structured postal codes
        NULL, 
        NEWID(), -- Ensure unique rowguid
        GETDATE()
    FROM master.dbo.spt_values v1
    CROSS JOIN master.dbo.spt_values v2
    WHERE v1.type = 'P' AND v1.number BETWEEN 1 AND 300 
    AND v2.type = 'P' AND v2.number BETWEEN 1 AND 200;
    GO
    
    -- DELETE 25000 records from the Address table
    DELETE FROM [Person].[Address] WHERE AddressID BETWEEN 35001 AND 60000;

    GO

    -- Insert 40000 records into the Address table
    INSERT INTO [Person].[Address] 
        ([AddressLine1], [AddressLine2], [City], [StateProvinceID], [PostalCode], [SpatialLocation], [rowguid], [ModifiedDate])
    SELECT 
        'Fragmented Street ' + CAST(v1.number AS VARCHAR(10)), 
        'Suite ' + CAST(v2.number AS VARCHAR(10)), 
        'FragmentCity', 
        100 + (v1.number % 60),  -- 60 different StateProvinceIDs (100-159)
        '99' + RIGHT('000' + CAST(v2.number AS VARCHAR(3)), 3), -- Structured postal codes
        NULL, 
        NEWID(), -- Ensure a unique rowguid per row
        GETDATE()
    FROM master.dbo.spt_values v1
    CROSS JOIN master.dbo.spt_values v2
    WHERE v1.type = 'P' AND v1.number BETWEEN 1 AND 200 
    AND v2.type = 'P' AND v2.number BETWEEN 1 AND 200;

    GO
    ```

    Diese Abfrage erhöht den Fragmentierungsgrad der Tabelle „Person.Address“ und ihrer Indizes, indem eine große Anzahl von Datensätzen hinzugefügt und gelöscht wird.

1. Führen Sie die erste Abfrage noch einmal aus. Jetzt sollten Sie vier stark fragmentierte Indizes sehen können.

1. Wählen Sie **Neue Abfrage** aus und kopieren Sie den folgenden T-SQL-Code und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

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

    Wählen Sie die Registerkarte **Meldungen** im Ergebnisbereich von SQL Server Management Studio. Notieren Sie sich die Anzahl der logischen Lesevorgänge, die von der Abfrage in der Tabelle **Address** durchgeführt wurden.

## Neuerstellung fragmentierter Indizes

1. Wählen Sie **Neue Abfrage** aus und kopieren Sie den folgenden T-SQL-Code und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

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

1. Wählen Sie eine **Neue Abfrage** und führen Sie die folgende Abfrage aus, um zu bestätigen, dass der Index **IX_Address_StateProvinceID** keine Fragmentierung von mehr als 50 % aufweist.

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

    Der Vergleich der Ergebnisse zeigt, dass die **IX_Address_StateProvinceI**-Fragmentierung von 88 % auf 0 % gesunken ist.

1. Führen Sie die Select-Anweisung aus dem vorherigen Abschnitt erneut aus. Notieren Sie sich die logischen Lesevorgänge auf der Registerkarte **Meldungen** im Bereich **Ergebnisse** von Management Studio. *Wurde die Anzahl der logischen Lesevorgänge geändert, bevor Sie den Index für die Adresstabelle neu erstellt haben*?

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

---

## Bereinigung

Wenn Sie die Datenbank oder die Lab-Dateien nicht für einen anderen Zweck verwenden, können Sie die Objekte bereinigen, die Sie in dieser Übung erstellt haben.

### Löschen des Ordners „C:\LabFiles“

1. Öffnen Sie den **Datei-Explorer** auf dem virtuellen Computer des Labs oder auf Ihrem lokalen Computer, falls kein solcher zur Verfügung gestellt wurde.
1. Navigieren Sie zu **C:\\**.
1. Löschen Sie den Ordner **C:\LabFiles**.

### Löschen der AdventureWorks2017-Datenbank

1. Starten Sie auf dem virtuellen Lab-Computer oder dem lokalen Computer, wenn kein Computer bereitgestellt wurde, eine SQL Server Management Studio-Sitzung (SSMS).
1. Wenn SSMS geöffnet wird, erscheint standardmäßig das Dialogfeld **Mit Server verbinden**. Wählen Sie die Standardinstanz und dann **Verbinden** aus. Möglicherweise müssen Sie das Kontrollkästchen **Serverzertifikat vertrauen** aktivieren.
1. Erweitern Sie im **Objekt-Explorer** den Ordner **Datenbanken**.
1. Klicken Sie mit der rechten Maustaste auf die **AdventureWorks2017**-Datenbank und wählen Sie **Löschen**.
1. Aktivieren Sie im Dialog **Objekt löschen** das Kontrollkästchen **Vorhandene Verbindungen schließen**.
1. Wählen Sie **OK** aus.

---

Sie haben dieses Lab erfolgreich abgeschlossen.

In dieser Übung haben Sie gelernt, wie man einen Index neu aufbaut und logische Lesevorgänge analysiert, um die Abfrageleistung zu verbessern.
