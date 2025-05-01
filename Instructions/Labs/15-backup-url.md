---
lab:
  title: 'Lab 15: Sichern und Wiederherstellen über eine URL'
  module: Plan and implement a high availability and disaster recovery solution
---

# Erstellen von Sicherungen über URLs

**Geschätzte Dauer: 30 Minuten**

Als DBA für AdventureWorks müssen Sie eine Datenbank unter einer URL in Azure sichern und sie nach einem menschlichen Fehler aus dem Azure-Blob-Speicher wiederherstellen.

## Umgebung einrichten

Wenn Ihr virtueller Computer für das Lab bereitgestellt und vorkonfiguriert wurde, sollten Sie die Lab-Dateien im Ordner **C:\LabFiles** finden. *Nehmen Sie sich einen Moment Zeit, um zu überprüfen, ob die Dateien bereits vorhanden sind. Überspringen Sie diesen Abschnitt.* Wenn Sie jedoch Ihren eigenen Computer verwenden oder die Lab-Dateien fehlen, müssen Sie sie von *GitHub* klonen, um fortzufahren.

1. Starten Sie auf dem virtuellen Lab-Computer oder dem lokalen Computer, wenn kein Computer bereitgestellt wurde, eine Visual Studio Code-Sitzung.

1. Öffnen Sie die Befehlspalette (Strg+Umschalt+P) und geben Sie **Git: Clone** ein. Wählen Sie die Option **Git: Clone** aus.

1. Fügen Sie die folgende URL in das Feld **Repository URL** ein und wählen Sie **Eingabe**.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. Speichern Sie das Repository im Ordner **C:\LabFiles** auf dem virtuellen Lab-Computer oder auf Ihrem lokalen Computer, falls kein virtueller Lab-Computer bereitgestellt wurde (erstellen Sie den Ordner, falls er nicht vorhanden ist).

## Wiederherstellen der Datenbank

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

## Konfigurieren von „Sichern unter URL“

1. Starten Sie auf dem virtuellen Lab-Computer oder dem lokalen Computer, wenn kein Computer bereitgestellt wurde, eine Visual Studio Code-Sitzung.

1. Öffnen Sie das geklonte Repository unter **C:\LabFiles\dp-300-database-administrator**.

1. Klicken Sie mit der rechten Maustaste auf den Ordner **Allfiles** und wählen Sie **In integriertem Terminal öffnen**. Dadurch wird ein Terminalfenster an der richtigen Stelle geöffnet.

1. Geben Sie im Terminal Folgendes ein und drücken Sie die **Eingabetaste**.

    ```bash
    az login
    ```

1. Sie werden dann aufgefordert, einen Browser zu öffnen und einen Code einzugeben. Folgen Sie den Anweisungen, um sich bei Ihrem Azure-Konto anzumelden.

1. *Überspringen Sie diesen Schritt, wenn Sie bereits eine Ressourcengruppe haben*. Wenn Sie noch keine Ressourcengruppe haben, erstellen Sie eine, indem Sie den folgenden Befehl im Terminal ausführen. Ersetzen Sie *contoso-rgXXX######* durch einen eindeutigen Namen für Ihre Ressourcengruppe. Der Name muss innerhalb von Azure eindeutig sein. Ersetzen Sie Ihren Standort (-l) durch den Standort Ihrer Ressourcengruppe.

    ```bash
    az group create -n "contoso-rglod#######" -l eastus2
    ```

    Ersetzen Sie **######** durch einige zufällige Zeichen.

1. Geben Sie im Terminal Folgendes ein und drücken Sie die **Eingabetaste**, um ein Speicherkonto zu erstellen. Stellen Sie sicher, dass Sie einen eindeutigen Namen für das Speicherkonto verwenden. *Der Name muss zwischen 3 und 24 Zeichen lang sein und darf nur Zahlen und Kleinbuchstaben enthalten*. Ersetzen Sie *########* durch 8 zufällige numerische Zeichen. Der Name muss innerhalb von Azure eindeutig sein. Ersetzen Sie contoso-rgXXX###### durch den Namen Ihrer Ressourcengruppe. Ersetzen Sie schließlich Ihren Standort (-l) durch den Speicherort Ihrer Ressourcengruppe.

    ```bash
    az storage account create -n "dp300bckupstrg########" -g "contoso-rgXXX########" --kind StorageV2 -l eastus2
    ```

