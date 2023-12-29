---
lab:
  title: 'Lab 2: Bereitstellen einer Azure SQL-Datenbank-Instanz'
  module: Plan and Implement Data Platform Resources
---

# Bereitstellen einer Azure SQL-Datenbank

**Geschätzte Dauer: 40 Minuten**

Die Kursteilnehmer konfigurieren grundlegende Ressourcen, die für die Bereitstellung einer Azure SQL-Datenbank mit einem virtuellen Netzwerkendpunkt erforderlich sind. Konnektivität für die SQL-Datenbank wird mithilfe von Azure Data Studio aus der VM des Labs überprüft.

Als Datenbankadministrator richten Sie eine neue SQL-Datenbank ein, einschließlich eines Endpunkts für das virtuelle Netzwerk, um die Sicherheit der Bereitstellung zu erhöhen und zu vereinfachen. Azure Data Studio wird verwendet, um das Abfragen von Daten mithilfe eines SQL-Notebooks und das Überprüfen der Ergebnisse auszuwerten.

## Navigieren Sie zum Azure-Portal.

1. Starten Sie auf dem virtuellen Lab-Computer eine Browsersitzung, und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com/). Stellen Sie eine Verbindung zum Portal her. Verwenden Sie dafür **Benutzernamen** und **Kennwort** von Azure, die auf der Registerkarte **Ressourcen** für diesen virtuellen Lab-Computer bereitgestellt werden.

    ![Abbildung 1](../images/dp-300-module-01-lab-01.png)

1. Suchen Sie im Azure-Portal im Suchfeld oben nach „Ressourcengruppen“ und wählen Sie dann **Ressourcengruppen** aus der Liste der Optionen aus.

    ![Abbildung 1](../images/dp-300-module-02-lab-45.png)

1. Überprüfen Sie auf der Seite **Ressourcengruppe** die aufgelistete Ressourcengruppe (sie sollte mit *contoso-rg* beginnen), notieren Sie sich den **Standort**, der Ihrer Ressourcengruppe zugewiesen ist. Sie werden ihn in der nächsten Übung verwenden.

    **Hinweis:** Möglicherweise haben Sie einen anderen Speicherort zugewiesen.

    ![Abbildung 1](../images/dp-300-module-02-lab-46.png)

## Erstellen eines virtuellen Netzwerks

1. Wählen Sie auf der Startseite von Azure-Portal das Menü links aus.  

    ![Abbildung 2](../images/dp-300-module-02-lab-01_1.png)

1. Klicken Sie im linken Navigationsbereich auf **virtuelle Netzwerke**.  

1. Klicken Sie auf **+ Erstellen**, um die Seite **Virtuelles Netzwerk erstellen** zu öffnen. BasicGeben Sie auf der Registerkarte **Grundlagen** die folgenden Informationen an:

    - **Abonnement:** &lt;Ihr Abonnement&gt;
    - **Ressourcengruppe:** beginnend mit *contoso-rg*
    - **Name:** lab02-vnet
    - **Region:** Wählen Sie dieselbe Region aus, in der Ihre Ressourcengruppe erstellt wurde.

1. Klicken Sie auf **Bewerten + erstellen**, bewerten Sie die Einstellungen für das neue virtuelle Netzwerk, und klicken Sie dann auf **Erstellen**.

1. Konfigurieren Sie den IP-Bereich des virtuellen Netzwerks für den Azure SQL-Datenbankendpunkt. Navigieren Sie dafür zum erstellten virtuellen Netzwerk und klicken Sie im Bereich **Einstellungen** auf **Subnetze**.

1. Klicken Sie auf den **Standardlink** des Subnetzes. Beachten Sie, dass möglicherweise ein anderer **Subnetzadressbereich** angezeigt wird.

1. Erweitern Sie im Bereich **Subnetz bearbeiten** auf der rechten Seite das Dropdownmenü **Dienste**, und klicken Sie auf **Microsoft.Sql**. Wählen Sie **Speichern** aus.

## Bereitstellen einer Azure SQL-Datenbank

