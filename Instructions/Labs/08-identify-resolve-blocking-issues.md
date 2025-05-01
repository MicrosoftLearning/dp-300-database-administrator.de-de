---
lab:
  title: 'Lab 8: Ermitteln und Beheben von Blockierproblemen'
  module: Optimize query performance in Azure SQL
---

# Ermitteln und Beheben von Blockierproblemen

**Geschätzte Dauer**: 15 Minuten

Die Kursteilnehmer planen anhand der in den Lektionen gewonnenen Informationen die Leistungen für ein digitales Transformationsprojekt in AdventureWorks. Sie untersuchen das Azure-Portal und andere Tools und bestimmen, wie sie native Tools zur Identifizierung und Lösung von Leistungsproblemen einsetzen können. Abschließend werden die Teilnehmer in der Lage sein, Blockierprobleme zu erkennen und angemessen zu lösen.

Sie wurden als Datenbankadministrator eingestellt, um leistungsbezogene Probleme zu ermitteln und geeignete Lösungen zu erarbeiten, um gefundene Probleme zu beheben. Sie müssen die Leistungsprobleme untersuchen und Methoden zu ihrer Behebung vorschlagen.

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

## Ausführen blockierter Abfrageberichte

1. Wählen Sie **Neue Abfrage** aus. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

    ```sql
    USE MASTER

    GO

    CREATE EVENT SESSION [Blocking] ON SERVER 
    ADD EVENT sqlserver.blocked_process_report(
    ACTION(sqlserver.client_app_name,sqlserver.client_hostname,sqlserver.database_id,sqlserver.database_name,sqlserver.nt_username,sqlserver.session_id,sqlserver.sql_text,sqlserver.username))
    ADD TARGET package0.ring_buffer
    WITH (MAX_MEMORY=4096 KB, EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS, MAX_DISPATCH_LATENCY=30 SECONDS, MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE, TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)

    GO

    -- Start the event session 
    ALTER EVENT SESSION [Blocking] ON SERVER 
    STATE = start;

    GO
    ```

    Der T-SQL-Code oben erstellt eine Sitzung für ein erweitertes Ereignis, bei dem blockierende Ereignisse erfasst werden. Die Daten beinhalten die folgenden Elemente:

    - Name der Clientanwendung
    - Clienthostname
    - Datenbank-ID
    - Datenbankname
    - NT-Benutzername
    - Sitzungs-ID
    - T-SQL-Text
    - Benutzername

1. Wählen Sie **Neue Abfrage** aus. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

    ```sql
    EXEC sys.sp_configure N'show advanced options', 1

    RECONFIGURE WITH OVERRIDE;

    GO
    EXEC sp_configure 'blocked process threshold (s)', 60

    RECONFIGURE WITH OVERRIDE;

    GO
    ```

    > &#128221; Beachten Sie, dass der obige Befehl den Schwellenwert in Sekunden angibt, bei dem Berichte über blockierte Prozesse erstellt werden. Daher müssen wir in dieser Lektion nicht so lange warten, bis der *blocked_process_report* ausgelöst wird.

1. Wählen Sie **Neue Abfrage** aus. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

    ```sql
    USE AdventureWorks2017

    GO

    BEGIN TRANSACTION
        UPDATE Person.Person 
        SET LastName = LastName;

    GO
    ```

1. Öffnen Sie ein weiteres Abfragefenster, indem Sie auf die Schaltfläche **Neue Abfrage** klicken. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das neue Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

    ```sql
    USE AdventureWorks2017

    GO

    SELECT TOP (1000) [LastName]
      ,[FirstName]
      ,[Title]
    FROM Person.Person
    WHERE FirstName = 'David'
    ```

    > &#128221; Beachten Sie, dass diese Abfrage keine Ergebnisse liefert und unendlich lange zu laufen scheint.

1. Erweitern Sie im **Objekt-Explorer** die Optionen **Verwaltung** -> **Erweiterte Ereignisse** -> **Sitzungen**.

    Beachten Sie, dass das soeben erstellte erweiterte Ereignis *Blockieren* in der Liste enthalten ist.

1. Erweitern Sie das erweiterte Ereignis *Sperrung* und klicken Sie mit der rechten Maustaste auf **Paket0.ring_buffer**. Wählen Sie **Zieldaten anzeigen**.

1. Wählen Sie den aufgelisteten Link aus.