1. Als Nächstes rufen Sie die Schlüssel für Ihr Speicherkonto ab, die Sie in den nachfolgenden Schritten verwenden werden. Führen Sie den folgenden Code im Terminal aus und verwenden Sie den eindeutigen Namen Ihres Speicherkontos und Ihrer Ressourcengruppe.

    ```bash
    az storage account keys list -g contoso-rgXXX######## -n dp300bckupstrg########
    ```

    Ihr Kontoschlüssel ist in den Ergebnissen des obigen Befehls enthalten. Stellen Sie sicher, dass Sie denselben Namen (hinter **-n**) und dieselbe Ressourcengruppe (hinter **-g**) verwenden, die Sie im vorherigen Befehl verwendet haben. Kopieren Sie den zurückgegebenen Wert für **key1** (ohne die Anführungszeichen).

1. Beim Sichern einer Datenbank in SQL Server unter einer URL wird ein Container innerhalb eines Speicherkontos verwendet. In diesem Schritt erstellen Sie einen Container speziell für die Speicherung von Sicherungskopien. Führen Sie dazu die folgenden Befehle aus.

    ```bash
    az storage container create --name "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --fail-on-exist
    ```

    Dabei ist **dp300bckupstrg########** der eindeutige Speicherkontoname, der bei der Erstellung des Speicherkontos verwendet wurde, und **storage_key** ist der zuvor generierte Schlüssel. Die Ausgabe sollte **true** zurückgeben.

1. Um zu überprüfen, ob die Container-Backups ordnungsgemäß erstellt wurden, führen Sie Folgendes aus:

    ```bash
    az storage container list --account-name "dp300bckupstrg########" --account-key "storage_key"
    ```

    Dabei ist **dp300bckupstrg########** der eindeutige Name des Speicherkontos, der bei der Erstellung des Speicherkontos verwendet wird, und **storage_key** ist der generierte Schlüssel.

1. Aus Sicherheitsgründen ist eine Shared Access Signature (SAS) auf Containerebene erforderlich. Führen Sie folgenden Befehl im Terminal aus:

    ```bash
    az storage container generate-sas -n "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --permissions "rwdl" --expiry "date_in_the_future" -o tsv
    ```

    Wobei **dp300bckupstrg########** der eindeutige Name des Speicherkontos ist, der beim Erstellen des Speicherkontos verwendet wurde, **storage_key** der generierte Schlüssel ist und **date_in_the_future** ein Zeitpunkt in der Zukunft ist. **date_in_the_future** muss in UTC sein. Zum Beispiel **2025-12-31T00:00Z**, was bedeutet, dass es am 31. Dezember 2025 um Mitternacht abläuft.

    Die Ausgabe sollte in etwa wie folgt aussehen. Kopieren Sie die gesamte SAS, und fügen Sie sie in **Editor** ein. Sie wird in der nächsten Aufgabe verwendet.

    *se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=c&sig=rnoGlveGql7ILhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D*

## Erstellen von Anmeldeinformationen

Nun, da die Funktionalität konfiguriert ist, können Sie eine Sicherungsdatei als Blob in Azure Storage Account generieren.

1. Starten Sie **SQL Server Management Studio (SSMS)**.

1. Sie werden aufgefordert, eine Verbindung mit SQL Server herzustellen. Vergewissern Sie sich, dass die **Windows-Authentifizierung** ausgewählt ist, und wählen Sie **Verbinden** aus.

1. Wählen Sie **Neue Abfrage** aus.

