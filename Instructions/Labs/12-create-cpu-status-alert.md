---
lab:
  title: 'Lab 12: Erstellen einer CPU-Statuswarnung für eine SQL Server-Instanz'
  module: Automate database tasks for Azure SQL
---

# Erstellen einer CPU-Statuswarnung für eine SQL Server-Instanz auf Azure

**Geschätzte Dauer**: 20 Minuten

Sie wurden als Senior Data Engineer eingestellt, um die Automatisierung der alltäglichen Datenbankverwaltungsvorgänge voranzutreiben. Diese Automatisierung soll sicherstellen, dass die Datenbanken für AdventureWorks weiterhin mit maximaler Leistung betrieben werden und Methoden für die Alarmierung auf der Grundlage bestimmter Kriterien bereitstellen.

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

## Erstellen einer Warnung, wenn eine CPU einen Durchschnitt von 80 Prozent überschreitet

1. Starten Sie auf dem virtuellen Lab-Computer oder Ihrem lokalen Computer (falls kein virtueller Computer bereitgestellt wurde) eine Browsersitzung und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com/). Melden Sie sich mit Ihren Azure-Anmeldedaten beim Portal an.

1. Geben Sie im Azure-Portal in der Suchleiste oben im Azure-Portal **SQL-Datenbanken** ein und wählen Sie **SQL-Datenbanken** aus. Wählen Sie den aufgelisteten Datenbanknamen **AdventureWorksLT** aus.

1. Navigieren Sie im Hauptblatt für die Datenbank **AdventureWorksLT** nach unten zum Abschnitt „Überwachung“. Wählen Sie **Warnungen** aus.

1. Wählen Sie **Warnungsregel erstellen** aus.

1. Wählen Sie auf der Seite **Warnregel erstellen** die Option **CPU-Prozentsatz**.

1. Im Abschnitt **Warnungslogik** wählen Sie **Statisch** für den **Schwellenwerttyp**. Prüfen Sie dann, dass der Typ **Aggregation** auf **Durchschnitt** und dass die Eigenschaft **Wert ist**  auf **Größer als** festgelegt ist. Geben Sie dann unter **Schwellenwert** einen Wert von **80** ein. Überprüfen Sie die Werte *Alles prüfen* und *Rückblickzeitraum*.

1. Wählen Sie **Weiter: Aktionen >** aus.

1. Wählen Sie auf der Registerkarte **Aktionen** die Option **Aktionsgruppe erstellen** aus.

1. Geben Sie auf dem Bildschirm **Aktionsgruppe** **emailgroup** in die Felder **Aktionsgruppenname** und **Anzeigename** ein und wählen Sie dann **Weiter: Benachrichtigungen**.

1. Geben Sie auf der Registerkarte **Benachrichtigungen** die folgenden Informationen ein:

    - **Benachrichtigungstyp:** E-Mail/SMS-Nachricht/Pushnachricht/Sprachnachricht

        > &#128221; Wenn Sie diese Option wählen, erscheint ein E-Mail/SMS-Nachrichten/Push/Voice-Flyout. Überprüfen Sie die E-Mail-Eigenschaft, und geben Sie den Azure-Benutzernamen ein, mit dem Sie sich angemeldet haben. Wählen Sie **OK** aus.

    - **Name:** DemoLab

1. Klicken Sie auf**Überprüfen + erstellen** und dann auf **Erstellen**.

1. Zurück auf der Seite **Warnungsregel erstellen**, wählen Sie **Weiter: Details** und geben Sie der Warnungsregel einen eindeutigen Namen.

1. Klicken Sie auf**Überprüfen + erstellen** und dann auf **Erstellen**.

1. Wenn die CPU-Auslastung im Durchschnitt 80% überschreitet, wird eine E-Mail verschickt.

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

---

Sie haben dieses Lab erfolgreich abgeschlossen.

Sie können Warnungen so konfigurieren, dass Sie bei Erreichen eines Schwellenwerts für eine bestimmte Metrik (etwa Datenbankgröße oder CPU-Auslastung) per E-Mail oder Webhook benachrichtigt werden. Sie haben gerade gelernt, wie einfach es ist, Warnungen für Azure SQL-Datenbanken zu konfigurieren.