1. Suchen Sie im Azure-Portal über die Suchleiste oben nach „SQL-Datenbank“, und wählen Sie aus der Optionsliste **SQL-Datenbank** aus.

    ![Abbildung 5](../images/dp-300-module-02-lab-10.png)

1. Klicken Sie auf der Seite **SQL-Datenbanken** auf **+ Erstellen**.

    ![Abbildung 6](../images/dp-300-module-02-lab-10_1.png)

1. Wählen Sie auf der Seite **SQL-Datenbank erstellen** die folgenden Optionen auf der **Registerkarte Grundlagen** aus, und klicken Sie dann auf **Weiter: Netzwerk**.

    - **Abonnement:** &lt;Ihr Abonnement&gt;
    - **Ressourcengruppe:** beginnend mit *contoso-rg*
    - **Datenbankname:** AdventureWorksLT
    - **Server:** Klicken Sie auf **Neuen Link erstellen**. Die Seite **SQL-Datenbank-Server erstellen** wird geöffnet. Geben Sie die Serverdetails wie folgt an:
        - **Servername:** dp300-lab-&lt;-Ihre Initialen (in Kleinbuchstaben)&gt; (Servername muss global eindeutig sein.)
        - **Standort:** &lt;Ihre lokale Region, die mit der für Ihre Ressourcengruppe ausgewählten Region übereinstimmt, andernfalls kann der Vorgang fehlschlagen.&gt;
        - **Authentifizierungsmethode**: Verwenden Sie SQL-Authentifizierung.
        - **Serveradministratoranmeldung:** dp300admin
        - **Kennwort**: dp300P@ssword!
        - **Kennwort bestätigen:** dp300P@ssword!

        Die Seite **SQL-Datenbank Server** sollte ähnlich wie die folgende Seite aussehen. Klicken Sie dann auf **OK**.

        ![Abbildung 7](../images/dp-300-module-02-lab-11.png)

    -  Stellen Sie zurück auf der Seite **SQL-Datenbank erstellen** sicher, dass **Pool für elastische Datenbanken verwenden?** auf **Nein** festgelegt ist.
    -  Wählen Sie unter **Compute und Speicher** den Link **Datenbank konfigurieren** aus. Wählen Sie auf der Seite **Konfigurieren** aus der Dropdownliste **Dienstebene** die Option **Einfach** und dann **Übernehmen** aus.

    **Hinweis:** Notieren Sie sich diesen Servernamen und Ihre Anmeldeinformationen. Sie werden ihn in späteren Labs benötigen.

1. Behalten Sie für die Option **Sicherungsspeicherredundanz** den Standardwert bei: **Georedundanter Sicherungsspeicher**.

1. Klicken Sie auf **Weiter: Netzwerk**.

1. Klicken Sie auf der **Registerkarte Netzwerk** für die Option **Netzwerkkonnektivität** auf das Optionsfeld **Privater Endpunkt**.

    ![Abbildung 8](../images/dp-300-module-02-lab-14.png)

1. Klicken Sie dann unter **Private Endpunkte** auf den Link **Privaten Endpunkt hinzufügen**.

    ![Abbildung 9](../images/dp-300-module-02-lab-15.png)

1. Füllen Sie das rechte Fenster **Privaten Endpunkt erstellen** wie folgt aus:

    - **Abonnement:** &lt;Ihr Abonnement&gt;
    - **Ressourcengruppe:** beginnend mit *contoso-rg*
    - **Standort:** &lt;Ihre lokale Region, die mit der für Ihre Ressourcengruppe ausgewählten Region übereinstimmt, andernfalls kann der Vorgang fehlschlagen.&gt;
    - Endgerät**Name:** DP-300-SQL-Endpunkt
    - **Untergeordnete Zielressource:** SqlServer
    - **Virtuelles Netzwerk:** lab02-vnet
    - **Subnetz:** lab02-vnet/default (10.x.0.0/24)
    - **Integration mit einer private DNS-Zone**: Ja
    - **Private DNS Zone:** Standardwert unverändert lassen
    - Überprüfen Sie die Einstellungen, und klicken Sie dann auf **OK**.  

    ![Abbildung 10](../images/dp-300-module-02-lab-16.png)

