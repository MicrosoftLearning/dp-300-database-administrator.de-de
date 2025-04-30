---
lab:
  title: 'Lab 13: Einsatz eines Runbooks zur Automatisierung der Neuerstellung von Indizes'
  module: Automate database tasks for Azure SQL
---

# Einsatz eines Runbooks zur Automatisierung der Neuerstellung von Indizes

**Geschätzte Dauer: 30 Minuten**

Sie wurden als Senior Data Engineer eingestellt, um die Automatisierung des täglichen Betriebs der Datenbankverwaltung voranzubringen. Diese Automatisierung soll sicherstellen, dass die Datenbanken für AdventureWorks weiterhin mit maximaler Leistung betrieben werden und Methoden für die Alarmierung auf der Grundlage bestimmter Kriterien bereitstellen. AdventureWorks nutzt SQL Server in Angeboten mit Infrastructure-as-a-Service- (IaaS) als auch Platform-as-a-Service- (PaaS).

> &#128221; Bei diesen Übungen müssen Sie möglicherweise T-SQL-Code kopieren und einfügen und vorhandene SQL-Ressourcen nutzen. Überprüfen Sie, ob der Code korrekt kopiert wurde, bevor Sie ihn ausführen.

## Umgebung einrichten

Wenn Ihr virtueller Computer für das Lab bereitgestellt und vorkonfiguriert wurde, sollten Sie die Lab-Dateien im Ordner **C:\LabFiles** finden. *Nehmen Sie sich einen Moment Zeit, um zu überprüfen, ob die Dateien bereits vorhanden sind. Überspringen Sie diesen Abschnitt.* Wenn Sie jedoch Ihren eigenen Computer verwenden oder die Lab-Dateien fehlen, müssen Sie sie von *GitHub* klonen, um fortzufahren.

1. Starten Sie auf dem virtuellen Lab-Computer oder dem lokalen Computer, wenn kein Computer bereitgestellt wurde, eine Visual Studio Code-Sitzung.

1. Öffnen Sie die Befehlspalette (Strg+Umschalt+P) und geben Sie **Git: Clone** ein. Wählen Sie die Option **Git: Clone** aus.

1. Fügen Sie die folgende URL in das Feld **Repository URL** ein und wählen Sie **Eingabe**.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. Speichern Sie das Repository im Ordner **C:\LabFiles** auf dem virtuellen Lab-Computer oder auf Ihrem lokalen Computer, falls kein virtueller Lab-Computer bereitgestellt wurde (erstellen Sie den Ordner, falls er nicht vorhanden ist).

## Einrichten Ihres SQL Server in Azure

Melden Sie sich bei Azure an, und überprüfen Sie, ob Sie über eine vorhandene Azure SQL Server-Instanz verfügen, die in Azure ausgeführt wird. *Überspringen Sie diesen Abschnitt, wenn Sie bereits eine SQL Server-Instanz in Azure ausführen*.

1. Starten Sie auf dem virtuellen Lab-Computer oder Ihrem lokalen Computer (falls kein virtueller Computer bereitgestellt wurde) eine Visual Studio Code-Sitzung und navigieren Sie zum geklonten Repository aus dem vorherigen Abschnitt.

1. Klicken Sie mit der rechten Maustaste auf den Ordner **/Allfiles/Labs** und wählen Sie **In integriertem Terminal öffnen** aus.

1. Verbinden wir uns mit Azure über die Azure CLI. Geben Sie den folgenden Befehl ein und wählen Sie **Eingabe**.

    ```bash
    az login
    ```

    > &#128221; Beachten Sie, dass ein Browserfenster geöffnet wird. Verwenden Sie Ihre Azure AD-Anmeldeinformationen, um sich anzumelden.

