---
lab:
  title: 'Lab 9: Ermitteln von Problemen beim Datenbankentwurf'
  module: Optimize query performance in Azure SQL
---

# Ermitteln von Problemen beim Datenbankentwurf

**Geschätzte Dauer**: 15 Minuten

Die Kursteilnehmer planen anhand der in den Lektionen gewonnenen Informationen die Leistungen für ein digitales Transformationsprojekt in AdventureWorks. Sie untersuchen das Azure-Portal und andere Tools und bestimmen, wie sie native Tools zur Identifizierung und Lösung von Leistungsproblemen einsetzen können. Schließlich sind die Teilnehmer in der Lage, ein Datenbankdesign auf Probleme mit der Normalisierung, der Datentypauswahl und dem Indexdesign zu überprüfen.

Sie wurden als Datenbankadministrator eingestellt, um leistungsbezogene Probleme zu ermitteln und geeignete Lösungen zu erarbeiten, um gefundene Probleme zu beheben. AdventureWorks verkauft seit über einem Jahrzehnt Fahrräder und Fahrradteile direkt an Verbraucher und Händler. Ihre Aufgabe besteht darin, Probleme bei der Abfrageleistung zu identifizieren und diese mithilfe der in diesem Modul erlernten Methoden zu beheben.

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

## Untersuchen der Abfrage und Ermitteln des Problems

