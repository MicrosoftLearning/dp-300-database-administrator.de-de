---
lab:
  title: 'Lab 12: Erstellen einer CPU-Statuswarnung für eine SQL Server-Instanz'
  module: Automate database tasks for Azure SQL
---

# Erstellen einer CPU-Statuswarnung für eine SQL Server-Instanz auf Azure

**Geschätzte Dauer**: 30 Minuten

Sie wurden als Senior Data Engineer eingestellt, um die Automatisierung der alltäglichen Datenbankverwaltungsvorgänge voranzutreiben. Diese Automatisierung soll sicherstellen, dass die Datenbanken für AdventureWorks weiterhin mit maximaler Leistung betrieben werden und Methoden für die Alarmierung auf der Grundlage bestimmter Kriterien bereitstellen.

## Erstellen einer Warnung, wenn eine CPU einen Durchschnitt von 80 Prozent überschreitet

1. Geben Sie in der Suchleiste am oberen Rand des Azure-Portals **SQL** ein, und wählen Sie **SQL-Datenbanken** aus. Wählen Sie den aufgelisteten Datenbanknamen **AdventureWorksLT** aus.

    ![Screenshot zur Auswahl einer SQL-Datenbank](../images/dp-300-module-12-lab-01.png)

1. Navigieren Sie im Hauptblatt für die Datenbank **AdventureWorksLT** nach unten zum Abschnitt „Überwachung“. Wählen Sie **Warnungen** aus.

    ![Screenshot zur Auswahl von Warnungen auf der Übersichtsseite für die SQL-Datenbank](../images/dp-300-module-12-lab-02.png)

1. Wählen Sie **Warnungsregel erstellen** aus.

    ![Screenshot zur Auswahl von „Neue Warnungsregel“](../images/dp-300-module-12-lab-03.png)

1. Wählen Sie in der Folie **Signal auswählen** die Option **CPU-Prozentsatz** aus.

    ![Screenshot zur Auswahl des CPU-Prozentsatzes](../images/dp-300-module-12-lab-04.png)

1. Wählen Sie in der Folie **Signal konfigurieren** die Option **Statisch** als **Schwellenwerteigenschaft** aus. Überprüfen Sie dann, ob der **Operator** **Größer als** und der **Aggregationstyp** **Durchschnitt** ist. Geben Sie dann in **Schwellenwert** einen Wert von **80** ein. Wählen Sie **Fertig** aus.

    ![Screenshot zur Eingabe von „80“ und Auswahl von „Fertig“](../images/dp-300-module-12-lab-05.png)

1. Klicken Sie auf die Registerkarte **Actions** (Aktionen).

    ![Screenshot zur Auswahl des Links „Aktionsgruppe auswählen“](../images/dp-300-module-12-lab-06.png)

1. Wählen Sie auf der Registerkarte **Aktionen** die Option **Aktionsgruppe erstellen** aus.

    ![Screenshot zur Auswahl von „Aktionsgruppe erstellen“](../images/dp-300-module-12-lab-07.png)

1. Geben Sie auf dem Bildschirm **Aktionsgruppe** die Zeichenfolge **emailgroup** in das Feld **Name der Aktionsgruppe** ein. Wählen Sie dann **Weiter: Benachrichtigungen**.

    ![Screenshot zur Eingabe von „emailgroup“ und Auswahl von „Weiter: Benachrichtigungen“](../images/dp-300-module-12-lab-08.png)

1. Geben Sie auf der Registerkarte **Benachrichtigungen** die folgenden Informationen ein:

    - **Benachrichtigungstyp:** E-Mail/SMS-Nachricht/Pushnachricht/Sprachnachricht
        - **Hinweis:** Wenn Sie diese Option auswählen, wird ein Flyout für E-Mail/SMS-Nachricht/Pushnachricht/Sprachnachricht angezeigt. Überprüfen Sie die E-Mail-Eigenschaft, und geben Sie den Azure-Benutzernamen ein, mit dem Sie sich angemeldet haben.
    - **Name:** DemoLab

    ![Screenshot zur Seite „Aktionsgruppe erstellen“ mit hinzugefügten Informationen](../images/dp-300-module-12-lab-09.png)

1. Klicken Sie auf **Überprüfen + erstellen** und dann auf **Erstellen**.

    ![Screenshot zur Seite „Warnungsregel erstellen“ mit Auswahl von „Warnungsregel erstellen“](../images/dp-300-module-12-lab-10.png)

    **Hinweis:** Bevor Sie **Erstellen** auswählen, können Sie auch **Aktionsgruppe testen (Vorschau)** auswählen, um die Warnung zu testen.

1. Eine solche E-Mail wird an die von Ihnen eingegebene E-Mail-Adresse gesendet, sobald die Regel erstellt ist.

    ![Screenshot zur Bestätigung per E-Mail](../images/dp-300-module-12-lab-11.png)

    Wenn die CPU-Auslastung im Durchschnitt 80 % überschreitet, wird eine E-Mail wie diese gesendet.

    ![Screenshot zur Warnung per E-Mail](../images/dp-300-module-12-lab-12.png)

Sie können Warnungen so konfigurieren, dass Sie bei Erreichen eines Schwellenwerts für eine bestimmte Metrik (etwa Datenbankgröße oder CPU-Auslastung) per E-Mail oder Webhook benachrichtigt werden. Sie haben gerade gelernt, wie einfach es ist, Warnungen für Azure SQL-Datenbanken zu konfigurieren.
