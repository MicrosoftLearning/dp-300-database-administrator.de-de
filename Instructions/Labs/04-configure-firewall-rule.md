---
lab:
  title: 'Lab 4: Konfigurieren von Firewallregeln für die Azure SQL-Datenbank'
  module: Implement a Secure Environment for a Database Service
---

# Implementieren einer sicheren Umgebung

**Geschätzte Dauer: 30 Minuten**

Die Kursteilnehmenden nutzen die im Unterricht erworbenen Kenntnisse, um die Sicherheit im Azure-Portal und in der Datenbank *AdventureWorksLT* zu konfigurieren und anschließend zu implementieren.

Sie wurden als verantwortlicher Datenbankadministrator eingestellt, um die Sicherheit der Datenbankumgebung zu gewährleisten. Für diese Aufgaben wird schwerpunktmäßig Azure SQL-Datenbank eingesetzt.

> &#128221; Diese Übungen fordern Sie auf, T-SQL-Code zu kopieren und einzufügen, und nutzen vorhandene SQL-Ressourcen. Überprüfen Sie, ob der Code korrekt kopiert wurde, bevor Sie ihn ausführen.

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

## Konfigurieren von Firewallregeln für Azure SQL-Datenbank

1. Starten Sie auf dem virtuellen Lab-Computer oder Ihrem lokalen Computer (falls kein virtueller Computer bereitgestellt wurde) eine Browsersitzung und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com/). Melden Sie sich mit Ihren Azure-Anmeldedaten beim Portal an.

1. Suchen Sie im Azure-Portal im oberen Suchfeld nach *SQL-Server* und wählen Sie dann **SQL-Server** aus der Liste der Optionen aus.

1. Wählen Sie den SQL-Server **dp300-lab-xxxxxxxx** aus, wobei *xxxxxxxx* eine zufällige numerische Zeichenfolge ist.

    > &#128221; Wenn Sie Ihren eigenen Azure SQL-Server verwenden, der nicht in diesem Lab erstellt wurde, wählen Sie den Namen dieses SQL-Servers aus.

1. Wählen Sie im Bildschirm *Übersicht* für Ihren SQL-Server rechts neben dem Servernamen die Schaltfläche **In Zwischenablage kopieren**.

1. Wählen Sie **Netzwerkeinstellungen anzeigen** aus.

1. Überprüfen Sie auf der Seite **Netzwerk** unter **Firewall-Regeln** die Liste und stellen Sie sicher, dass Ihre Client-IP-Adresse aufgeführt ist. Wenn sie nicht aufgeführt ist, wählen Sie **+ Client-IPv4-Adresse hinzufügen (Ihre IP-Adresse)** und dann **Speichern**.

    > &#128221; Beachten Sie, dass Ihre Client-IP-Adresse automatisch für Sie eingegeben wurde. Wenn Sie die IP-Adresse Ihres Clients zur Liste hinzufügen, können Sie über SQL Server Management Studio (SSMS) oder andere Client-Tools eine Verbindung zu Ihrer Azure SQL-Datenbank herstellen. **Notieren Sie sich die Client-IP-Adresse, denn Sie wird im Verlauf dieser Übung benötigt.**

1. Öffnen Sie SQL Server Management Studio. Fügen Sie im Dialogfeld „Mit Server verbinden“ den Namen Ihres Azure SQL-Datenbankservers ein und melden Sie sich mit den folgenden Anmeldeinformationen an:

    - **Servername:** &lt;_Namen des Azure SQL-Datenbankservers hier einfügen_&gt;
    - **Authentifizierung:** SQL Server-Authentifizierung
    - **Server-Admin-Anmeldung:** Ihre Azure SQL-Datenbank-Server-Admin-Anmeldung
    - **Kennwort:** Ihr Azure SQL-Datenbank-Serveradminkennwort

1. Wählen Sie **Verbinden**.

1. Erweitern Sie im Objekt-Explorer den Serverknoten, und klicken Sie mit der rechten Maustaste auf **Datenbanken**. Wählen Sie **Datenschichtanwendung importieren** aus.

1. Klicken Sie im Dialogfeld **Datenschichtanwendung importieren** im ersten Bildschirm auf **Weiter**.

