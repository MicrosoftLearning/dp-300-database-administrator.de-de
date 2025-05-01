---
lab:
  title: 'Lab 14: Konfigurieren der Georeplikation für Azure SQL-Datenbank'
  module: Plan and implement a high availability and disaster recovery solution
---

# Konfigurieren der Georeplikation für Azure SQL-Datenbank

**Geschätzte Dauer: 30 Minuten**

Als DBA in AdventureWorks müssen Sie die Georeplikation für Azure SQL Database aktivieren und sicherstellen, dass sie ordnungsgemäß funktioniert. Darüber hinaus verlagern Sie die Datenbank in einem Failover manuell über das Portal in eine andere Region.

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

## Aktivieren der Georeplikation

1.  Starten Sie auf dem virtuellen Lab-Computer oder Ihrem lokalen Computer (falls kein virtueller Computer bereitgestellt wurde) eine Browsersitzung und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com/). Melden Sie sich mit Ihren Azure-Anmeldedaten beim Portal an.

1. Navigieren Sie im Azure-Portal zurück zu Ihrer Datenbank, indem Sie nach **SQL-Datenbanken** suchen.

1. Wählen Sie die SQL-Datenbank **AdventureWorksLT** aus.

1. Wählen Sie auf dem Blatt für die Datenbank im Bereich **Datenverwaltung** die Option **Replikate** aus.

1. Wählen Sie **+ Replikat erstellen** aus.

1. Auf der Seite **SQL-Datenbank erstellen – Georeplikat** sehen Sie, dass die Abschnitte **Projektdetails** und **Primäre Datenbank** bereits mit dem Abonnement, der Ressourcengruppe und dem Datenbanknamen ausgefüllt sind.

1. Wählen Sie im Abschnitt **Replikatkonfiguration** die Option **Georeplikat** als *Replikat-Typ*.

1. Als **Details der sekundären Geodatenbank** geben Sie die folgenden Werte an:

    - **Abonnement**: &lt;Ihr Abonnementname&gt; (identisch mit der primären Datenbank).
    - **Ressourcengruppe**: &lt;Wählen Sie dieselbe Ressourcengruppe wie für die primäre Datenbank aus.&gt; 
    - **Datenbankname**: Der Datenbankname ist ausgegraut und entspricht dem primären Datenbanknamen.
    - **Server**: Klicken Sie auf **Neu erstellen**.
    - Geben Sie auf der Seite **SQL-Datenbankserver erstellen** die folgenden Werte ein:

        - **Servername**: Geben Sie einen eindeutigen Namen für den sekundären Server ein. Der Name muss für alle Azure SQL-Datenbank-Server eindeutig sein.
        - **Ort**: Wählen Sie eine andere Region aus der primären Datenbank aus. Beachten Sie, dass in Ihrem Abonnement möglicherweise nicht alle Regionen verfügbar sind.
        - Aktivieren Sie das Kontrollkästchen **Zugriff der Azure-Dienste auf den Server zulassen**. Beachten Sie, dass Sie in einer Produktionsumgebung den Zugriff auf den Server einschränken möchten.
        - Als Authentifizierung wählen Sie **SQL-Authentifizierung**. Beachten Sie, dass Sie in einer Produktionsumgebung möglicherweise die Authentifizierung **Nur Microsoft Entra verwenden** verwenden möchten. Geben Sie **sqladmin* als Admin-Anmeldenamen und ein sicheres Kennwort ein. Das Kennwort muss die Komplexitätsanforderungen für Azure SQL-Kennwörter erfüllen, mindestens 12 Zeichen lang sein und mindestens 1 Großbuchstaben, 1 Kleinbuchstaben, 1 Ziffer und 1 Sonderzeichen enthalten.
        - Wählen Sie **OK**, um den Server zu erstellen.

    - **Möchten Sie einen Pool für elastische Datenbanken verwenden?**: Nein.
    - **Compute + Speicher**: Universell, Gen 5, 2 vCores, 32 GB Arbeitsspeicher.
    - **Redundanz für Sicherungsspeicher**: Lokal redundanter Sicherungsspeicher (LRS). Beachten Sie, dass Sie in einer Produktionsumgebung möglicherweise **Georedundanten Speicher (GRS)** verwenden möchten.

1. Klicken Sie auf **Überprüfen + erstellen**.

1. Klicken Sie auf **Erstellen**. Es dauert ein paar Minuten, um den sekundären Server und die sekundäre Datenbank zu erstellen. Sobald der Vorgang abgeschlossen ist, ändert sich der Fortschrittsstatus von **Bereitstellung wird ausgeführt** zu **Ihre Bereitstellung wurde abgeschlossen**.

1. Wählen Sie **Zur Ressource wechseln**, um zur Datenbank des sekundären Servers für den nächsten Schritt zu navigieren.

## Failover der SQL-Datenbank in eine sekundäre Region

Nachdem das Azure SQL-Datenbank-Replikat erstellt wurde, führen Sie ein Failover aus.

1. Falls noch nicht auf der Datenbank des sekundären Servers vorhanden, suchen Sie im Azure-Portal nach **sql databases** und wählen Sie die SQL-Datenbank **AdventureWorksLT** auf dem sekundären Server aus.

1. Wählen Sie auf dem Hauptblatt für die SQL-Datenbank unter **Datenverwaltung** die Option **Replikate** aus.

1. Beachten Sie, dass die Verbindung zur Georeplikation nun hergestellt ist. Der Wert *Replikatstatus* der primären Datenbank lautet **Online** und der Wert *Replikatstatus* der Georeplikate lautet **Lesbar**.

1. Wählen Sie das Menü **...** für den sekundären Georeplikationsserver und wählen Sie **Erzwungenes Failover**.

    > &#128221; Beim erzwungenen Failover wird die sekundäre Datenbank in die primäre Rolle versetzt. Alle Sitzungen werden während dieses Vorgangs getrennt.

1. Wenn Sie in einer Warnmeldung dazu aufgefordert werden, klicken Sie auf **Ja**.

1. Der Status des primären Replikats ändert sich in **Ausstehend** und der des sekundären zu **Failover**. 

     > &#128221; Beachten Sie, dass das Failover schnell ist, da die Datenbank klein ist. In einer Produktionsumgebung kann dieser Vorgang einige Minuten dauern.

1. Wenn er abgeschlossen ist, werden die Rollen getauscht, wobei das sekundäre Replikat zum neuen primären wird und das alte primäre zum sekundären. Möglicherweise müssen Sie die Seite aktualisieren, um den neuen Status zu sehen.

Die lesbare sekundäre Datenbank kann sich in derselben Azure-Region befinden wie die primäre, oder, was häufiger der Fall ist, in einer anderen Region. Diese Art von lesbaren Sekundärdatenbanken werden auch als Geo-Sekundärdatenbanken oder Geo-Replikate bezeichnet.

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

Sie haben nun gelernt, wie Sie Geo-Replikate für Azure SQL Database aktivieren und sie per manuellem Failover über das Portal in eine andere Region verlagern können.
