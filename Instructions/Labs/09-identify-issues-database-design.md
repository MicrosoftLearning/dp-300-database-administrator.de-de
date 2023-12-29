---
lab:
  title: 'Lab 9: Ermitteln von Problemen beim Datenbankentwurf'
  module: Optimize query performance in Azure SQL
---

# Ermitteln von Problemen beim Datenbankentwurf

**Geschätzte Dauer**: 15 Minuten

Die Kursteilnehmer planen anhand der in den Lektionen gewonnenen Informationen die Leistungen für ein digitales Transformationsprojekt in AdventureWorks. Sie untersuchen das Azure-Portal und andere Tools und bestimmen, wie sie native Tools zur Identifizierung und Lösung von Leistungsproblemen einsetzen können. Schließlich sind die Teilnehmer in der Lage, ein Datenbankdesign auf Probleme mit der Normalisierung, der Datentypauswahl und dem Indexdesign zu überprüfen.

Sie wurden als Datenbankadministrator eingestellt, um leistungsbezogene Probleme zu ermitteln und geeignete Lösungen zu erarbeiten, um gefundene Probleme zu beheben. AdventureWorks verkauft seit über einem Jahrzehnt Fahrräder und Fahrradteile direkt an Verbraucher und Händler. Ihre Aufgabe besteht darin, Probleme bei der Abfrageleistung zu identifizieren und diese mithilfe der in diesem Modul erlernten Methoden zu beheben.

**Hinweis:** In diesen Übungen werden Sie aufgefordert, T-SQL-Code zu kopieren und einzufügen. Überprüfen Sie, ob der Code korrekt kopiert wurde, bevor Sie ihn ausführen.

## Wiederherstellen einer Datenbank

1. Laden Sie die Sicherungsdatei der Datenbank unter **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak** in den Pfad **C:\LabFiles\Monitor and optimize** auf dem virtuellen Lab-Computer herunter. (Erstellen Sie die Ordnerstruktur, falls sie nicht vorhanden ist.)

    ![Abbildung 03](../images/dp-300-module-07-lab-03.png)

1. Wählen Sie die Windows-Starttaste und geben Sie SSMS ein. Wählen Sie **Microsoft SQL Server Management Studio 18** aus der Liste aus.  

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

## Untersuchen der Abfrage und Ermitteln des Problems

1. Wählen Sie **Neue Abfrage** aus. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das Abfragefenster ein. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

    ```sql
    USE AdventureWorks2017
    GO
    
    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

1. Wählen Sie das Symbol **Tatsächlichen Ausführungsplan einbeziehen** wie unten gezeigt, bevor Sie die Abfrage ausführen oder drücken Sie **STRG+M**. Dadurch wird der Ausführungsplan beim Ausführen der Abfrage angezeigt. Klicken Sie auf **Ausführen**, um die Abfrage auszuführen.

    ![Abbildung 01](../images/dp-300-module-09-lab-01.png)

1. Navigieren Sie zum Ausführungsplan. Wählen Sie dazu im Ergebnisbereich die Registerkarte **Ausführungsplan**. Zeigen Sie im Ausführungsplan mit dem Mauszeiger auf den `SELECT`-Operator. Es wird eine Warnmeldung angezeigt, die durch ein Ausrufezeichen in einem gelben Dreieck gekennzeichnet ist (siehe unten). Lesen Sie den Inhalt der Warnmeldung.

    ![Abbildung 02](../images/dp-300-module-09-lab-02.png)

## Ermitteln von Möglichkeiten zum Korrigieren der Warnmeldung

Die Tabellenstruktur *[HumanResources].[Employee]* wird in der folgenden DDL-Anweisung (Data Definition Language) angezeigt. Überprüfen Sie anhand der DDL-Anweisung die Felder, die in der vorherigen SQL-Abfrage verwendet werden. Achten Sie dabei auf die Typen.

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

        Sie werden feststellen, dass der Wert, der in der `WHERE`-Klausel mit der Spalte *NationalIDNumber* verglichen wird, als Zahl verglichen wird, da **14417807** keine Zeichenfolge mit Anführungszeichen ist. 

        Wenn Sie die Tabellenstruktur untersuchen, werden Sie feststellen, dass für die Spalte *NationalIDNumber* anstelle eines `INT`-Datentyps der Datentyp `NVARCHAR` verwendet wird. Aufgrund dieser Inkonsistenz konvertiert der Datenbankoptimierer die Zahl implizit in ein `NVARCHAR`-Wert konvertiert, was die Abfrageleistung zusätzlich beeinträchtigt, da ein suboptimaler Plan erstellt wird.

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

    ![Abbildung 03](../images/dp-300-module-09-lab-03.png)

    **Hinweis:** Die Warnung nicht mehr angezeigt, und der Abfrageplan wurde verbessert. Durch Ändern der `WHERE`-Klausel, sodass der mit der *NationalIDNumber*-Spalte verglichene Wert mit dem Datentyp der Spalte in der Tabelle übereinstimmt, konnte der Optimierer die implizite Konvertierung beseitigen.

### Den Datentyp ändern

1. Wir können die Warnung vor der impliziten Konvertierung auch beheben, indem wir die Tabellenstruktur ändern.

    Kopieren Sie als Versuch zur Korrektur des Index die folgende Abfrage, und fügen Sie sie in ein neues Abfragefenster ein. Auf diese Weise wird der Datentyp der Spalte geändert. Klicken Sie auf **Ausführen**, oder drücken Sie <kbd>F5</kbd>, um zu versuchen, die Abfrage auszuführen.

    ```sql
    ALTER TABLE [HumanResources].[Employee] ALTER COLUMN [NationalIDNumber] INT NOT NULL;
    ```

    Das Ändern des Datentyps der Spalte *NationalIDNumber* in INT würde das Konvertierungsproblem lösen. Doch diese Änderung verursacht ein neues Problem, das Sie als Datenbankadministrator lösen müssen.

    ![Abbildung 04](../images/dp-300-module-09-lab-04.png)

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

1. Alternativ können Sie die folgende Abfrage ausführen, um zu bestätigen, dass der Datentyp erfolgreich geändert wurde.

    ```sql
    SELECT c.name, t.name
    FROM sys.all_columns c INNER JOIN sys.types t
        ON (c.system_type_id = t.user_type_id)
    WHERE OBJECT_ID('[HumanResources].[Employee]') = c.object_id
        AND c.name = 'NationalIDNumber'
    ```
    
    ![Abbildung 05](../images/dp-300-module-09-lab-05.png)
    
1. Nun überprüfen wir den Ausführungsplan. Führen Sie die ursprüngliche Abfrage erneut ohne die Anführungszeichen aus.

    ```sql
    USE AdventureWorks2017
    GO

    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

    ![Abbildung 06](../images/dp-300-module-09-lab-06.png)

    Prüfen Sie den Abfrageplan. Sie stellen fest, dass Sie jetzt eine Ganzzahl zum Filtern nach *NationalIDNumber* verwenden können, ohne Warnung der impliziten Konvertierung auszulösen. Der SQL-Abfrageoptimierer kann nun den optimalen Plan generieren und ausführen.

In dieser Übung haben Sie gelernt, wie Sie Abfrageprobleme, die durch implizite Datentypkonvertierungen verursacht werden, erkennen und wie Sie diese beheben können, um den Abfrageplan zu verbessern.
