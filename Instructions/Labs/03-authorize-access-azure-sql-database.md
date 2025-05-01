---
lab:
  title: "Lab 3 – Autorisieren des Zugriffs auf Azure SQL-Datenbank mittels Microsoft Entra\_ID"
  module: Implement a Secure Environment for a Database Service
---

# Konfigurieren der Datenbankauthentifizierung und -autorisierung

**Geschätzte Dauer**: 25 Minuten

Die Kursteilnehmenden nutzen die im Unterricht erworbenen Kenntnisse, um die Sicherheit im Azure-Portal und in der Datenbank *AdventureWorksLT* zu konfigurieren und anschließend zu implementieren.

Sie wurden als verantwortlicher Datenbankadministrator eingestellt, um die Sicherheit der Datenbankumgebung zu gewährleisten.

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

1. Sobald das Skript abgeschlossen ist, werden der Name der Ressourcengruppe, der Name des SQL-Servers und der Datenbank sowie der Name des Administratorbenutzers und das Kennwort zurückgegeben. Notieren Sie sich diese Werte, da Sie sie später im Lab benötigen werden.

---

## Autorisieren des Zugriffs auf Azure SQL-Datenbank mittels Microsoft Entra

Sie können Anmeldungen aus Microsoft Entra-Konten mithilfe der T-SQL-Syntax `CREATE USER [anna@contoso.com] FROM EXTERNAL PROVIDER` als Benutzer*innen eigenständiger Datenbanken erstellen. Ein*e Benutzer*in einer eigenständigen Datenbank wird einer Identität im Microsoft Entra-Verzeichnis zugeordnet, das der Datenbank zugeordnet ist, und hat keine Anmeldung in der `master`-Datenbank.

Mit der Einführung von Microsoft Entra-Serveranmeldungen in Azure SQL-Datenbank können Sie Anmeldungen aus Microsoft Entra-Prinzipalen in der virtuellen `master`-Datenbank einer SQL-Datenbank-Instanz erstellen. Sie können Microsoft Entra-Anmeldungen aus Microsoft Entra *-Benutzer*innen, -Gruppen und -Dienstprinzipalen* erstellen. Weitere Informationen finden Sie unter [Microsoft Entra-Serverprinzipale](/azure/azure-sql/database/authentication-azure-ad-logins).

Außerdem können Sie das Azure-Portal nur zum Erstellen von Administrator*innen verwenden, und Rollen der rollenbasierten Zugriffssteuerung von Azure werden nicht an logische Server von Azure SQL-Datenbank weitergegeben. Sie müssen zusätzliche Server- und Datenbankberechtigungen gewähren, indem Sie Transact-SQL (T-SQL) verwenden. Erstellen wir nun einen Microsoft Entra-Admin für den SQL Server.

1. Starten Sie auf dem virtuellen Lab-Computer oder Ihrem lokalen Computer (falls kein virtueller Computer bereitgestellt wurde) eine Browsersitzung und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com/). Melden Sie sich mit Ihren Azure-Anmeldedaten beim Portal an.

1. Suchen Sie auf der Startseite des Azure-Portals nach **SQL-Servern** und wählen Sie diese aus.

1. Wählen Sie den SQL-Server **dp300-lab-xxxxxxxx** aus, wobei *xxxxxxxx* eine zufällige numerische Zeichenfolge ist.

    > &#128221; Wenn Sie Ihren eigenen Azure SQL-Server verwenden, der nicht in diesem Lab erstellt wurde, wählen Sie den Namen dieses SQL-Servers aus.

1. Wählen Sie im Blatt *Übersicht* die Option **Nicht konfiguriert** neben *Microsoft Entra Admin*.

1. Wählen Sie im nächsten Bildschirm **Administrator festlegen** aus.

1. Suchen Sie in der Seitenleiste **Microsoft Entra ID** nach dem Azure-Benutzernamen, mit dem Sie sich beim Azure-Portal angemeldet haben, und klicken Sie dann auf **Auswählen**.

1. Wählen Sie **Speichern** aus, um den Vorgang abzuschließen. Dadurch wird Ihr Benutzername zum Microsoft Entra-Admin für den Server.

1. Wählen Sie auf der linken Seite **Übersicht** aus, und kopieren Sie dann den **Servernamen**.