1. Erstellen Sie die Anmeldeinformationen, die für den Zugriff auf Speicher in der Cloud verwendet werden sollen, mit dem folgenden Transact-SQL-Code. Geben Sie die entsprechenden Werte ein, und wählen Sie dann **Ausführen** aus.

    ```sql
    IF NOT EXISTS  
    (SELECT * 
        FROM sys.credentials  
        WHERE name = 'https://<storage_account_name>.blob.core.windows.net/backups')  
    BEGIN
        CREATE CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]
        WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
        SECRET = '<key_value>'
    END;
    GO  
    ```

    Dabei sind beide Instanzen von **<storage_account_name>** der eindeutige Name des erstellten Speicherkontos und **<key_value>** der Wert, der am Ende der vorherigen Aufgabe generiert wurde, ähnlich wie im folgenden Beispiel:

    *se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=c&sig=rnoGlveGql7ILhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D*

1. Sie können überprüfen, ob die Anmeldedaten erfolgreich erstellt wurden, indem Sie zu **Sicherheit -> Anmeldedaten** im Objekt-Explorer in SSMS navigieren.

1. Wenn Sie sich vertippt haben und die Anmeldeinformationen neu erstellen müssen, können Sie sie mit dem folgenden Befehl löschen, wobei Sie darauf achten müssen, den Namen des Speicherkontos zu ändern:

    ```sql
    -- Only run this command if you need to go back and recreate the credential! 
    DROP CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]  
    ```

## Datenbanksicherung auf URLs

1. Sichern Sie mit SSMS die Datenbank **AdventureWorks2017** auf Azure mit dem folgenden Befehl in Transact-SQL:

    ```sql
    BACKUP DATABASE AdventureWorks2017   
    TO URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak';
    GO 
    ```

    Dabei ist **<storage_account_name>** der erstellte eindeutige Speicherkontoname. 

    Wenn ein Fehler auftritt, überprüfen Sie, ob Sie sich bei der Erstellung der Anmeldeinformationen vertippt haben und ob alles erfolgreich erstellt wurde.

## Überprüfen der Sicherung über Azure CLI

Um festzustellen, ob die Datei tatsächlich in Azure vorhanden ist, können Sie den Storage-Explorer (Vorschau) oder die Azure Cloud Shell verwenden.

1. Führen Sie im Visual Studio-Code-Terminal diesen Azure CLI-Befehl aus:

    ```bash
    az storage blob list -c "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --output table
    ```

    Verwenden Sie denselben eindeutigen Speicherkontonamen (nach dem **--account-name**) und Kontoschlüssel (nach dem **--account-key**), den Sie in den vorherigen Befehlen verwendet haben.

    Wir können bestätigen, dass die Sicherungsdatei erfolgreich erstellt wurde.

## Überprüfen der Sicherung im Speicherbrowser

1. Öffnen Sie in einem Browserfenster das Azure-Portal, suchen Sie nach **Speicherkonten** und wählen Sie diesen Eintrag aus.

1. Wählen Sie den eindeutigen Speicherkontonamen aus, den Sie für die Sicherungen angelegt haben.

1. Wählen Sie im linken Navigationsbereich **Speicherbrowser** aus. Erweitern Sie die Option **Blobcontainer**.

1. Wählen Sie **Sicherungen** aus.

1. Beachten Sie, dass die Sicherungsdatei im Container gespeichert ist.

## Wiederherstellen über URL

In dieser Aufgabe lernen Sie, wie Sie eine Datenbank aus einem Azure-Blob-Speicher wiederherstellen können.

1. Wählen Sie in **SQL Server Management Studio (SSMS)** die Option **Neue Abfrage** aus, fügen Sie dann die folgende Abfrage ein, und führen Sie sie aus.

    ```sql
    USE AdventureWorks2017;
    GO
    SELECT * FROM Person.Address WHERE AddressId = 1;
    GO
    ```

1. Führen Sie diesen Befehl aus, um die Adresse dieser Kundschaft zu ändern.

    ```sql
    UPDATE Person.Address
    SET AddressLine1 = 'This is a human error'
    WHERE AddressId = 1;
    GO
    ```

1. Führen Sie **Schritt 1** erneut aus, um zu überprüfen, ob die Adresse geändert wurde. Stellen Sie sich nun vor, jemand hätte Tausende oder Millionen von Zeilen ohne WHERE-Klausel geändert – oder mit der falschen WHERE-Klausel. Eine der Lösungen beinhaltet die Wiederherstellung der Datenbank aus der letzten verfügbaren Sicherung.