1. Klicken Sie im Bildschirm **Import-Einstellungen** auf **Durchsuchen** und navigieren Sie zum Ordner **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\04**, klicken Sie auf die Datei **AdventureWorksLT.bacpac** und dann auf **Öffnen**. Zurück zum Bildschirm **Datenschichtanwendung importieren** wählen Sie **Weiter**.

1. Nehmen Sie auf der Anzeige **Datenbankeinstellungen** die folgenden Änderungen vor:

    - **Datenbankname:** AdventureWorksFromBacpac
    - **Edition von Microsoft Azure SQL-Datenbank**: Basic

1. Wählen Sie **Weiter** aus.

1. Klicken Sie auf dem Bildschirm **Zusammenfassung** auf **Fertig stellen**. Dieser Vorgang kann einige Minuten in Anspruch nehmen. Nachdem der Import abgeschlossen ist, werden die unten aufgeführten Ergebnisse angezeigt. Klicken Sie dann auf **Schließen**.

1. Erweitern Sie im **Objekt-Explorer** von SQL Server Management Studio den Ordner **Datenbanken**. Klicken Sie anschließend mit der rechten Maustaste auf die Datenbank **AdventureWorksFromBacpac**, und wählen Sie **Neue Abfrage** aus.

1. Führen Sie die folgende T-SQL-Abfrage aus, indem Sie den Text in das Abfragefenster einfügen.
    1. **Wichtig:** Ersetzen Sie **000.000.000.000** durch Ihre Client-IP-Adresse. Wählen Sie **Execute**(Ausführen).

    ```sql
    EXECUTE sp_set_database_firewall_rule 
            @name = N'AWFirewallRule',
            @start_ip_address = '000.000.000.000', 
            @end_ip_address = '000.000.000.000'
    ```

1. Als Nächstes erstellen Sie in der Datenbank **AdventureWorksFromBacpac** einen eingeschränkten Benutzer. Klicken Sie auf **Neue Abfrage**, und führen Sie den folgenden T-SQL-Code aus.

    ```sql
    USE [AdventureWorksFromBacpac]
    GO
    CREATE USER ContainedDemo WITH PASSWORD = 'P@ssw0rd01'
    ```

    > &#128221; Dieser Befehl erstellt ein in der **AdventureWorksFromBacpac**-Datenbank enthaltenes Benutzerkonto. Im nächsten Schritt testen wir diese Anmeldeinformationen.

1. Navigieren Sie zum **Objekt-Explorer**. Klicken Sie auf **Verbinden**, und **Datenbank-Engine**.

1. Versuchen Sie, mit den im vorherigen Schritt erstellten Anmeldeinformationen eine Verbindung herzustellen. Verwenden Sie die folgenden Informationen:

    - **Anmeldename:** containeddemo
    - **Kennwort**: P@ssw0rd01

     Klicken Sie auf **Verbinden**.

     Sie erhalten die folgende Fehlermeldung:

    <span style="color:red">Anmeldung für den Benutzer „ContainedDemo“ fehlgeschlagen (Microsoft SQL Server, Fehler: 18456)</span>

    > &#128221; Dieser Fehler wird generiert, weil die Verbindung versucht hat, sich bei der Datenbank *master* anzumelden und nicht bei **AdventureWorksFromBacpac**, wo das Benutzerkonto erstellt wurde. Ändern Sie den Verbindungskontext, indem Sie **OK** auswählen, um die Fehlermeldung zu schließen, und dann unter **Mit Server verbinden** auf **Optionen >>** klicken.

1. Geben Sie auf der Registerkarte **Verbindungseigenschaften** den Datenbanknamen **AdventureWorksFromBacpac** ein, und wählen Sie dann **Verbinden** aus.

1. Beachten Sie, dass Sie sich erfolgreich mit dem Benutzer **ContainedDemo** authentifizieren konnten. Dieses Mal waren Sie direkt bei **AdventureWorksFromBacpac** angemeldet, der einzigen Datenbank, auf die der neu angelegte Benutzer Zugriff hat.

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

In dieser Übung haben Sie Firewallregeln für Server und Datenbank für den Zugriff auf eine auf Azure SQL Database gehostete Datenbank konfiguriert. Außerdem haben Sie mit T-SQL-Anweisungen einen eingeschränkten Benutzer erstellt und in SQL Server Management Studio den Zugriff überprüft.