1. Wählen Sie **Neue Abfrage** aus. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

    ```sql
    USE AdventureWorks2017

    GO
    
    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

1. Wählen Sie das Symbol **Aktuellen Ausführungsplan einschließen** rechts neben der Schaltfläche **Ausführen**, bevor Sie die Abfrage ausführen, oder drücken Sie **STRG + M**. Dadurch wird der Ausführungsplan beim Ausführen der Abfrage angezeigt. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

1. Navigieren Sie zum Ausführungsplan. Wählen Sie dazu im Ergebnisbereich die Registerkarte **Ausführungsplan**. Sie werden feststellen, dass der **SELECT-Operator** ein gelbes Dreieck mit einem Ausrufezeichen enthält. Dies gibt an, dass dem Operator eine Warnmeldung zugeordnet ist. Bewegen Sie den Mauszeiger über das Warnsymbol, um die Meldung zu sehen und die Warnmeldung zu lesen.

    > &#128221; Die Warnmeldung gibt an, dass in der Abfrage eine implizite Konvertierung vorhanden ist. Dies bedeutet, dass der SQL Server-Abfrageoptimierer den Datentyp einer der Spalten in der Abfrage in einen anderen Datentyp konvertieren musste, um die Abfrage auszuführen.

## Ermitteln von Möglichkeiten zum Korrigieren der Warnmeldung

Die Tabellenstruktur *[HumanResources].[Employee]* wird durch die folgende DDL-Anweisung (Data Definition Language) definiert. Überprüfen Sie anhand der DDL-Anweisung die Felder, die in der vorherigen SQL-Abfrage verwendet werden. Achten Sie dabei auf die Typen.

```sql
CREATE TABLE [HumanResources].[Employee](
     [BusinessEntityID] [int] NOT NULL,
     [NationalIDNumber] [nvarchar](15) NOT NULL,
     [LoginID] [nvarchar](256) NOT NULL,
     [OrganizationNode] [hierarchyid] NULL,
     [OrganizationLevel] AS ([OrganizationNode].[GetLevel]()),
     [JobTitle] [nvarchar](50) NOT NULL,
     [BirthDate] [date] NOT NULL,
     [MaritalStatus] [nchar](1) NOT NULL,
     [Gender] [nchar](1) NOT NULL,
     [HireDate] [date] NOT NULL,
     [SalariedFlag] [dbo].[Flag] NOT NULL,
     [VacationHours] [smallint] NOT NULL,
     [SickLeaveHours] [smallint] NOT NULL,
     [CurrentFlag] [dbo].[Flag] NOT NULL,
     [rowguid] [uniqueidentifier] ROWGUIDCOL NOT NULL,
     [ModifiedDate] [datetime] NOT NULL
) ON [PRIMARY]
```

1. Welche Änderung würden Sie gemäß der im Ausführungsplan angezeigten Warnmeldung empfehlen?

    1. Identifizieren Sie das Feld, das die implizite Konvertierung verursacht, sowie die Ursache dafür.
    1. Überprüfen Sie die Abfrage:

        ```sql
        SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
        FROM HumanResources.Employee
        WHERE NationalIDNumber = 14417807;
        ```

        Sie werden feststellen, dass der Wert, der mit der Spalte *NationalIDNumber* in der Klausel **WHERE** verglichen wird, als Zahl verglichen wird, da **14417807** nicht in einer Zeichenfolge in Anführungszeichen steht.

        Nach Prüfung der Tabellenstruktur werden Sie feststellen, dass die Spalte *NationalIDNumber* den Datentyp **NVARCHAR** und nicht den Datentyp **INT** verwendet. Diese Inkonsistenz veranlasst den Datenbankoptimierer, die Zahl implizit in einen *NVARCHAR*-Wert umzuwandeln, was zu einem zusätzlichen Mehraufwand bei der Abfrageleistung führt, indem ein suboptimaler Plan erstellt wird.

Es gibt zwei Ansätze, die wir implementieren können, um die implizite Konvertierungswarnung zu beheben. In den nächsten Schritten werden wir beide Ansätze untersuchen.

### Den Code ändern

1. Wie würden Sie den Code ändern, um die implizite Konvertierung zu beheben? Den Code ändern und die Abfrage erneut ausführen.

    Falls noch nicht aktiviert, aktivieren Sie die Funktion **Tatsächlichen Ausführungsplan einschließen** (**STRG+M**). 

    In diesem Szenario genügt das Hinzufügen eines einfachen Anführungszeichens auf jeder Seite des Wertes, um ihn von einem Zahlen- in ein Zeichenformat zu ändern. Lassen Sie das Abfragefenster für diese Abfrage geöffnet.

    Führen Sie die aktualisierte SQL-Abfrage aus:

    ```sql
    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = '14417807';
    ```

    > &#128221; Beachten Sie, dass die Warnmeldung jetzt verschwunden ist und der Abfrageplan verbessert wurde. Durch Ändern der *WHERE*-Klausel, so dass der Wert, der mit der *NationalIDNumber*-Spalte verglichen wird, mit dem Datentyp der Spalte in der Tabelle übereinstimmt, konnte der Optimierer die implizite Konvertierung beseitigen und einen besseren Plan erstellen.

### Den Datentyp ändern

1. Wir können die Warnung vor der impliziten Konvertierung auch beheben, indem wir die Tabellenstruktur ändern.

    Kopieren Sie als Versuch zur Korrektur des Index die folgende Abfrage, und fügen Sie sie in ein neues Abfragefenster ein. Auf diese Weise wird der Datentyp der Spalte geändert. Klicken Sie auf **Ausführen**, oder drücken Sie <kbd>F5</kbd>, um zu versuchen, die Abfrage auszuführen.

    ```sql
    ALTER TABLE [HumanResources].[Employee] ALTER COLUMN [NationalIDNumber] INT NOT NULL;
    ```

    Das Ändern des Datentyps der Spalte *NationalIDNumber* in INT würde das Konvertierungsproblem lösen. Doch diese Änderung verursacht ein neues Problem, das Sie als Datenbankadministrator lösen müssen. Wenn Sie die obige Abfrage ausführen, wird die folgende Fehlermeldung angezeigt:

    <span style="color:red">Msg 5074, Level 16, Sate 1, Line1  The index 'AK_Employee_NationalIDNumber' is dependent on column 'NationalIDNumber  Msg 4922, Level 16, State 9, Line 1  ALTER TABLE ALTER COLUMN NationalIDNumber failed because one or more objects access this column</span>

    Die Spalte *NationalIDNumber* ist Teil eines bereits vorhandenen nicht gruppierten Index. Daher muss der Index neu erstellt werden, um den Datentyp zu ändern. **Dies könnte zu einer längeren Ausfallzeit in der Produktion führen und unterstreicht noch einmal die Wichtigkeit der Auswahl der richtigen Datentypen in Ihrem Entwurf.**

1. Kopieren Sie zur Lösung dieses Problems den folgenden Code, und fügen Sie ihn in Ihr Abfragefenster ein. Klicken Sie anschließend auf **Ausführen**, um den Code auszuführen.

    ```sql
    USE AdventureWorks2017

    GO
    
    --Dropping the index first
    DROP INDEX [AK_Employee_NationalIDNumber] ON [HumanResources].[Employee]

    GO

    --Changing the column data type to resolve the implicit conversion warning
    ALTER TABLE [HumanResources].[Employee] ALTER COLUMN [NationalIDNumber] INT NOT NULL;

    GO

    --Recreating the index
    CREATE UNIQUE NONCLUSTERED INDEX [AK_Employee_NationalIDNumber] ON [HumanResources].[Employee]( [NationalIDNumber] ASC );

    GO
    ```

1. Führen Sie die folgende Abfrage aus, um zu bestätigen, dass der Datentyp erfolgreich geändert wurde.

    ```sql
    SELECT c.name, t.name
    FROM sys.all_columns c INNER JOIN sys.types t
        ON (c.system_type_id = t.user_type_id)
    WHERE OBJECT_ID('[HumanResources].[Employee]') = c.object_id
        AND c.name = 'NationalIDNumber'
    ```

1. Nun überprüfen wir den Ausführungsplan. Führen Sie die ursprüngliche Abfrage erneut ohne die Anführungszeichen aus.

    ```sql
    USE AdventureWorks2017
    GO

    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

     Prüfen Sie den Abfrageplan. Sie stellen fest, dass Sie jetzt eine Ganzzahl zum Filtern nach *NationalIDNumber* verwenden können, ohne Warnung der impliziten Konvertierung auszulösen. Der SQL-Abfrageoptimierer kann nun den optimalen Plan generieren und ausführen.