1. Der neue Endpunkt wird in der Liste **Private Endpunkte** aufgeführt.

    ![Abbildung 11](../images/dp-300-module-02-lab-17.png)

1. Klicken Sie auf **Weiter: Sicherheit** und dann **Weiter: Zusätzliche Einstellungen**.  

1. Wählen Sie auf der Seite **Zusätzliche Einstellungen** die Option **Beispiel** für die Option **Vorhandene Daten verwenden** aus. Wählen Sie **OK** aus, wenn eine Popupmeldung für die Beispieldatenbank angezeigt wird.

    ![Abbildung 12](../images/dp-300-module-02-lab-18.png)

1. Klicken Sie auf **Überprüfen und erstellen**.

1. Überprüfen Sie die Einstellungen, und klicken Sie auf **Erstellen**.

1. Klicken Sie nach Abschluss der Bereitstellung auf **Zu Ressource wechseln**.

## Zugriff auf eine Azure SQL-Datenbank aktivieren

1. Wählen Sie auf der Seite **SQL-Datenbank** den Abschnitt **Übersicht** und dann den Link für den Servernamen im oberen Abschnitt aus:

    ![Abbildung 13](../images/dp-300-module-02-lab-19.png)

1. Wählen Sie auf dem Navigationsblatt für SQL-Server die Option **Netzwerk** im **Abschnitt Sicherheit**.

    ![Abbildung 14](../images/dp-300-module-02-lab-20.png)

1. Wählen Sie auf der Registerkarte **Öffentlicher Zugriff** die Option **Ausgewählte Netzwerke** aus, und überprüfen Sie dann die **Azure-Dienste und -Ressourcen für den Zugriff auf diese Servereigenschaft**. Klicken Sie auf **Speichern**.

    ![Abbildung 15](../images/dp-300-module-02-lab-21.png)

## Herstellen einer Verbindung mit Azure SQL-Datenbank in Azure Data Studio

1. Starten Sie Azure Data Studio auf dem virtuellen Computer für dieses Lab.

    - Möglicherweise wird dieses Popup beim ersten Start von Azure Data Studio angezeigt. Wenn ja, klicken Sie auf **Ja (empfohlen)**.  

        ![Abbildung 16](../images/dp-300-module-02-lab-22.png)

1. Klicken Sie in Azure Data Studio in der oberen linken Ecke auf die Schaltfläche **Verbindungen** und **Verbindungen hinzufügen**.

    ![Abbildung 17](../images/dp-300-module-02-lab-25.png)

1. Geben Sie in der Seitenleiste **Verbindung** im Abschnitt **Details zur Verbindung** die Verbindungsinformationen für eine Verbindung mit der zuvor erstellten SQL-Datenbank ein.

    - Verbindungstyp: **Microsoft SQL Server**
    - Geben Sie den Namen des zuvor erstellten SQL-Servers ein. Beispiel: **dp300-lab-xxxxxxxx.database.windows.net** (Wobei "xxxxxxxx" eine zufällige Zahl ist)
    - Authentifizierungstyp: **SQL-Anmeldung**
    - Benutzername: **dp300admin**
    - Kennwort: **dp300P@ssword!**
    - Erweitern Sie das Datenbankdropdown, und wählen Sie **AdventureWorksLT** aus. 
        - **HINWEIS:** Möglicherweise werden Sie aufgefordert, eine Firewallregel hinzuzufügen, die ihren Client-IP-Zugriff auf diesen Server ermöglicht. Wenn Sie dazu aufgefordert werden, klicken Sie auf **Konto hinzufügen** und melden Sie sich bei Ihrem Azure-Konto an. Klicken Sie auf dem Bildschirm **Neue Firewallregel erstellen** auf **OK**.

        ![Abbildung 18](../images/dp-300-module-02-lab-26.png)

        Alternativ können Sie auf Azure-Portal manuell eine Firewallregel für Ihren SQL Server erstellen, indem Sie zu Ihrem SQL-Server navigieren, **Netzwerk** auswählen und dann **Ihre Client-IPv4-Adresse (Ihre IP-Adresse) hinzufügen**

        ![Abbildung 18](../images/dp-300-module-02-lab-47.png)

    Füllen Sie wieder zurück in der Randleiste „Verbindung“ Ihre Verbindungsdetails weiter aus:  

    - Behalten Sie für die Servergruppe die Einstellung **&lt;Standard&gt;** bei.
    - Unter „Name (optional)“ kann bei Bedarf ein Anzeigename der Datenbank angegeben werden.
    - Überprüfen Sie die Einstellungen, und klicken Sie auf **Verbinden**  

    ![Abbildung 19](../images/dp-300-module-02-lab-27.png)