1. Öffnen Sie SQL Server Management Studio (SSMS) und wählen Sie **Verbinden** > **Datenbankmodul**. Fügen Sie in das Feld **Servername** den Namen Ihres Servers ein. Ändern Sie den Authentifizierungstyp in **Microsoft Entra MFA**.

1. Wählen Sie **Verbinden**.

## Verwalten des Zugriffs auf Datenbankobjekte

In dieser Aufgabe verwalten Sie den Zugriff auf die Datenbank und deren Objekte. Als Erstes erstellen Sie zwei Benutzer in der Datenbank *AdventureWorksLT*.

1. Melden Sie sich in SSMS mit dem Azure Server-Administratorkonto oder dem Microsoft Entra-Administratorkonto bei der Datenbank *AdventureWorksLT* über den virtuellen Lab-Computer oder Ihren lokalen Computer an, falls keiner zur Verfügung gestellt wurde.

1. Verwenden Sie den **Objekt-Explorer**, und erweitern Sie **Datenbanken**.

1. Klicken Sie mit der rechten Maustaste auf **AdventureWorksLT**, und wählen Sie **Neue Abfrage** aus.

1. Kopieren Sie im Fenster „Neue Abfrage“ den folgenden T-SQL, und fügen Sie ihn ein. Führen Sie die Abfrage aus, um die beiden Benutzer zu erstellen.

    ```sql
    CREATE USER [DP300User1] WITH PASSWORD = 'Azur3Pa$$';
    GO

    CREATE USER [DP300User2] WITH PASSWORD = 'Azur3Pa$$';
    GO
    ```

    **Hinweis:** Beachten Sie, dass diese Benutzer im Geltungsbereich der AdventureWorksLT-Datenbank erstellt werden. Als Nächstes erstellen Sie eine benutzerdefinierte Rolle, und fügen ihr die Benutzer hinzu.

1. Führen Sie den folgenden T-SQL-Code im selben Abfragefenster aus.

    ```sql
    CREATE ROLE [SalesReader];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User1];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User2];
    GO
    ```

    Erstellen Sie als Nächstes eine neue gespeicherte Prozedur im Schema **SalesLT**.

1. Führen Sie den folgenden T-SQL-Code in Ihrem Abfragefenster aus.

    ```sql
    CREATE OR ALTER PROCEDURE SalesLT.DemoProc
    AS
    SELECT P.Name, Sum(SOD.LineTotal) as TotalSales ,SOH.OrderDate
    FROM SalesLT.Product P
    INNER JOIN SalesLT.SalesOrderDetail SOD on SOD.ProductID = P.ProductID
    INNER JOIN SalesLT.SalesOrderHeader SOH on SOH.SalesOrderID = SOD.SalesOrderID
    GROUP BY P.Name, SOH.OrderDate
    ORDER BY TotalSales DESC
    GO
    ```

    Verwenden Sie als nächstes die Syntax `EXECUTE AS USER`, um die Sicherheit zu testen. Dies ermöglicht es der Datenbank-Engine, eine Abfrage im Kontext Ihres Benutzers auszuführen.

1. Führen Sie den folgenden T-SQL-Code aus.

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

    Dies schlägt mit folgender Meldung fehl:

    <span style="color:red">Msg 229, Level 14, State 5, Procedure SalesLT.DemoProc, Line 1 [Batch Start Line 0]  The EXECUTE permission was denied on the object 'DemoProc', database 'AdventureWorksLT', schema 'SalesLT'.</span>

1. Gewähren Sie als Nächstes der Rolle Berechtigungen zum Ausführen der gespeicherten Prozedur. Führen Sie den folgenden T-SQL-Code aus.

    ```sql
    REVERT;
    GRANT EXECUTE ON SCHEMA::SalesLT TO [SalesReader];
    GO
    ```

    Der erste Befehl kehrt den Ausführungskontext wieder zurück in den Datenbankbesitzer um.

1. Führen Sie den vorherigen T-SQL-Code erneut aus.

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

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

In dieser Übung haben Sie erfahren, wie Sie mit Microsoft Entra ID Azure-Anmeldedaten Zugriff auf einen in Azure gehosteten SQL Server gewähren können. Außerdem haben Sie mit einer T-SQL-Anweisung neue Datenbankbenutzer angelegt und ihnen Berechtigungen zum Ausführen gespeicherter Prozeduren erteilt.