1. Um die Datenbank wiederherzustellen und sie auf den Stand vor der irrtümlichen Änderung des Kundennamens zu bringen, führen Sie Folgendes aus.

    > &#128221; Mit **SET SINGLE_USER WITH ROLLBACK IMMEDIATE** werden alle offenen Transaktionen zurückgesetzt. Dies kann verhindern, dass die Wiederherstellung aufgrund aktiver Verbindungen fehlschlägt.

    ```sql
    USE [master]
    GO

    ALTER DATABASE AdventureWorks2017 SET SINGLE_USER WITH ROLLBACK IMMEDIATE
    GO

    RESTORE DATABASE AdventureWorks2017 
    FROM URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak'
    GO

    ALTER DATABASE AdventureWorks2017 SET MULTI_USER
    GO
    ```

    Dabei ist **<storage_account_name>** der eindeutige Name des Speicherkontos, den Sie erstellt haben.

1. Führen Sie **Schritt 1** erneut aus, um zu überprüfen, ob der Kundenname wiederhergestellt wurde.

Es ist wichtig, die jeweiligen Komponenten und Interaktionen zwischen den einzelnen Komponenten zu verstehen, um Daten mit dem Azure Blob Storage-Dienst zu sichern oder wiederherzustellen.

---

## Bereinigen von Ressourcen

Wenn Sie den Azure SQL Server nicht für andere Zwecke verwenden, können Sie die in diesem Lab erstellten Ressourcen bereinigen.

### Löschen der Ressourcengruppe

Wenn Sie eine neue Ressourcengruppe für dieses Lab erstellt haben, können Sie die Ressourcengruppe löschen, um alle in diesem Lab erstellten Ressourcen zu entfernen.

1. Wählen Sie im Azure-Portal **Ressourcengruppen** im linken Navigationsbereich oder suchen Sie in der Suchleiste nach **Ressourcengruppen** und wählen Sie die Option aus den Ergebnissen aus.

1. Wechseln Sie zur Ressourcengruppe, die Sie für dieses Lab erstellt haben. Die Ressourcengruppe enthält den Azure SQL Server und andere Ressourcen, die in diesem Lab erstellt wurden.

1. Wählen Sie **Ressourcengruppe löschen** aus dem Menü ganz oben aus.

1. Geben Sie im Dialog **Ressourcengruppe löschen** den Namen der zu bestätigenden Ressourcengruppe ein und wählen Sie **Löschen**.

1. Warten Sie, bis die Ressourcengruppe gelöscht wurde.

1. Schließen Sie das Azure-Portal.

### Löschen Sie nur die Lab-Ressourcen

Wenn Sie keine neue Ressourcengruppe für dieses Lab erstellt haben und die Ressourcengruppe und die vorherigen Ressourcen intakt lassen möchten, können Sie die in dieser Übung erstellten Ressourcen weiterhin löschen.

1. Wählen Sie im Azure-Portal **Ressourcengruppen** im linken Navigationsbereich oder suchen Sie in der Suchleiste nach **Ressourcengruppen** und wählen Sie die Option aus den Ergebnissen aus.

1. Wechseln Sie zur Ressourcengruppe, die Sie für dieses Lab erstellt haben. Die Ressourcengruppe enthält den Azure SQL Server und andere Ressourcen, die in diesem Lab erstellt wurden.

1. Wählen Sie alle Ressourcen aus, denen der zuvor im Lab angegebene SQL Server-Name vorangestellt ist.

1. Wählen Sie im Menü oben **Löschen** aus.

1. Im Dialog **Ressourcen löschen** geben Sie **Löschen** ein und wählen **Löschen**.

1. Wählen Sie erneut **Löschen**, um die Löschung der Ressourcen zu bestätigen.

1. Warten Sie, bis die Ressourcen gelöscht wurden.

1. Schließen Sie das Azure-Portal.

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

Sie haben nun gesehen, dass Sie eine Datenbank unter einer URL in Azure sichern und bei Bedarf wiederherstellen können.