1. Sobald Sie bei Azure angemeldet sind, ist es an der Zeit, eine Ressourcengruppe zu erstellen, wenn sie noch nicht vorhanden ist, und einen SQL Server und eine Datenbank unter dieser Ressourcengruppe zu erstellen. Geben Sie den folgenden Befehl ein und wählen Sie **Eingabe**. *Die Ausführung des Skripts dauert einige Minuten.*

    ```bash
    cd ./Setup
    ./deploy-sql-database.ps1
    ```

    > &#128221; Beachten Sie, dass dieses Skript standardmäßig eine Ressourcengruppe mit dem Namen **contoso-rg** erstellt oder eine Ressource verwendet, deren Name mit *contoso-rg* beginnt, falls sie vorhanden ist. Standardmäßig werden auch alle Ressourcen in der Region **USA, Westen 2** (westus2) erstellt. Schließlich wird ein zufälliges 12-Zeichen-Kennwort für das **SQL-Administratorkennwort** generiert. Sie können diese Werte ändern, indem Sie einen oder mehrere der Parameter **-rgName**, **-location** und **-sqlAdminPw** mit Ihren eigenen Werten verwenden. Das Kennwort muss die Anforderungen an die Komplexität von Azure SQL-Kennwörtern erfüllen, mindestens 12 Zeichen lang und mindestens 1 Großbuchstaben, 1 Kleinbuchstaben, 1 Zahl und 1 Sonderzeichen enthalten.

    > &#128221; Beachten Sie, dass das Skript Ihre aktuelle öffentliche IP-Adresse zu den SQL Server-Firewallregeln hinzufügt.

1. Sobald das Skript abgeschlossen ist, werden der Name der Ressourcengruppe, der Name des SQL-Servers und der Datenbank sowie der Name des Administratorbenutzers und das Kennwort zurückgegeben. *Notieren Sie sich diese Werte, da Sie sie später in der Übung benötigen.*

---

## Automation-Konto erstellen

1. Starten Sie auf dem virtuellen Lab-Computer oder Ihrem lokalen Computer (falls kein virtueller Computer bereitgestellt wurde) eine Browsersitzung und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com/). Melden Sie sich mit Ihren Azure-Anmeldedaten beim Portal an.

1. Geben Sie im Azure-Portal in die Suchleiste *Automation* ein, und wählen Sie **Automation-Konten** aus den Suchergebnissen aus. Wählen Sie dann **+ Erstellen**.

1. Geben Sie auf der Seite **Automation-Konto erstellen** die folgenden Informationen ein, und wählen Sie dann **Überprüfen+ Erstellen** aus.

    - **Ressourcengruppe:** &lt;Ihre Ressourcengruppe&gt;
    - **Name des Automatisierungskontos:** autoAccount
    - **Region:** Verwenden Sie die Standardeinstellung.

1. Wählen Sie auf der Seite „Überprüfen“ die Option **Erstellen** aus.

    > &#128221; Das Erstellen Ihres Automatisierungskontos kann einige Minuten dauern.

## Herstellen einer Verbindung zu einer bestehenden Instanz von Azure SQL-Datenbank

1. Navigieren Sie im Azure-Portal zurück zu Ihrer Datenbank, indem Sie nach **SQL-Datenbanken** suchen.

1. Wählen Sie die SQL-Datenbank **AdventureWorksLT** aus.

1. Wählen Sie im Hauptabschnitt der Seite für Ihre SQL-Datenbank die Option **Abfrage-Editor (Vorschau)** aus.

1. Sie werden aufgefordert, sich mit dem Datenbank-Admin-Konto bei Ihrer Datenbank anzumelden. Wählen Sie **OK**.

    Dadurch wird eine neue Registerkarte in Ihrem Browser geöffnet. Wählen Sie **Client-IP hinzufügen** und dann **Speichern**. Nach dem Speichern kehren Sie zur vorherigen Registerkarte zurück und wählen Sie erneut **OK**.

    > &#128221; Möglicherweise wird die folgende Fehlermeldung angezeigt: *Der vom Login angeforderte Server „Ihr-SQL-Servername“ kann nicht geöffnet werden. Der Client mit der IP-Adresse „xxx.xxx.xxx.xxx“ hat keine Zugriffsberechtigung für den Server.* Falls ja, müssen Sie Ihre aktuelle öffentliche IP-Adresse den SQL Server-Firewallregeln hinzufügen.

    Wenn Sie die Firewallregeln einrichten müssen, führen Sie die folgenden Schritte aus:

    1. Wählen Sie **Server-Firewall festlegen** aus der oberen Menüleiste der **Übersichtsseite** der Datenbank aus.
    1. Wählen Sie **Aktuelle IPv4-Adresse hinzufügen (xxx.xxx.xxx.xxx)** und dann **Speichern**.
    1. Nach dem Speichern kehren Sie zur Datenbankseite **AdventureWorksLT** zurück und wählen Sie erneut **Abfrage-Editor (Vorschau)**.
    1. Sie werden aufgefordert, sich mit dem Datenbank-Admin-Konto bei Ihrer Datenbank anzumelden. Wählen Sie **OK**.

