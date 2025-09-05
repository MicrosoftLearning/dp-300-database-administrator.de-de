---
lab:
  title: 'Lab 1: Bereitstellen von SQL Server auf einer Azure-VM'
  module: Plan and Implement Data Platform Resources
---

# Bereitstellen von SQL Server auf einer Azure-VM

**Geschätzte Dauer: 30 Minuten**

Die Kursteilnehmenden erkunden das Azure-Portal und erstellen damit eine Azure-VM mit installiertem SQL Server 2022. Dann stellen sie über das Remotedesktopprotokoll eine Verbindung mit einem virtuellen Computer her.

Sie sind der Datenbankadministrator für AdventureWorks. Sie müssen eine Testumgebung erstellen, die für eine Machbarkeitsstudie verwendet werden kann. Für die Machbarkeitsstudie werden SQL Server auf einer Azure-VM und ein Backup der AdventureWorksDW-Datenbank verwendet. Sie müssen den virtuellen Computer einrichten, die Datenbank wiederherstellen und abfragen, um sicherzustellen, dass sie verfügbar ist.

## Bereitstellen von SQL Server auf einer Azure-VM

1. Starten Sie eine Browsersitzung, navigieren Sie zu [https://portal.azure.com](https://portal.azure.com/), und melden Sie sich mit dem Microsoft-Konto an, das mit Ihrem Azure-Abonnement verknüpft ist.

1. Suchen Sie oben auf der Seite nach der Suchleiste. Suchen Sie nach **Azure SQL**. Wählen Sie das unter **Dienste** angezeigte Suchergebnis für **Azure SQL**.

1. Wählen Sie auf dem Blatt **Azure Cosmos DB** die Option **Erstellen** aus.

1. Öffnen Sie auf dem Blatt **SQL-Bereitstellungsoption auswählen** das Dropdownfeld unter **SQL-VMs**. Wählen Sie die Option **Free SQL Server License: SQL Server 2022 Developer on Windows Server 2022** (Kostenlose SQL Server-Lizenz: SQL Server 2022-Entwickelnde unter Windows Server 2022) aus. Wählen Sie dann **Erstellen** aus.

1. Geben Sie auf der Seite **Virtuellen Computer erstellen** die folgenden Informationen ein und *belassen Sie alle anderen Optionen auf den Standardwerten*:

    - **Abonnement:** &lt;Ihr Abonnement&gt;
    - **Ressourcengruppe:** &lt;Ihre Ressourcengruppe&gt;
    - **Name der virtuellen Maschine:** AzureSQLServerVM
    - **Region:**&lt;Wählen Sie Ihre lokale Region aus, die mit der für Ihre Ressourcengruppe ausgewählten Region übereinstimmt.&gt;
    - **Verfügbarkeitsoptionen**: Keine Infrastrukturredundanz erforderlich
    - **Bild:** Kostenlose SQL Server-Lizenz: SQL Server 2022 Developer auf Windows Server 2022 – Gen2
    - **Ausführen mit Azure Spotrabatt:** Nein (deaktiviert)
    - **Größe:** Standard *D2s_v5* (2 VCPUs, 8 GiB Arbeitsspeicher) *Möglicherweise müssen Sie den Link „Alle Größen anzeigen“ auswählen, um diese Option zu sehen)*
    - **Benutzername des Admin-Kontos:**&lt;Wählen Sie einen Namen für Ihr Admin-Konto aus.&gt;
    - **Admin-Konto-Kennwort:**&lt;Wählen Sie ein sicheres Kennwort aus.&gt;
    - **Eingangsports auswählen:** RDP (3389)
    - **Möchten Sie eine vorhandene Windows Server-Lizenz verwenden?:** Nein (deaktiviert)

    > &#128221; Notieren Sie sich den Benutzernamen und das Kennwort für die spätere Verwendung.

1. Navigieren Sie zur Registerkarte **Datenträger**, und überprüfen Sie die Konfiguration.

1. Navigieren Sie zur Registerkarte **Netzwerke**, und überprüfen Sie die Konfiguration.

1. Navigieren Sie zur Registerkarte **Verwaltung**, und überprüfen Sie die Konfiguration.

    Vergewissern Sie sich, dass **„Auto_shutdown aktivieren"** deaktiviert ist.

