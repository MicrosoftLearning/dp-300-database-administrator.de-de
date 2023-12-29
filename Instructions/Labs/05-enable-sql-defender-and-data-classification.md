---
lab:
  title: 'Lab 5: Aktivieren von Microsoft Defender für SQL und Datenklassifizierung'
  module: Implement a Secure Environment for a Database Service
---

# Aktivieren von Microsoft Defender für SQL und Datenklassifizierung

**Geschätzte Dauer**: 20 Minuten

Die Kursteilnehmer nutzen die in den Lektionen erworbenen Informationen, um die Sicherheit im Azure-Portal und in der AdventureWorks-Datenbank zu konfigurieren und anschließend zu implementieren.

Sie wurden als verantwortlicher Datenbankadministrator eingestellt, um die Sicherheit der Datenbankumgebung zu gewährleisten. Für diese Aufgaben wird schwerpunktmäßig Azure SQL-Datenbank eingesetzt.

## Aktivieren von Microsoft Defender für SQL

1. Starten Sie auf dem virtuellen Lab-Computer eine Browsersitzung, und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com/). Stellen Sie eine Verbindung zum Portal her. Verwenden Sie dafür **Benutzernamen** und **Kennwort** von Azure, die auf der Registerkarte **Ressourcen** für diesen virtuellen Lab-Computer bereitgestellt werden.

    ![Abbildung 1](../images/dp-300-module-01-lab-01.png)

1. Suchen Sie im Azure-Portal über die Suchleiste oben nach „SQL-Server“, und wählen Sie aus der Optionsliste **SQL-Server** aus.

    ![Screenshot einer automatisch generierten Beschreibung eines Posts in sozialen Medien](../images/dp-300-module-04-lab-1.png)

1. Wählen Sie den Servernamen **dp300-lab-XXXXXXXX** aus, um zur Detailseite zu gelangen. (Möglicherweise ist Ihrem SQL Server eine andere Ressourcengruppe und ein anderer Speicherort zugewiesen.)

    ![Screenshot einer automatisch generierten Beschreibung eines Posts in sozialen Medien](../images/dp-300-module-04-lab-2.png)

1. Navigieren Sie auf dem Hauptblatt Ihres Azure SQL-Servers zum Abschnitt **Sicherheit**, und wählen Sie dort **Microsoft Defender for** aus.

    ![Screenshot der Auswahl der Option „Microsoft Defender for Cloud“](../images/dp-300-module-05-lab-01.png)

    Klicken Sie auf der Seite **Microsoft Defender for Cloud** auf **Microsoft Defender für die SQL aktivieren**.

1. Die folgende Benachrichtigung wird angezeigt, nachdem Azure Defender for SQL erfolgreich aktiviert wurde.

    ![Screenshot der Auswahl der Option „Konfigurieren“](../images/dp-300-module-05-lab-02_1.png)

1. Wählen Sie auf der Seite **Microsoft Defender für Cloud** den Link **Konfigurieren** aus. (Möglicherweise müssen Sie die Seite aktualisieren, um diese Option anzuzeigen.)

    ![Screenshot der Auswahl der Option „Konfigurieren“](../images/dp-300-module-05-lab-02.png)

1. Beachten Sie auf der Seite **Servereinstellungen**, dass der Umschalter unter **MICROSOFT DEFENDER FOR SQL** auf **EIN** festgelegt ist.

## Aktivieren der Datenklassifizierung

1. Navigieren Sie auf dem Hauptblatt Ihres Azure SQL-Servers zum Abschnitt **Einstellungen**, wählen Sie **SQL-Datenbanken** und dann den Namen der Datenbank aus.

    ![Screenshot zur Auswahl der AdventureWorksLT-Datenbank](../images/dp-300-module-05-lab-04.png)

1. Navigieren Sie auf dem Hauptblatt für die **AdventureWorksLT**-Datenbank zum Abschnitt **Sicherheit**, und wählen Sie **Datenermittlung und -klassifizierung** (Datenermittlung und -klassifizierung) aus.

    ![Screenshot zur Option „Datenermittlung und -klassifizierung“](../images/dp-300-module-05-lab-05.png)

1. Auf der Seite **Datenermittlung und -klassifizierung** wird eine Informationsmeldung mit folgendem Wortlaut angezeigt: **Derzeit wird die SQL Information Protection-Richtlinie verwendet. Wir haben 15 Spalten mit Klassifizierungsempfehlungen gefunden**. Wählen Sie diesen Link aus.

    ![Screenshot: angezeigte Klassifizierungsempfehlungen](../images/dp-300-module-05-lab-06.png)

1. Aktivieren Sie auf dem nächsten Bildschirm von **Datenermittlung und -klassifizierung** das Kontrollkästchen neben **Alle auswählen**, wählen Sie **Ausgewählte Empfehlungen annehmen** aus, und klicken Sie auf **Speichern**, um die Klassifizierungen in der Datenbank zu speichern.

    ![Screenshot: Option „Ausgewählte Empfehlungen annehmen“](../images/dp-300-module-05-lab-07.png)

1. Zurück im Bildschirm **Datenermittlung und -klassifizierung** stellen Sie fest, dass fünfzehn Spalten in fünf verschiedenen Tabellen erfolgreich klassifiziert wurden.

    ![Screenshot: Option „Ausgewählte Empfehlungen annehmen“](../images/dp-300-module-05-lab-08.png)

In dieser Übung haben Sie die Sicherheit einer Azure SQL-Datenbank durch die Aktivierung von Microsoft Defender for SQL verbessert. Außerdem haben Sie auf der Grundlage von Empfehlungen des Azure-Portals klassifizierte Spalten erstellt.