1. Wählen Sie im **Abfrage-Editor (Vorschau)** die Option **Abfrage öffnen**.

1. Wählen Sie das Symbol *Ordner durchsuchen* und navigieren Sie zum Ordner **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\Module13**. Wählen Sie die Datei **usp_AdaptiveIndexDefrag.sql** und wählen Sie **Öffnen** und anschließend **OK** aus.

1. Löschen Sie **USE msdb** und **GO** in den Zeilen 5 und 6 der Abfrage, und wählen Sie dann **Ausführen**.

1. Erweitern Sie den Ordner **Gespeicherte Prozeduren**, um die neu erstellten gespeicherten Prozeduren zu sehen.

## Konfigurieren von Automation-Kontoressourcen

In den nächsten Schritte konfigurieren Sie die Ressourcen, die zur Vorbereitung der Runbook-Erstellung erforderlich sind. Wählen Sie dann **Automation-Konten** aus.

1. Geben Sie im Azure-Portal im oberen Suchfeld **Automatisierung** ein und wählen Sie **Automatisierungskonten**.

1. Wählen Sie das **autoAccount**-Automatisierungskonto, das Sie erstellt haben.

1. Wählen Sie auf dem Blatt „Automation“ im Abschnitt **Freigegebene Ressourcen** die Option **Module** aus. Wählen Sie **Katalog durchsuchen** aus.

1. Suchen Sie im Katalog nach **SqlServer**.

1. Wählen Sie **SqlServer**, um zum nächsten Bildschirm zu gelangen, und klicken Sie dann auf die Schaltfläche **Auswählen**.

1. Wählen Sie auf der Seite **Modul hinzufügen** die neueste verfügbare Laufzeitversion und dann **Importieren** aus. Dadurch wird das PowerShell-Modul in Ihr Automation-Konto importiert.

1. Sie müssen Anmeldeinformationen erstellen, um sich sicher bei Ihrer Datenbank anzumelden. Navigieren Sie im Blatt für das *Automatisierungskonto* zum Abschnitt **Gemeinsame Ressourcen** und wählen Sie **Anmeldedaten** aus.

1. Wählen Sie **+ Anmeldeinformationen hinzufügen**, geben Sie die folgenden Informationen ein, und wählen Sie dann **Erstellen** aus.

    - Name: **SQLUser**
    - Benutzername: **sqladmin**
    - Kennwort: &lt;Geben Sie ein sicheres Kennwort ein, 12 Zeichen lang und enthält mindestens 1 Großbuchstaben, 1 Kleinbuchstaben, 1 Zahl und 1 Sonderzeichen.&gt;
    - Kennwort bestätigen: &lt;Geben Sie das zuvor eingegebene Kennwort erneut ein.&gt;

## Erstellen eines PowerShell-Runbooks

1. Navigieren Sie im Azure-Portal zurück zu Ihrer Datenbank, indem Sie nach **SQL-Datenbanken** suchen.

1. Wählen Sie die SQL-Datenbank **AdventureWorksLT** aus.

1. Auf der Seite **Übersicht** kopieren Sie den **Servernamen** Ihrer Azure SQL-Datenbank (Ihr Servername sollte mit *dp300-lab* beginnen). Sie werden ihn später wieder einfügen.

1. Geben Sie im Azure-Portal im oberen Suchfeld **Automatisierung** ein und wählen Sie **Automatisierungskonten**.

1. Wählen Sie das **AutoAccount**- Automatisierungskonto aus.

1. Erweitern Sie den Abschnitt **Prozessautomatisierung** des Automatisierungs-Kontoblatts und wählen Sie **Runbooks**.

1. Wählen Sie **+ Runbook erstellen** aus.

    > &#128221; Wie wir bereits erfahren haben, gibt es zwei bestehende Runbooks, die erstellt wurden. Diese wurden während der Bereitstellung des Automatisierungskontos automatisch erstellt.

1. Geben Sie den Runbooknamen **IndexMaintenance** und den Runbooktyp **PowerShell** ein. Wählen Sie die neueste verfügbare Laufzeitversion aus und wählen Sie dann **Überprüfen + Erstellen** aus.

1. Wählen Sie auf der Seite **Runbook erstellen** die Option **Erstellen**.

