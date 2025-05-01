---
lab:
  title: 'Lab 2: Bereitstellen einer Azure SQL-Datenbank-Instanz'
  module: Plan and Implement Data Platform Resources
---

# Bereitstellen einer Azure SQL-Datenbank

**Geschätzte Dauer: 40 Minuten**

Die Kursteilnehmer konfigurieren grundlegende Ressourcen, die für die Bereitstellung einer Azure SQL-Datenbank mit einem virtuellen Netzwerkendpunkt erforderlich sind. Die Verbindung zur SQL-Datenbank wird mithilfe von SQL Server Management Studio von der Lab-VM (sofern verfügbar) oder von Ihrem lokalen Computer aus überprüft.

Als Datenbankadministrator richten Sie eine neue SQL-Datenbank ein, einschließlich eines Endpunkts für das virtuelle Netzwerk, um die Sicherheit der Bereitstellung zu erhöhen und zu vereinfachen. SQL Server Management Studio wird verwendet, um die Verwendung eines SQL-Notebooks für die Datenabfrage und die Speicherung der Ergebnisse zu bewerten.

## Navigieren Sie zum Azure-Portal.

1. Öffnen Sie auf dem virtuellen Lab-Computer, falls verfügbar, auf Ihrem lokalen Computer ein Browserfenster.