1. Azure Data Studio stellt eine Verbindung zur Datenbank her und zeigt einige grundlegende Informationen zur Datenbank, einschließlich einer partiellen Objektliste.

    ![Abbildung 20](../images/dp-300-module-02-lab-28.png)

## Abfragen einer Azure SQL-Datenbank-Instanz mit einem SQL-Notebook

1. Klicken Sie in Azure Data Studio, das mit der AdventureWorksLT-Datenbank dieses Labs verbunden ist, auf die Schaltfläche **Neues Notizbuch**.

    ![Abbildung 21](../images/dp-300-module-02-lab-29.png)

1. Klicken Sie auf den Link **+ Text**, um im Notizbuch ein neues Textfeld hinzuzufügen.  

    ![Abbildung 22](../images/dp-300-module-02-lab-30.png)

**Hinweis:** Im Notebook können Sie Nur-Text einbetten, um Abfragen oder Ergebnismengen zu erläutern.

1. Geben Sie den Text **Top Ten Customers by Order SubTotal** (Die zehn größten Kunden nach Bestellzwischensumme) ein, bei Bedarf auch in Fettdruck.

    ![Screenshot einer automatisch generierten Mobiltelefonbeschreibung](../images/dp-300-module-02-lab-31.png)

1. Klicken Sie auf die Schaltfläche **+ Zelle** und **Codezelle**, um am Ende des Notizbuchs eine neue Codezelle einzufügen.  

    ![Abbildung 23](../images/dp-300-module-02-lab-32.png)

5. Fügen Sie die folgende SQL-Anweisung in die neue Zelle ein:

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

1. Klicken Sie auf den blauen Kreis mit dem Pfeil, um die Abfrage auszuführen. Beachten Sie, dass die Ergebnisse in die Zelle mit der Abfrage eingefügt werden.

1. Klicken Sie auf die Schaltfläche **+ Text**, um eine neue Textzelle hinzuzufügen.

1. Geben Sie den Text **Top Ten Ordered Product Categories** (Die zehn meistbestellten Produktkategorien) ein, bei Bedarf auch in Fettdruck.

1. Klicken Sie noch einmal auf **+ Code**, um eine neue Zelle hinzuzufügen, und fügen Sie die folgende SQL-Anweisung in die Zelle ein.

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

1. Klicken Sie auf den blauen Kreis mit dem Pfeil, um die Abfrage auszuführen.

1. Klicken Sie in der Symbolleiste auf **Alle ausführen**, um alle Zellen im Notebook auszuführen und die Ergebnisse anzuzeigen.

    ![Abbildung 17](../images/dp-300-module-02-lab-33.png)

1. Speichern Sie das Notizbuch in Azure Data Studio im Menü „Datei“ (entweder „Speichern“ oder „Speichern unter“) im **Pfad C:\Labfiles\Bereitstellen einer Azure SQL-Datenbank**. (Erstellen Sie die Ordnerstruktur, falls sie nicht vorhanden ist). (Stellen Sie sicher, dass die Dateierweiterung **.ipynb** lautet.)

1. Schließen Sie die Registerkarte für das Notizbuch in Azure Data Studio. Klicken Sie im Menü „Datei“ auf „Datei öffnen“, und öffnen Sie das Notebook, das Sie soeben gespeichert haben. Beachten Sie, dass die Abfrageergebnisse zusammen mit den Abfragen im Notebook gespeichert wurden.

In dieser Übung haben Sie gesehen, wie Sie eine Azure SQL-Datenbank mit einem Endpunkt für das virtuelle Netzwerk bereitstellen. Sie konnten auch eine Verbindung mit der SQL-Datenbank herstellen, die Sie mit SQL Server Management Studio erstellt haben.