1. Nachdem das Runbook erstellt wurde, kopieren Sie den folgenden Powershell-Codeausschnitt und fügen ihn in den Runbook-Editor ein. 

    > &#128221; Bitte überprüfen Sie, ob der Code korrekt kopiert wurde, bevor Sie das Runbook speichern.

    ```powershell
    $AzureSQLServerName = ''
    $DatabaseName = 'AdventureWorksLT'
    
    $Cred = Get-AutomationPSCredential -Name "SQLUser"
    $SQLOutput = $(Invoke-Sqlcmd -ServerInstance $AzureSQLServerName -UserName $Cred.UserName -Password $Cred.GetNetworkCredential().Password -Database $DatabaseName -Query "EXEC dbo.usp_AdaptiveIndexDefrag" -Verbose) 4>&1

    Write-Output $SQLOutput
    ```

    > &#128221; Beachten Sie, dass der obige Code ein PowerShell-Skript ist, das die gespeicherte Prozedur **usp_AdaptiveIndexDefrag** in der **AdventureWorksLT**-Datenbank ausführt. Das Skript verwendet das Cmdlet **Invoke-Sqlcmd**, um eine Verbindung mit dem SQL-Server herzustellen und die gespeicherte Prozedur auszuführen. Das Cmdlet **Get-AutomationPSCredential** wird zum Abrufen der im Automatisierungskonto gespeicherten Anmeldedaten verwendet.

1. Geben Sie in der ersten Zeile des Skripts den Servernamen ein, den Sie in den vorherigen Schritten kopiert haben.

1. Wählen Sie **Speichern** und dann **Veröffentlichen** aus.

1. Wählen Sie **Ja**, um die Veröffentlichung zu bestätigen.

1. Das Runbook *IndexMaintenance* ist jetzt veröffentlicht.

## Erstellen eines Zeitplans für ein Runbook

Als Nächstes planen Sie die Ausführung des Runbooks in regelmäßigen Abständen.

1. Wählen Sie im linken Navigationsbereich Ihres Runbooks **IndexMaintenance** unter **Ressourcen** die Option **Zeitpläne**. 

1. Wählen Sie **+ Zeitplan hinzufügen** aus.

1. Wählen Sie **Zeitplan mit Runbook verknüpfen** aus.

1. Wählen Sie **+ Zeitplan hinzufügen** aus.

1. Geben Sie die nachstehenden Informationen ein und wählen Sie dann **Erstellen**.

    - **Name:** DailyIndexDefrag
    - **Beschreibung:** Tägliche Index-Defragmentierung für die AdventureWorksLT-Datenbank.
    - **Start:** 4:00 Uhr (nächster Tag)
    - **Zeitzone:**&lt;Wählen Sie die Zeitzone aus, die Ihrem Standort entspricht.&gt;
    - **Serie:** Wiederkehrend
    - **Wiederholung:** 1 Tag
    - **Ablaufdatum festlegen:** Nein

    > &#128221; Beachten Sie, dass die Startzeit am nächsten Tag auf 4:00 Uhr festgelegt ist. Die Zeitzone ist auf Ihre lokale Zeitzone festgelegt. Die Wiederholung ist auf jeden Tag festgelegt. Läuft nie ab.

1. Klicken Sie auf **Erstellen** und dann auf **OK**.

1. Der Zeitplan ist nun erstellt und mit dem Runbook verknüpft. Wählen Sie **OK** aus.

Azure Automation bietet einen cloudbasierten Automatisierungs- und Konfigurationsdienst, der eine konsistente Verwaltung Ihrer Azure- und sonstigen Umgebungen unterstützt.

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

### Löschen des LabFiles-Ordners

Wenn Sie einen neuen LabFiles-Ordner für dieses Lab erstellt haben und ihn nicht mehr benötigen, können Sie den LabFiles-Ordner löschen, um alle in diesem Lab erstellten Dateien zu entfernen.

1. Öffnen Sie auf dem virtuellen Lab-Computer oder auf Ihrem lokalen Computer, falls kein solcher zur Verfügung gestellt wurde, den Datei-Explorer und navigieren Sie zum Laufwerk **C:\\**.
1. Klicken Sie mit der rechten Maustaste auf den Ordner **LabFiles** und wählen Sie **Löschen**.
1. Wählen Sie **Ja**, um die Löschung des Ordners zu bestätigen.

---

Sie haben dieses Lab erfolgreich abgeschlossen.

In dieser Übung haben Sie die Defragmentierung von Indizes in einer SQL Server-Datenbank automatisiert. Sie wird täglich um 4:00 Uhr ausgeführt.