1. Die XML-Datei zeigt Ihnen, welche Prozesse blockiert werden und welcher Prozess die Blockierung verursacht. Sie können die Abfragen sehen, die in diesem Prozess ausgeführt wurden, sowie Systeminformationen. Beachten Sie, dass die Sitzungs-IDs (SPID) angezeigt werden.

1. Alternativ können Sie die folgende Abfrage ausführen, um Sitzungen zu identifizieren, die andere Sitzungen blockieren, einschließlich einer Liste der Sitzungs-IDs, die pro *session_id* blockiert werden. Öffnen Sie ein Fenster **Neue Abfrage**, kopieren Sie den folgenden T-SQL-Code und fügen Sie ihn ein, und wählen Sie **Ausführen**.

    ```sql
    WITH cteBL (session_id, blocking_these) AS 
    (SELECT s.session_id, blocking_these = x.blocking_these FROM sys.dm_exec_sessions s 
    CROSS APPLY    (SELECT isnull(convert(varchar(6), er.session_id),'') + ', '  
                    FROM sys.dm_exec_requests as er
                    WHERE er.blocking_session_id = isnull(s.session_id ,0)
                    AND er.blocking_session_id <> 0
                    FOR XML PATH('') ) AS x (blocking_these)
    )
    SELECT s.session_id, blocked_by = r.blocking_session_id, bl.blocking_these
    , batch_text = t.text, input_buffer = ib.event_info, * 
    FROM sys.dm_exec_sessions s 
    LEFT OUTER JOIN sys.dm_exec_requests r on r.session_id = s.session_id
    INNER JOIN cteBL as bl on s.session_id = bl.session_id
    OUTER APPLY sys.dm_exec_sql_text (r.sql_handle) t
    OUTER APPLY sys.dm_exec_input_buffer(s.session_id, NULL) AS ib
    WHERE blocking_these is not null or r.blocking_session_id > 0
    ORDER BY len(bl.blocking_these) desc, r.blocking_session_id desc, r.session_id;
    ```

    > &#128221; Beachten Sie, dass die obige Abfrage dieselben SPIDs wie der XML-Code zurückgibt.

1. Klicken Sie mit der rechten Maustaste auf das erweiterte Ereignis mit dem Namen **Blockieren**, und wählen Sie dann **Sitzung beenden** aus.

1. Navigieren Sie zurück zu der Abfragesitzung, die die Blockierung verursacht, und geben Sie `ROLLBACK TRANSACTION` in der Zeile unter der Abfrage ein. Markieren Sie die Option `ROLLBACK TRANSACTION`, und wählen Sie **Ausführen** aus.

1. Navigieren Sie zurück zur Abfragesitzung, die blockiert wurde. Sie werden feststellen, dass die Abfrage nun abgeschlossen wurde.

## Aktivieren der Momentaufnahme-Isolationsstufe „READ COMMITTED“

1. Klicken Sie in SQL Server Management Studio auf **Neue Abfrage**. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf die Schaltfläche **Ausführen**, um diese Abfrage auszuführen.

    ```sql
    USE master

    GO
    
    ALTER DATABASE AdventureWorks2017 SET READ_COMMITTED_SNAPSHOT ON WITH ROLLBACK IMMEDIATE;

    GO
    ```

1. Führen Sie die Abfrage, die die Blockierung verursacht hat, in einem neuen Abfrage-Editor erneut aus. *Führen Sie nicht den Befehl „ROLLBACK TRANSACTION“ aus*.

    ```sql
    USE AdventureWorks2017
    GO
    
    BEGIN TRANSACTION
        UPDATE Person.Person 
        SET LastName = LastName;
    GO
    ```

1. Führen Sie die Abfrage, die blockiert wurde, in einem neuen Abfrage-Editor erneut aus.

    ```sql
    USE AdventureWorks2017
    GO
    
    SELECT TOP (1000) [LastName]
     ,[FirstName]
     ,[Title]
    FROM Person.Person
    WHERE firstname = 'David'
    ```

    Warum wird dieselbe Abfrage abgeschlossen, während sie in der vorherigen Aufgabe durch die Aktualisierungsanweisung blockiert wurde?

    Die Isolationsebene Read Commit Snapshot ist eine optimistische Form der Transaktionsisolation, und die letzte Abfrage zeigt die letzte festgeschriebene Version der Daten, anstatt blockiert zu werden.

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

In dieser Übung haben Sie gelernt, wie Sie blockierte Sitzungen identifizieren und solche Szenarien entschärfen können.