1. Navigieren Sie zum Azure-Portal auf [https://portal.azure.com](https://portal.azure.com/). Melden Sie sich mit Ihrem Azure-Konto oder den bereitgestellten Anmeldedaten (falls vorhanden) beim Azure-Portal an.

1. Suchen Sie im Azure-Portal im Suchfeld oben nach *Ressourcengruppen* und wählen Sie dann **Ressourcengruppen** aus der Liste der Optionen aus.

1. Auf der Seite **Ressourcengruppe**, falls vorhanden, wählen Sie die Ressourcengruppe, die mit *contoso-rg* beginnt. Wenn diese Ressourcengruppe nicht existiert, erstellen Sie entweder eine neue Ressourcengruppe mit dem Namen *contoso-rg* in Ihrer lokalen Region oder verwenden Sie eine vorhandene Ressourcengruppe und notieren Sie sich die Region, in der sie sich befindet.

## Erstellen eines virtuellen Netzwerks

1. Wählen Sie auf der Startseite von Azure-Portal das Menü links aus.  

1. Wählen Sie im linken Navigationsbereich **virtuelle Netzwerke** aus.  

1. Wählen Sie **+ Erstellen** aus, um die Seite **Virtuelles Netzwerk erstellen** zu öffnen. BasicGeben Sie auf der Registerkarte **Grundlagen** die folgenden Informationen an:

    - **Abonnement:** &lt;Ihr Abonnement&gt;
    - **Ressourcengruppe:** Beginnen Sie mit *DP300* oder der Ressourcengruppe, die Sie zuvor ausgewählt haben.
    - **Name:** lab02-vnet
    - **Region:** Wählen Sie dieselbe Region aus, in der Ihre Ressourcengruppe erstellt wurde.

1. Wählen Sie **Überprüfen + Erstellen**, legen Sie die Einstellungen für das neue virtuelle Netzwerk fest und wählen Sie dann **Erstellen**.

## Bereitstellung einer Azure SQL-Datenbank im Azure-Portal

1. Suchen Sie im Azure-Portal im Suchfeld oben nach *SQL-Datenbanken* und wählen Sie dann **SQL-Datenbanken** aus der Liste der Optionen aus.

1. Wählen Sie im Bereich **SQL-Datenbanken** die Option **+ Erstellen**.

1. Wählen Sie auf der Seite **SQL-Datenbank erstellen** auf der Registerkarte **Grundlagen** die folgenden Optionen und wählen Sie dann **Weiter: Netzwerk**.

    - **Abonnement:** &lt;Ihr Abonnement&gt;
    - **Ressourcengruppe:** Beginnen Sie mit *DP300* oder der Ressourcengruppe, die Sie zuvor ausgewählt haben.
    - **Datenbankname:** AdventureWorksLT
    - **Server:** wählen Sie den Link **Neu erstellen**. Die Seite **SQL-Datenbank-Server erstellen** wird geöffnet. Geben Sie die Serverdetails wie folgt an:
        - **Servername:** dp300-lab-&lt;Ihre Initialen (in Kleinbuchstaben)&gt; und bei Bedarf eine zufällige 5-stellige Zahl (der Servername muss weltweit eindeutig sein).
        - **Standort:** &lt;Ihre lokale Region, die mit der für Ihre Ressourcengruppe ausgewählten Region übereinstimmt, andernfalls kann der Vorgang fehlschlagen.&gt;
        - **Authentifizierungsmethode**: Verwenden Sie SQL-Authentifizierung.
        - **Serveradministratoranmeldung:** dp300admin
        - **Kennwort:** Wählen Sie ein komplexes Kennwort aus, und notieren Sie es.
        - **Kennwort bestätigen:** Wählen Sie dasselbe zuvor ausgewählte Kennwort aus.
    - Wählen Sie **OK**, um zur Seite **SQL-Datenbank erstellen** zurückzukehren.
    - **Möchten Sie einen Pool für elastische SQL-Datenbanken verwenden?** Setzen Sie dies auf **Nein**.
    - **Workloadumgebung**: Entwicklung
    - Wählen Sie unter der Option **Compute + Speicher** den Link **Datenbank konfigurieren**. Auf der Seite **Konfigurieren** wählen Sie im Dropdown-Menü **Dienstebene** die Option **Basic** und dann **Anwenden**.

1. Für die Option **Sicherungsspeicherredundanz** behalten Sie den Standardwert bei: **Lokalredundanter Sicherungsspeicher**.

1. Wählen Sie dann **Weiter: Netzwerk**.

1. Wählen Sie auf der Registerkarte **Netzwerk** für die Option **Netzwerkverbindung** das Optionsfeld **Privater Endpunkt**.

1. Wählen Sie dann den Link **+ Privaten Endpunkt hinzufügen** unter der Option **Private Endpunkte**.

1. Füllen Sie das rechte Fenster **Privaten Endpunkt erstellen** wie folgt aus:

    - **Abonnement:** &lt;Ihr Abonnement&gt;
    - **Ressourcengruppe:** Beginnen Sie mit *DP300* oder der Ressourcengruppe, die Sie zuvor ausgewählt haben.
    - **Standort:** &lt;Ihre lokale Region, die mit der für Ihre Ressourcengruppe ausgewählten Region übereinstimmt, andernfalls kann der Vorgang fehlschlagen.&gt;
    - Endgerät**Name:** DP-300-SQL-Endpunkt
    - **Untergeordnete Zielressource:** SqlServer
    - **Virtuelles Netzwerk:** lab02-vnet
    - **Subnetz:** lab02-vnet/default (10.x.0.0/24)
    - **Integration mit einer private DNS-Zone**: Ja
    - **Private DNS Zone:** Standardwert unverändert lassen
    - Legen Sie die Einstellungen fest, und wählen Sie dann **OK**.  

1. Der neue Endpunkt wird in der Liste **Private Endpunkte** aufgeführt.

1. Wählen Sie **Weiter: Sicherheit**, und dann **Weiter: Zusätzliche Einstellungen** aus.  

1. Wählen Sie auf der Seite **Zusätzliche Einstellungen** die Option **Beispiel** für die Option **Vorhandene Daten verwenden** aus. Wählen Sie **OK** aus, wenn eine Popupmeldung für die Beispieldatenbank angezeigt wird.

1. Klicken Sie auf **Überprüfen + erstellen**.

1. Überprüfen Sie die Einstellungen, bevor Sie auf **Erstellen** klicken.

1. Klicken Sie nach Abschluss der Bereitstellung auf **Zu Ressource wechseln**.

## Aktivieren des Zugriffs auf eine Azure SQL-Datenbank

1. Wählen Sie auf der Seite **SQL-Datenbank** den Abschnitt **Übersicht** und wählen Sie dann den Link für den Servernamen im oberen Abschnitt.

1. Wählen Sie auf dem Navigationsblatt für SQL-Server die Option **Netzwerk** im **Abschnitt Sicherheit**.

1. Wählen Sie auf der Registerkarte **Öffentlicher Zugriff** **Ausgewählte Netzwerke** aus.

1. Wählen Sie **+ Client IPv4-Adresse hinzufügen** aus. Dadurch wird eine Firewallregel hinzugefügt, damit Ihre aktuelle IP-Adresse auf den SQL-Server zugreifen kann.

1. Markieren Sie die Eigenschaft **Zugriff von Azure-Diensten und -Ressourcen auf diesen Server zulassen**.

1. Wählen Sie **Speichern**.

---

## Verbinden mit einer Azure SQL-Datenbank in SQL Server Management Studio

1. Wählen Sie im Azure-Portal die **SQL-Datenbanken** im linken Navigationsbereich. Und wählen Sie dann die Datenbank **AdventureWorksLT**.

1. Kopieren Sie den Wert **Servername** von der Seite **Übersicht**.

1. Starten Sie SQL Server Management Studio auf dem virtuellen Lab-Computer, falls vorhanden oder auf Ihrem lokalen Computer, falls nicht.

1. Fügen Sie im Dialog **Mit dem Server verbinden** den **Servernamen** ein, den Sie aus dem Azure-Portal kopiert haben.

1. Wählen Sie in der Dropdown-Liste **Authentifizierung** die Option **SQL Server-Authentifizierung**.

1. Geben Sie in das Feld **Anmeldung** **dp300admin** ein.

1. Geben Sie in das Feld **Kennwort** das Kennwort ein, das bei der Erstellung des SQL-Servers ausgewählt wurde.

1. Wählen Sie **Verbinden**.

1. SQL Server Management Studio stellt eine Verbindung zu Ihrem Azure SQL-Datenbankserver her. Sie können den Server und dann den Knoten **Datenbanken** erweitern, um die Datenbank *AdventureWorksLT* zu sehen.

## Abfrage einer Azure SQL-Datenbank mit SQL Server Management Studio

1. Klicken Sie in SQL Server Management Studio mit der rechten Maustaste auf die Datenbank *AdventureWorksLT* und wählen Sie **Neue Abfrage**.

1. Fügen Sie die folgende SQL-Anweisung in das Abfragefenster ein:

    ```sql
    SELECT TOP 10 cust.[CustomerID], 
        cust.[CompanyName], 
        SUM(sohead.[SubTotal]) as OverallOrderSubTotal
    FROM [SalesLT].[Customer] cust
        INNER JOIN [SalesLT].[SalesOrderHeader] sohead
             ON sohead.[CustomerID] = cust.[CustomerID]
    GROUP BY cust.[CustomerID], cust.[CompanyName]
    ORDER BY [OverallOrderSubTotal] DESC
    ```

1. Wählen Sie die Schaltfläche **Ausführen** in der Symbolleiste, um die Abfrage auszuführen.

1. Im Bereich **Ergebnisse** sehen Sie sich die Ergebnisse der Abfrage an.

1. Klicken Sie mit der rechten Maustaste auf die Datenbank *AdventureWorksLT*, und wählen Sie **Neue Abfrage** aus.

1. Fügen Sie die folgende SQL-Anweisung in das Abfragefenster ein:

    ```sql
    SELECT TOP 10 cat.[Name] AS ProductCategory, 
        SUM(detail.[OrderQty]) AS OrderedQuantity
    FROM salesLT.[ProductCategory] cat
        INNER JOIN [SalesLT].[Product] prod
            ON prod.[ProductCategoryID] = cat.[ProductCategoryID]
        INNER JOIN [SalesLT].[SalesOrderDetail] detail
            ON detail.[ProductID] = prod.[ProductID]
    GROUP BY cat.[name]
    ORDER BY [OrderedQuantity] DESC
    ```

1. Wählen Sie die Schaltfläche **Ausführen** in der Symbolleiste, um die Abfrage auszuführen.

1. Im Bereich **Ergebnisse** sehen Sie sich die Ergebnisse der Abfrage an.

1. Schließen Sie SQL Server Management Studio. Wählen Sie **Nein**, wenn Sie zum Speichern der Änderungen aufgefordert werden.

---

## Bereinigen von Ressourcen

Wenn Sie den virtuellen Computer nicht für andere Zwecke verwenden, können Sie die in dieser Übung erstellten Ressourcen bereinigen.

### Löschen der Ressourcengruppe

Wenn Sie eine neue Ressourcengruppe für dieses Lab erstellt haben, können Sie die Ressourcengruppe löschen, um alle in diesem Lab erstellten Ressourcen zu entfernen.

1. Wählen Sie im Azure-Portal **Ressourcengruppen** im linken Navigationsbereich oder suchen Sie in der Suchleiste nach **Ressourcengruppen** und wählen Sie die Option aus den Ergebnissen aus.

1. Wechseln Sie zur Ressourcengruppe, die Sie für dieses Lab erstellt haben. Die Ressourcengruppe enthält den virtuellen Computer und andere in dieser Übung erstellte Ressourcen.

1. Wählen Sie **Ressourcengruppe löschen** aus dem Menü ganz oben aus.

1. Geben Sie im Dialog **Ressourcengruppe löschen** den Namen der zu bestätigenden Ressourcengruppe ein und wählen Sie **Löschen**.

1. Warten Sie, bis die Ressourcengruppe gelöscht wurde.

1. Schließen Sie das Azure-Portal.

### Löschen Sie nur die Lab-Ressourcen

Wenn Sie keine neue Ressourcengruppe für dieses Lab erstellt haben und die Ressourcengruppe und die vorherigen Ressourcen intakt lassen möchten, können Sie die in dieser Übung erstellten Ressourcen weiterhin löschen.

1. Wählen Sie im Azure-Portal **Ressourcengruppen** im linken Navigationsbereich oder suchen Sie in der Suchleiste nach **Ressourcengruppen** und wählen Sie die Option aus den Ergebnissen aus.

1. Wechseln Sie zur Ressourcengruppe, die Sie für dieses Lab erstellt haben. Die Ressourcengruppe enthält den virtuellen Computer und andere in dieser Übung erstellte Ressourcen.

1. Wählen Sie alle Ressourcen aus, denen der zuvor im Lab angegebene SQL Server-Name vorangestellt ist. Wählen Sie außerdem das virtuelle Netzwerk und die private DNS-Zone aus, die Sie erstellt haben.

1. Wählen Sie im Menü oben **Löschen** aus.

1. Im Dialog **Ressourcen löschen** geben Sie **Löschen** ein und wählen **Löschen**.

1. Wählen Sie erneut **Löschen**, um die Löschung der Ressourcen zu bestätigen.

1. Warten Sie, bis die Ressourcen gelöscht wurden.

1. Schließen Sie das Azure-Portal.

---

Sie haben dieses Lab erfolgreich abgeschlossen.

In dieser Übung haben Sie gesehen, wie Sie eine Azure SQL-Datenbank mit einem Endpunkt für das virtuelle Netzwerk bereitstellen. Sie konnten auch eine Verbindung mit der SQL-Datenbank herstellen, die Sie mit SQL Server Management Studio erstellt haben.
