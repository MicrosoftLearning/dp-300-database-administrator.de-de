---
lab:
  title: 'Lab 14: Konfigurieren der Georeplikation für Azure SQL-Datenbank'
  module: Plan and implement a high availability and disaster recovery solution
---

# Konfigurieren der Georeplikation für Azure SQL-Datenbank

**Geschätzte Dauer**: 30 Minuten

Als DBA in AdventureWorks müssen Sie die Georeplikation für Azure SQL Database aktivieren und sicherstellen, dass sie ordnungsgemäß funktioniert. Darüber hinaus verlagern Sie die Datenbank in einem Failover manuell über das Portal in eine andere Region.

## Aktivieren der Georeplikation

1. Starten Sie auf dem virtuellen Lab-Computer eine Browsersitzung, und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com/). Stellen Sie eine Verbindung zum Portal her. Verwenden Sie dafür **Benutzernamen** und **Kennwort** von Azure, die auf der Registerkarte **Ressourcen** für diesen virtuellen Lab-Computer bereitgestellt werden.

    ![Screenshot zur Anmeldeseite des Azure-Portals](../images/dp-300-module-01-lab-01.png)

1. Navigieren Sie im Azure-Portal zurück zu Ihrer Datenbank, indem Sie nach **SQL-Datenbanken** suchen.

    ![Screenshot zur Suche nach bestehenden SQL-Datenbanken](../images/dp-300-module-13-lab-03.png)

1. Wählen Sie die SQL-Datenbank **AdventureWorksLT** aus.

    ![Screenshot mit Auswahl der AdventureWorks-SQL-Datenbank](../images/dp-300-module-13-lab-04.png)

1. Wählen Sie auf dem Blatt für die Datenbank im Bereich **Datenverwaltung** die Option **Replikate** aus.

    ![Screenshot zur Auswahl der Georeplikation.](../images/dp-300-module-14-lab-01.png)

1. Wählen Sie **+ Replikat erstellen** aus.

    ![Screenshot der ausgewählten Georeplikation.](../images/dp-300-module-14-lab-02.png)

1. Wählen Sie auf der Seite **SQL-Datenbank erstellen – Georeplikat** unter **Server** den Link **Neu erstellen** aus.

    ![Screenshot zeigt den Link „Neuen Server erstellen“.](../images/dp-300-module-14-lab-03.png)

    >[!NOTE]
    > Wenn Sie einen neuen Server zum Hosten der sekundären Datenbank erstellen, können Sie die obige Fehlermeldung ignorieren.

1. Geben Sie auf der Seite **SQL-Datenbank-Server erstellen** einen eindeutigen **Servernamen** Ihrer Wahl, eine gültige **Serveradministratoranmeldung** und ein sicheres **Kennwort** ein. Wählen Sie einen **Speicherort** als Zielbereich und dann **OK** aus, um den Server zu erstellen.

    ![Screenshot: Seite „SQL-Datenbank-Server erstellen“](../images/dp-300-module-14-lab-04.png)

1. Wählen Sie zurück auf der Seite **SQL-Datenbank erstellen – Georeplikat** die Option **Bewerten + Erstellen** aus.

    ![Screenshot: Seite „SQL-Datenbank-Server erstellen“](../images/dp-300-module-14-lab-05.png)

1. Wählen Sie **Erstellen** aus.

    ![Screenshot zeigt die Seite „Bewerten + Erstellen“.](../images/dp-300-module-14-lab-06.png)

1. Der sekundäre Server und die Datenbank werden nun erstellt. Sie können den Status über das Benachrichtigungssymbol im oberen Bereich des Portals überprüfen. 

    ![Screenshot zeigt die Seite „Bewerten + Erstellen“.](../images/dp-300-module-14-lab-07.png)

1. Im Erfolgsfall wechselt es von **Bereitstellung in Bearbeitung** zu **Bereitstellung erfolgreich**.

    ![Screenshot zeigt die Seite „Bewerten + Erstellen“.](../images/dp-300-module-14-lab-08.png)

## Failover der SQL-Datenbank in eine sekundäre Region

Nachdem das Azure SQL-Datenbank-Replikat erstellt wurde, führen Sie ein Failover aus.

1. Navigieren Sie zur Seite „SQL-Server“, und beachten Sie den neuen Server in der Liste. Wählen Sie den sekundären Server aus (der Servername kann bei Ihnen abweichen).

    ![Screenshot der Seite „SQL-Server“.](../images/dp-300-module-14-lab-09.png)

1. Klicken Sie auf dem Blatt „SQL Server“ im Abschnitt **Einstellungen** auf **SQL-Datenbanken**.

    ![Screenshot mit der Option für SQL-Datenbanken.](../images/dp-300-module-14-lab-10.png)

1. Wählen Sie auf dem Hauptblatt für die SQL-Datenbank unter **Datenverwaltung** die Option **Replikate** aus.

    ![Screenshot zur Auswahl der Georeplikation.](../images/dp-300-module-14-lab-01.png)

1. Beachten Sie, dass die Verbindung zur Georeplikation nun hergestellt ist.

    ![Screenshot mit der Option „Replikate“](../images/dp-300-module-14-lab-11.png)

1. Wählen Sie das Menü **...** für den sekundären Server aus, und wählen Sie **Erzwungenes Failover**.

    ![Screenshot zeigt die Option für erzwungenes Failover](../images/dp-300-module-14-lab-12.png)

    > [!NOTE]
    > Beim erzwungenen Failover wird die sekundäre Datenbank in die primäre Rolle versetzt. Alle Sitzungen werden während dieses Vorgangs getrennt.

1. Wenn Sie in einer Warnmeldung dazu aufgefordert werden, klicken Sie auf **Ja**.

    ![Screenshot mit einer Warnmeldung zum erzwungenen Failover.](../images/dp-300-module-14-lab-13.png)

1. Der Status des primären Replikats ändert sich in **Ausstehend** und der des sekundären zu **Failover**. 

    ![Screenshot mit einer Warnmeldung zum erzwungenen Failover.](../images/dp-300-module-14-lab-14.png)

    > [!NOTE]
    > Dieser Vorgang kann einige Minuten dauern. Wenn er abgeschlossen ist, werden die Rollen getauscht, wobei das sekundäre Replikat zum neuen primären wird und das alte primäre zum sekundären.

Die lesbare sekundäre Datenbank kann sich in derselben Azure-Region befinden wie die primäre, oder, was häufiger der Fall ist, in einer anderen Region. Diese Art von lesbaren Sekundärdatenbanken werden auch als Geo-Sekundärdatenbanken oder Geo-Replikate bezeichnet.

Sie haben nun gelernt, wie Sie Geo-Replikate für Azure SQL Database aktivieren und sie per manuellem Failover über das Portal in eine andere Region verlagern können.
