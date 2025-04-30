---
lab:
  title: 'Lab 6: Isolieren von Leistungsproblemen durch Überwachung'
  module: Monitor and optimize operational resources in Azure SQL
---

# Isolieren von Leistungsproblemen durch Überwachung

**Geschätzte Dauer: 30 Minuten**

Die Kursteilnehmenden planen anhand der in den Lektionen gewonnenen Informationen die Leistungen für ein digitales Transformationsprojekt in AdventureWorksLT. Sie untersuchen das Azure-Portal und andere Tools und bestimmen, wie sie Tools zur Identifizierung und Lösung von Leistungsproblemen einsetzen können.

Sie wurden als Datenbankadministrator eingestellt, um leistungsbezogene Probleme zu ermitteln und geeignete Lösungen zu erarbeiten, um gefundene Probleme zu beheben. Sie müssen das Azure-Portal verwenden, um die Leistungsprobleme zu identifizieren, und Methoden vorschlagen, um diese zu beheben.

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

## Überprüfen der CPU-Auslastung im Azure-Portal

1. Starten Sie auf dem virtuellen Lab-Computer oder Ihrem lokalen Computer (falls kein virtueller Computer bereitgestellt wurde) eine Browsersitzung und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com/). Melden Sie sich mit Ihren Azure-Anmeldedaten beim Portal an.

1. Suchen Sie im Azure-Portal im oberen Suchfeld nach *SQL-Server* und wählen Sie dann **SQL-Server** aus der Liste der Optionen aus.

1. Wählen Sie den SQL-Server **dp300-lab-xxxxxxxx** aus, wobei *xxxxxxxx* eine zufällige numerische Zeichenfolge ist.

    > &#128221; Wenn Sie Ihren eigenen Azure SQL-Server verwenden, der nicht in diesem Lab erstellt wurde, wählen Sie den Namen dieses SQL-Servers aus.

1. Wählen Sie auf der Hauptseite von Azure SQL Server unter **Sicherheit** die Option **Netzwerke**.

1. Überprüfen Sie auf der Seite **Netzwerke**, ob Ihre aktuelle öffentliche IP-Adresse bereits zur Liste **Firewall-Regeln** hinzugefügt wurde. Wenn nicht, wählen Sie **+ Client-IPv4-Adresse hinzuzufügen (Ihre IP-Adresse)**, um sie hinzuzufügen, und wählen Sie dann **Speichern**.

1. Navigieren Sie im Hauptblatt Ihres Azure-SQL-Servers zum Abschnitt **Einstellungen** und wählen Sie **SQL-Datenbanken** und dann die Datenbank **AdventureWorksLT** aus.

1. Klicken Sie in der linken Navigationsleiste auf **Abfrage-Editor (Vorschau)**.

    **Hinweis**: Dieses Feature befindet sich in der Vorschau.

1. Wählen Sie den Benutzernamen des SQL Server-Admins aus, und geben Sie das Kennwort oder Ihre Microsoft Entra-Anmeldeinformationen ein, wenn die Verbindung mit der Datenbank hergestellt werden soll.

1. Geben Sie unter **Abfrage 1** die folgende Abfrage ein, und klicken Sie auf **Ausführen**:

    ```sql
    DECLARE @Counter INT 
    SET @Counter=1
    WHILE ( @Counter <= 10000)
    BEGIN
        SELECT 
             RTRIM(a.Firstname) + ' ' + RTRIM(a.LastName)
            , b.AddressLine1
            , b.AddressLine2
            , RTRIM(b.City) + ', ' + RTRIM(b.StateProvince) + '  ' + RTRIM(b.PostalCode)
            , CountryRegion
            FROM SalesLT.Customer a
            INNER JOIN SalesLT.CustomerAddress c 
                ON a.CustomerID = c.CustomerID
            RIGHT OUTER JOIN SalesLT.Address b
                ON b.AddressID = c.AddressID
        ORDER BY a.LastName ASC
        SET @Counter  = @Counter  + 1
    END
    ```

1. Warten Sie, bis die Abfrage beendet ist.

1. Führen Sie die Abfrage *zweimal* erneut aus, um eine CPU-Auslastung für die Datenbank zu generieren.

1. Wählen Sie im Blade für die Datenbank **AdventureWorksLT** das Symbol **Metriken** im Abschnitt **Überwachung**.

    Wenn die Meldung *Ihre nicht gespeicherten Änderungen werden verworfen* erscheint, wählen Sie **OK**.

1. Ändern Sie die Menüoption **Metrik** so, dass sie den **CPU-Prozentsatz** widerspiegelt, und wählen Sie dann eine **Aggregation** von **Mittelwert** aus. Dadurch wird der durchschnittliche CPU-Prozentsatz für den angegebenen Zeitrahmen angezeigt.

1. Beobachten Sie den CPU-Durchschnitt über einen Zeitraum hinweg. Beachten Sie eine Spitzenauslastung bei der CPU-Auslastung am Ende des Diagramms, wenn die Abfrage ausgeführt wurde.

## Ermitteln von Abfragen mit hoher CPU-Auslastung

1. Suchen Sie im Abschnitt **Intelligente Leistung** des Blatts für die **AdventureWorksLT**-Datenbank nach dem Symbol für **Query Performance Insight**.

1. Wählen Sie **Einstellungen zurücksetzen** aus.

1. Wählen Sie die Abfrage in der Tabelle unter dem Diagramm aus. Wenn Sie die Abfrage, die wir zuvor mehrmals ausgeführt haben, nicht sehen, warten Sie 2 bis 5 Minuten und wählen Sie **Aktualisieren**.

    > &#128221; Wenn mehr als eine Abfrage aufgelistet ist, wählen Sie die einzelnen Abfragen aus, um die Ergebnisse zu verfolgen. Beachten Sie die umfangreiche Menge an Informationen, die für jede Abfrage verfügbar sind.

1. Bei der Abfrage, die Sie vorhin ausgeführt haben, ist zu beachten, dass die Gesamtdauer über eine Minute betrug und dass die Abfrage etwa 30.000 ausgeführt wurde.

1. Wenn Sie den SQL-Text auf der Seite **Abfragedetails** mit der von Ihnen ausgeführten Abfrage vergleichen, werden Sie feststellen, dass die **Abfragedetails** nur die Anweisung **SELECT** und nicht die Schleife **WHILE** oder eine andere Anweisung enthalten. Dies geschieht, weil **Query Performance Insight** sich auf Daten aus dem **Abfragespeicher** stützt, der nur DML-Anweisungen (Datenbearbeitungssprache) wie **SELECT, INSERT, UPDATE, DELETE, MERGE,** und **BULK INSERT** verfolgt, aber DDL-Anweisungen (Datendefinitionssprache) ignoriert.

Nicht alle Leistungsprobleme hängen mit einer hohen CPU-Auslastung durch eine einzelne Abfrageausführung zusammen. In diesem Fall wurde die Abfrage tausende Mal ausgeführt, was auch zu einer hohen CPU-Auslastung führen kann.

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

In dieser Übung haben Sie gelernt, wie Sie Serverressourcen für eine Azure SQL-Datenbank untersuchen und potenzielle Probleme mit der Abfrageleistung mithilfe von Query Performance Insight identifizieren können.
