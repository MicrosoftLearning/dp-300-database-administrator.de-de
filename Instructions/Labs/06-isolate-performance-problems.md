---
lab:
  title: 'Lab 6: Isolieren von Leistungsproblemen durch Überwachung'
  module: Monitor and optimize operational resources in Azure SQL
---

# Isolieren von Leistungsproblemen durch Überwachung

**Geschätzte Dauer: 30 Minuten**

Die Kursteilnehmer planen anhand der in den Lektionen gewonnenen Informationen die Leistungen für ein digitales Transformationsprojekt in AdventureWorks. Sie untersuchen das Azure-Portal und andere Tools und bestimmen, wie sie Tools zur Identifizierung und Lösung von Leistungsproblemen einsetzen können.

Sie wurden als Datenbankadministrator eingestellt, um leistungsbezogene Probleme zu ermitteln und geeignete Lösungen zu erarbeiten, um gefundene Probleme zu beheben. Sie müssen das Azure-Portal verwenden, um die Leistungsprobleme zu identifizieren, und Methoden vorschlagen, um diese zu beheben.

**Hinweis:** Diese Übungen bitten Sie, T-SQL-Code zu kopieren und einzufügen, und verwenden vorhandene SQL-Ressourcen. Überprüfen Sie, ob der Code korrekt kopiert wurde, bevor Sie ihn ausführen.

## Überprüfen der CPU-Auslastung im Azure-Portal

1. Starten Sie auf dem virtuellen Lab-Computer eine Browsersitzung, und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com/). Stellen Sie eine Verbindung zum Portal her. Verwenden Sie dafür **Benutzernamen** und **Kennwort** von Azure, die auf der Registerkarte **Ressourcen** für diesen virtuellen Lab-Computer bereitgestellt werden.

    ![Abbildung 1](../images/dp-300-module-01-lab-01.png)

1. Suchen Sie im Azure-Portal über die Suchleiste oben nach „SQL-Server“, und wählen Sie aus der Optionsliste **SQL-Server** aus.

    ![Screenshot einer automatisch generierten Beschreibung eines Posts in sozialen Medien](../images/dp-300-module-04-lab-1.png)

1. Wählen Sie den Servernamen **dp300-lab-XXXXXXXX** aus, um zur Detailseite zu gelangen. (Möglicherweise ist Ihrem SQL Server eine andere Ressourcengruppe und ein anderer Speicherort zugewiesen.)

    ![Screenshot einer automatisch generierten Beschreibung eines Posts in sozialen Medien](../images/dp-300-module-04-lab-2.png)

1. Navigieren Sie auf dem Hauptblatt Ihres Azure SQL-Servers zum Abschnitt **Einstellungen**, wählen Sie **SQL-Datenbanken** und dann den Namen der Datenbank aus.

    ![Screenshot zur Auswahl der AdventureWorksLT-Datenbank](../images/dp-300-module-05-lab-04.png)

1. Wählen Sie auf der Hauptseite der Datenbank die Option **Serverfirewall festlegen** aus.

    ![Screenshot mit Auswahl von „Serverfirewall festlegen“](../images/dp-300-module-06-lab-01.png)

1. Wählen Sie auf der Seite **Netzwerk** die Option **+ Ihre Client-IPv4-Adresse (Ihre IP-Adresse) hinzufügen** und dann **Speichern** aus.

    ![Screenshot mit Auswahl von „Client-IP-Adresse hinzufügen“](../images/dp-300-module-06-lab-02.png)

1. Wählen Sie in der Navigationsleiste über **Netzwerk** den Link aus, der mit **AdventureWorksLT** beginnt.

    ![Screenshot mit Auswahl von AdventureWorks](../images/dp-300-module-06-lab-03.png)

1. Klicken Sie in der linken Navigationsleiste auf **Abfrage-Editor (Vorschau)**.

    ![Screenshot mit Auswahl des Links Abfrage-Editor (Vorschau)](../images/dp-300-module-06-lab-04.png)

    **Hinweis**: Dieses Feature befindet sich in der Vorschau.

1. Geben Sie im Feld **Kennwort** **P@ssw0rd01** ein, und wählen Sie **OK**.

    ![Screenshot mit den Verbindungseigenschaften des Abfrage-Editors](../images/dp-300-module-06-lab-05.png)

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

    ![Screenshot mit der Abfrage](../images/dp-300-module-06-lab-06.png)

1. Warten Sie, bis die Abfrage beendet ist.

1. Wählen Sie im Blade für die Datenbank **AdventureWorksLT** das Symbol **Metriken** im Abschnitt **Überwachung**.

    ![Screenshot mit Auswahl des Symbols für Metriken](../images/dp-300-module-06-lab-07.png)

1. Ändern Sie die Menüoption **Metrik** so, dass sie den **CPU-Prozentsatz** widerspiegelt, und wählen Sie dann eine **Aggregation** von **Mittelwert** aus. Dadurch wird der durchschnittliche CPU-Prozentsatz für den angegebenen Zeitrahmen angezeigt.

    ![Screenshot mit CPU-Prozentsatz](../images/dp-300-module-06-lab-08.png)

1. Beobachten Sie den CPU-Durchschnitt über einen Zeitraum hinweg. Ihre Ergebnisse können leicht abweichen. Alternativ können Sie die Abfrage auch mehrmals ausführen, um aussagekräftigere Ergebnisse zu erhalten.

    ![Screenshot mit der durchschnittlichen Aggregation](../images/dp-300-module-06-lab-09.png)

## Ermitteln von Abfragen mit hoher CPU-Auslastung

1. Suchen Sie im Abschnitt **Intelligente Leistung** des Blatts für die **AdventureWorksLT**-Datenbank nach dem Symbol für **Query Performance Insight**.

    ![Screenshot des Symbols Query Performance Insight](../images/dp-300-module-06-lab-10.png)

1. Wählen Sie **Einstellungen zurücksetzen** aus.

    ![Screenshot mit der Option „Einstellungen zurücksetzen“](../images/dp-300-module-06-lab-11.png)

1. Klicken Sie im Raster unterhalb des Diagramms auf die Abfrage. Wenn keine Abfrage angezeigt wird, warten Sie etwa zwei Minuten, und wählen Sie **Aktualisieren** aus.

    **Hinweis:** Bei Ihnen können Dauer und Abfrage-ID abweichen. Wenn Sie mehr als eine Abfrage sehen, klicken Sie auf jede einzelne, um die Ergebnisse zu verfolgen.

    ![Screenshot mit der durchschnittlichen Aggregation](../images/dp-300-module-06-lab-12.png)

Für diese Abfrage können Sie sehen, dass die Gesamtdauer über eine Minute betrug und dass sie etwa 10.000 Mal ausgeführt wurde.

In dieser Übung haben Sie gelernt, wie Sie Serverressourcen für eine Azure SQL-Datenbank untersuchen und potenzielle Probleme mit der Abfrageleistung mithilfe von Query Performance Insight identifizieren können.