>&#128221; Während das Ändern des Datentyps einer Spalte implizite Konvertierungsprobleme beheben kann, ist dies nicht immer die beste Lösung. In diesem Fall hätte die Änderung des Datentyps der Spalte *NationalIDNumber* in einen **INT**-Datentyp Ausfallzeiten in der Produktion verursacht, da der Index für diese Spalte gelöscht und neu erstellt werden müsste. Es ist wichtig, die Auswirkungen einer Änderung des Datentyps einer Spalte auf bestehende Abfragen und Indizes zu berücksichtigen, bevor Sie Änderungen vornehmen. Darüber hinaus kann es andere Abfragen geben, die sich darauf verlassen, dass die Spalte *NationalIDNumber* ein **NVARCHAR**-Datentyp ist, sodass eine Änderung des Datentyps diese Abfragen unterbrechen könnte.

---

## Bereinigung

Wenn Sie die Datenbank oder die Lab-Dateien nicht für einen anderen Zweck verwenden, können Sie die Objekte bereinigen, die Sie in dieser Übung erstellt haben.

### Löschen des Ordners „C:\LabFiles“

1. Öffnen Sie den **Datei-Explorer** auf dem virtuellen Computer des Labs oder auf Ihrem lokalen Computer, falls kein solcher zur Verfügung gestellt wurde.
1. Navigieren Sie zu **C:\\**.
1. Löschen Sie den Ordner **C:\LabFiles**.

## Löschen der AdventureWorks2017-Datenbank

1. Starten Sie auf dem virtuellen Lab-Computer oder dem lokalen Computer, wenn kein Computer bereitgestellt wurde, eine SQL Server Management Studio-Sitzung (SSMS).
1. Wenn SSMS geöffnet wird, erscheint standardmäßig das Dialogfeld **Mit Server verbinden**. Wählen Sie die Standardinstanz und dann **Verbinden** aus. Möglicherweise müssen Sie das Kontrollkästchen **Serverzertifikat vertrauen** aktivieren.
1. Erweitern Sie im **Objekt-Explorer** den Ordner **Datenbanken**.
1. Klicken Sie mit der rechten Maustaste auf die **AdventureWorks2017**-Datenbank und wählen Sie **Löschen**.
1. Aktivieren Sie im Dialog **Objekt löschen** das Kontrollkästchen **Vorhandene Verbindungen schließen**.
1. Wählen Sie **OK** aus.

---

Sie haben dieses Lab erfolgreich abgeschlossen.

In dieser Übung haben Sie gelernt, wie Sie Abfrageprobleme, die durch implizite Datentypkonvertierungen verursacht werden, erkennen und wie Sie diese beheben können, um den Abfrageplan zu verbessern.