1. Navigieren Sie zur Registerkarte **Erweitert**, und überprüfen Sie die Konfiguration.

1. Navigieren Sie zur Registerkarte **SQL Server-Einstellungen**, und überprüfen Sie die Konfiguration.

    > &#128221; Auf diesem Bildschirm können Sie auch den Speicher für Ihre SQL Server-VM konfigurieren. Die Vorlagen für SQL Server auf einer Azure-VM erstellen standardmäßig einen Premium-Datenträger mit einem Lesecache für Daten sowie einen Premium-Datenträger ohne Cache für das Transaktionsprotokoll und verwendet den lokalen SSD-Datenträger (D:\ unter Windows) für die tempdb.

1. Wählen Sie die Schaltfläche **Überprüfen + erstellen** aus. Wählen Sie dann **Erstellen** aus.

1. Warten Sie auf dem Blatt „Bereitstellung“, bis die Bereitstellung abgeschlossen ist. Die Bereitstellung der VM dauert etwa 5 bis 10 Minuten. Wenn Ihre Bereitstellung abgeschlossen ist, wählen Sie **Go to resource** (Zu Ressource wechseln) aus.

    > &#128221; Die Bereitstellung kann mehrere Minuten dauern.

1. Scrollen Sie auf der **Übersichtsseite** für die VM durch die Menüoptionen für die Ressource, um die verfügbaren Optionen zu überprüfen.

---

## Herstellen einer Verbindung mit SQL Server auf einer Azure-VM

1. Wählen Sie für den virtuellen Computer auf der Seite **Übersicht** das Pulldown-Menü **Verbinden** und dann **Verbinden** aus.

1. Wählen Sie im Bereich „Verbinden“ die Schaltfläche **RDP-Datei herunterladen** aus.

    > &#128221; Wenn die Fehlermeldung **Port-Voraussetzung nicht erfüllt** angezeigt wird. Stellen Sie sicher, dass Sie den Link auswählen, um eine Sicherheitsgruppenregel für eingehende Netzwerke mit dem Zielport hinzuzufügen, der im Feld *Portnummer* erwähnt wird.

1. Öffnen Sie die heruntergeladene RDP-Datei. Wenn Sie in einem Dialogfeld gefragt werden, ob Sie eine Verbindung herstellen möchten, wählen Sie **Verbinden** aus.

1. Geben Sie während des Bereitstellungsprozesses des virtuellen Computers den Benutzernamen und das Kennwort ein. Wählen Sie dann **OK** aus.

1. Wenn Sie im Dialogfeld **Remotedesktop verbinden** gefragt werden, ob Sie eine Verbindung herstellen möchten, wählen Sie **Ja** aus.

1. Wählen Sie die Suchleiste neben der Windows-Startschaltfläche und geben Sie SSMS ein. Wählen Sie **Microsoft SQL Server Management Studio** von der Liste aus.  

1. Wenn SSMS geöffnet wird, beachten Sie, dass das Dialogfeld **Mit Server verbinden** mit dem Standardinstanznamen vorausgefüllt ist. Aktivieren Sie die Option **Serverzertifikat vertrauen** und wählen Sie dann **Verbinden**.

1. Schließen Sie SSMS, indem Sie oben rechts auf **X** klicken.

1. Sie können jetzt die Verbindung mit dem virtuellen Computer trennen, um die RDP-Sitzung zu schließen.

Das Azure-Portal bietet Ihnen leistungsstarke Tools für die Verwaltung einer SQL Server-Instanz, die auf einer VM gehostet wird. Diese Tools umfassen die Kontrolle über das automatisierte Patchen, automatisierte Sicherungen und eine einfache Möglichkeit zum Einrichten der Hochverfügbarkeit.

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

1. Wählen Sie alle Ressourcen mit dem Präfix des virtuellen Computers aus, den Sie zuvor in der Übung angegeben haben.

1. Wählen Sie im Menü oben **Löschen** aus.

1. Im Dialog **Ressourcen löschen** geben Sie **Löschen** ein und wählen **Löschen**.

1. Wählen Sie erneut **Löschen**, um die Löschung der Ressourcen zu bestätigen.

1. Warten Sie, bis die Ressourcen gelöscht wurden.

1. Schließen Sie das Azure-Portal.

---

Sie haben dieses Lab erfolgreich abgeschlossen.
