---
lab:
  title: 'Lab 5: Aktivieren von Microsoft Defender für SQL und Datenklassifizierung'
  module: Implement a Secure Environment for a Database Service
---

# Aktivieren von Microsoft Defender für SQL und Datenklassifizierung

**Geschätzte Dauer: 30 Minuten**

Die Kursteilnehmer nutzen die in den Lektionen erworbenen Informationen, um die Sicherheit im Azure-Portal und in der AdventureWorks-Datenbank zu konfigurieren und anschließend zu implementieren.

Sie wurden als verantwortlicher Datenbankadministrator eingestellt, um die Sicherheit der Datenbankumgebung zu gewährleisten. Für diese Aufgaben wird schwerpunktmäßig Azure SQL-Datenbank eingesetzt.

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

## Aktivieren von Microsoft Defender für SQL

1. Starten Sie auf dem virtuellen Lab-Computer oder Ihrem lokalen Computer (falls kein virtueller Computer bereitgestellt wurde) eine Browsersitzung und navigieren Sie zu [https://portal.azure.com](https://portal.azure.com/). Melden Sie sich mit Ihren Azure-Anmeldedaten beim Portal an.

1. Suchen Sie im Azure-Portal im oberen Suchfeld nach *SQL-Server* und wählen Sie dann **SQL-Server** aus der Liste der Optionen aus.

1. Wählen Sie den SQL-Server **dp300-lab-xxxxxxxx** aus, wobei *xxxxxxxx* eine zufällige numerische Zeichenfolge ist.

    > &#128221; Wenn Sie Ihren eigenen Azure SQL-Server verwenden, der nicht in diesem Lab erstellt wurde, wählen Sie den Namen dieses SQL-Servers aus.

1. Wählen Sie im Bereich *Übersicht* die Option **Nicht konfiguriert** neben *Microsoft Defender for SQL*.

1. Wählen Sie das **X** oben rechts aus, um das Übersichtsfenster *Microsoft Defender for Cloud* zu schließen.

1. Wählen Sie **Aktivieren** unter *Microsoft Defender for SQL* aus.

1. In einer Produktionsumgebung sollten mehrere Empfehlungen aufgeführt sein. Wählen Sie **Alle Empfehlungen in Defender for Cloud anzeigen** und prüfen Sie alle *Microsoft Defender*-Empfehlungen, die für Ihren Azure SQL Server aufgeführt sind, und implementieren Sie sie entsprechend.

## Sicherheitsrisikobewertung

1. Navigieren Sie in der Hauptansicht Ihres Azure SQL-Servers zum Abschnitt **Einstellungen** und wählen Sie **SQL-Datenbanken** und dann die Datenbank mit dem Namen **AdventureWorksLT** aus.

1. Wählen Sie die Einstellung **Microsoft Defender for Cloud** unter **Sicherheit**.

1. Wählen Sie das **X** oben rechts, um den Übersichtsbereich für *Microsoft Defender for Cloud* zu schließen und das **Microsoft Defender for Cloud**-Dashboard für Ihre `AdventureWorksLT`-Datenbank anzuzeigen.

1. Um mit der Überprüfung der Funktionen für die Sicherheitsrisikobewertung zu beginnen, wählen Sie unter **Vulnerability assessment findings** (Ergebnisse der Sicherheitsrisikobewertung) die Option **View additional findings in Vulnerability Assessment** (Zusätzliche Ergebnisse in der Sicherheitsrisikobewertung anzeigen) aus.  

1. Wählen Sie **Überprüfen** aus, um die aktuellsten Ergebnisse der Sicherheitsrisikobewertung zu erhalten. Dieser Vorgang dauert einige Augenblicke, während die Schwachstellenanalyse die Datenbank überprüft.

1. Jedes Sicherheitsrisiko weist eine Risikostufe auf (hoch, mittel oder niedrig) und zusätzliche Informationen auf. Die geltenden Regeln basieren auf Benchmarks, die vom [Center for Internet Security](https://www.cisecurity.org/benchmark/microsoft_sql_server/?azure-portal=true) zur Verfügung gestellt werden. Wählen Sie auf der Registerkarte **Ergebnisse** ein Sicherheitsrisiko aus. Notieren Sie sich die **ID** des Sicherheitsrisikos, z.B. **VA1143** (falls aufgeführt).

1. Abhängig von der Sicherheitsüberprüfung werden alternative Ansichten und Empfehlungen angezeigt. Gehen Sie die bereitgestellten Informationen durch. Für diese Sicherheitsüberprüfung können Sie die Schaltfläche **Alle Ergebnisse als Baseline hinzufügen** und dann **Ja** auswählen, um die Baseline festzulegen. Da jetzt eine Baseline vorhanden ist, gibt diese Sicherheitsüberprüfung bei zukünftigen Scans einen Fehler aus, bei denen sich die Ergebnisse von der Baseline unterscheiden. Wählen Sie oben rechts **X** aus, um den Bereich für die bestimmte Regel zu schließen.  

1. Lassen Sie den **Scan** erneut laufen und bestätigen Sie, dass die ausgewählte Sicherheitsprüfung nun als *Bestanden* angezeigt wird.

    Wenn Sie die vorherige bestandene Sicherheitsprüfung auswählen, sollte die von Ihnen konfigurierte Baseline angezeigt werden. Wenn zukünftig eine Änderung auftritt, wird sie von den Überprüfungen der Sicherheitsrisikobewertung übernommen, und bei der Sicherheitsprüfung tritt ein Fehler auf.  

## Advanced Threat Protection

1. Klicken Sie rechts oben auf **X**, um den Bereich „Sicherheitsrisikobewertung“ zu schließen und zum Dashboard **Microsoft Defender for Cloud** für Ihre Datenbank zurückzukehren. Unter **Sicherheitsincidents und Warnungen** sollten keine Elemente angezeigt werden. Dies bedeutet, dass **Advanced Threat Protection** keine Probleme erkannt hat. Advanced Threat Protection erkennt Anomalien bei Aktivitäten, die auf ungewöhnliche und potenziell schädliche Versuche hinweisen, auf Datenbanken zuzugreifen oder diese zu nutzen.  

    > &#128221; In dieser Phase werden Sie wahrscheinlich keine Sicherheitswarnungen sehen. Im nächsten Schritt führen Sie einen Test aus, mit dem eine Warnung ausgelöst wird, damit Sie die Ergebnisse in Advanced Threat Protection überprüfen können.  

    Sie können Advanced Threat Protection verwenden, um Bedrohungen zu identifizieren und Sie zu warnen, wenn der Verdacht besteht, dass eines der folgenden Probleme auftritt:  

    - Einschleusung von SQL-Befehlen
    - Sicherheitsrisiko durch Einschleusung von SQL-Befehlen
    - Datenexfiltration
    - Unsichere Aktion
    - Brute-Force
    - Ungewöhnliche Clientanmeldung

    In diesem Abschnitt erfahren Sie, wie eine Warnung zu einer Einschleusung von SQL-Befehlen durch SSMS ausgelöst werden kann. Warnungen zur Einschleusung von SQL-Befehlen sind für benutzerdefinierte Anwendungen gedacht und nicht für Standardtools wie SSMS. Wenn Sie daher eine Warnung über SSMS als Test für eine Einschleusung von SQL-Befehlen auslösen möchten, müssen Sie den **Anwendungsnamen** „festlegen“, bei dem es sich um eine Verbindungseigenschaft für Clients handelt, die eine Verbindung mit SQL Server oder Azure SQL herstellen.

1. Öffnen Sie SQL Server Management Studio (SSMS) auf dem virtuellen Lab-Computer oder auf Ihrem lokalen Computer, falls kein virtueller Computer bereitgestellt wurde. Fügen Sie im Dialogfeld „Mit Server verbinden“ den Namen Ihres Azure SQL-Datenbankservers ein und melden Sie sich mit den folgenden Anmeldeinformationen an:

    - **Servername:** &lt;_Namen des Azure SQL-Datenbankservers hier einfügen_&gt;
    - **Authentifizierung:** SQL Server-Authentifizierung
    - **Server-Admin-Anmeldung:** Ihre Azure SQL-Datenbank-Server-Admin-Anmeldung
    - **Kennwort:** Ihr Azure SQL-Datenbank-Serveradminkennwort

1. Wählen Sie **Verbinden**.

1. Wählen Sie in SSMS die Option **Datei** > **Neu** > **Datenbank-Engine-Abfrage** aus, um eine Abfrage mithilfe einer neuen Verbindung zu erstellen.  

1. Melden Sie sich im Hauptanmeldefenster wie gewohnt mit SQL-Authentifizierung und Ihrem Azure SQL Server-Namen und den Admin-Anmeldeinformationen bei der Datenbank **AdventureWorksLT** an. Bevor Sie eine Verbindung herstellen, wählen Sie **Optionen** > **Verbindungseigenschaften** aus. Geben Sie **AdventureWorksLT** für die Option **Mit der Datenbank verbinden** ein.  

1. Wählen Sie die Registerkarte **Zusätzliche Verbindungsparameter** aus, und fügen Sie dann die folgende Verbindungszeichenfolge in das Textfeld ein:  

    ```sql
    Application Name=webappname
    ```

1. Wählen Sie **Verbinden**.  

1. Fügen Sie die folgende Abfrage in das neue Abfragefenster ein, und wählen Sie dann **Ausführen** aus:  

    ```sql
    SELECT * FROM sys.databases WHERE database_id like '' or 1 = 1 --' and family = 'test1';
    ```

1. Gehen Sie im Azure-Portal zu Ihrer **AdventureWorksLT**-Datenbank. Wählen Sie Bereich auf der linken Seite unter **Sicherheit** die Option **Microsoft Defender für Cloud** aus.

1. Wählen Sie unter **Sicherheitsvorfälle und Warnungen** die Option **Warnmeldungen für diese Ressourcen in Microsoft Defender for Cloud prüfen**.  

1. Jetzt können Sie die allgemeinen Sicherheitswarnungen sehen.  

1. Wählen Sie **Potenzieller Angriff durch Einschleusung von SQL-Befehlen** aus, um spezifischere Warnungen anzuzeigen und Untersuchungsschritte zu erhalten.

1. Wählen Sie **Vollständige Details anzeigen**, um die Details der Meldung anzuzeigen.

1. Beachten Sie, dass auf der Registerkarte **Warnungsdetails** die Anweisung *Angreifbar* angezeigt wird. Dies ist die SQL-Anweisung, die ausgeführt wurde, um die Warnung auszulösen. Dies war auch die SQL-Anweisung, die in SSMS ausgeführt wurde. Beachten Sie außerdem, dass die **Client-Anwendung** als **Webanwendung** angezeigt wird. Dies ist der Name, den Sie in der Zeichenfolge für die Verbindung in SSMS angegeben haben.

1. Als Bereinigungsschritt sollten Sie in Erwägung ziehen, alle Abfrage-Editoren in SSMS zu schließen und alle Verbindungen zu entfernen, sodass Sie in den nächsten Übungen nicht versehentlich zusätzliche Warnungen auslösen.

## Aktivieren der Datenklassifizierung

1. Navigieren Sie in der Hauptansicht Ihres Azure SQL-Servers zum Abschnitt **Einstellungen** und wählen Sie **SQL-Datenbanken** und dann die Datenbank mit dem Namen **AdventureWorksLT** aus.

1. Navigieren Sie auf dem Hauptblatt für die **AdventureWorksLT**-Datenbank zum Abschnitt **Sicherheit**, und wählen Sie **Datenermittlung und -klassifizierung** (Datenermittlung und -klassifizierung) aus.

1. Auf der Seite **Datenermittlung und -klassifizierung** wird eine Informationsmeldung mit folgendem Wortlaut angezeigt: **Derzeit wird die SQL Information Protection-Richtlinie verwendet. Wir haben 15 Spalten mit Klassifizierungsempfehlungen gefunden**. Wählen Sie diesen Link aus.

1. Aktivieren Sie auf dem nächsten Bildschirm von **Datenermittlung und -klassifizierung** das Kontrollkästchen neben **Alle auswählen**, wählen Sie **Ausgewählte Empfehlungen annehmen** aus, und klicken Sie auf **Speichern**, um die Klassifizierungen in der Datenbank zu speichern.

1. Zurück im Bildschirm **Datenermittlung und -klassifizierung** stellen Sie fest, dass fünfzehn Spalten in fünf verschiedenen Tabellen erfolgreich klassifiziert wurden. Überprüfen Sie die *Informationsart* und *Vertraulichkeitsbezeichnung* für jede der Spalten.

## Konfigurieren der Datenklassifizierung und -maskierung

1. Gehen Sie im Azure-Portal zu Ihrer Azure SQL-Datenbank-Instanz **AdventureWorksLT** (nicht Ihr logischer Server).

1. Wählen Sie im linken Bereich unter **Sicherheit** die Option **Datenermittlung und -klassifizierung** aus.  

1. In der SalesLT-Kundentabelle hat *Datenerkennung und Klassifizierung* `FirstName` und `LastName` als zu klassifizierend identifiziert, aber nicht `MiddleName`. Verwenden Sie die Dropdownlisten, um sie jetzt hinzuzufügen. Wählen Sie **Name** für den *Informationstyp* und **Vertraulich – DSGVO** für die *Vertraulichkeitsbezeichnung* und wählen Sie dann **Klassifizierung hinzufügen**.  

1. Wählen Sie **Speichern**.

1. Bestätigen Sie, dass die Klassifizierung erfolgreich hinzugefügt wurde, indem Sie die Registerkarte **Übersicht** anzeigen, und bestätigen Sie, dass `MiddleName` jetzt in der Liste der klassifizierten Spalten unter dem Schema SalesLT angezeigt wird.

1. Wählen Sie im linken Bereich **Übersicht** aus, um zur Übersicht Ihrer Datenbank zurückzukehren.  

   Die dynamische Datenmaskierung (DDM) ist sowohl in Azure SQL als auch in SQL Server verfügbar. DDM verhindert die Offenlegung vertraulicher Daten durch Maskierung vertraulicher Daten für nicht berechtigte Benutzende auf SQL Server-Ebene anstatt auf Anwendungsebene, wo Sie diese Art von Regeln codieren müssen. Azure SQL zeigt Empfehlungen an, welche Daten maskiert werden sollten. Sie können Masken jedoch auch manuell hinzufügen.

   In den nächsten Schritten maskieren Sie die Spalten `FirstName`, `MiddleName` und `LastName`, die Sie im vorherigen Schritt überprüft haben.  

1. Navigieren Sie im Azure-Portal zu Ihrer Azure SQL-Datenbank-Instanz. Wählen Sie im linken Bereich unter **Sicherheit** die Option **Dynamische Datenmaskierung** und dann **Maske hinzufügen** aus.  

1. Wählen Sie in den Dropdownlisten zuerst das Schema **SalesLT**, die Tabelle **Customer** und die Spalte **FirstName** aus. Sie können die Maskierungsoptionen überprüfen. Die Standardeinstellung ist für dieses Szenario jedoch geeignet. Klicken Sie auf **Hinzufügen**, um die Maskierungsregel hinzuzufügen.  

1. Wiederholen Sie die vorherigen Schritte für **MiddleName** und **LastName** in dieser Tabelle.  

    Sie haben jetzt drei Maskierungsregeln.  

1. Wählen Sie **Speichern**.

    > &#128221; Hinweis: Wenn Ihr Azure SQL Server-Name nicht aus nur Kleinbuchstaben, Zahlen und Gedankenstrichen besteht, schlägt dieser Schritt fehl, und Sie können nicht mit den Datenmaskierungsabschnitten fortfahren.

1. Wählen Sie im linken Bereich **Übersicht** aus, um zur Übersicht Ihrer Datenbank zurückzukehren.

## Abrufen klassifizierter und maskierter Daten

Anschließend simulieren Sie eine Abfrage der klassifizierten Spalten und lernen die dynamische Datenmaskierung in Aktion kennen.

1. Gehen Sie zu SQL Server Management Studio (SSMS), verbinden Sie sich mit Ihrem Azure SQL Server und öffnen Sie ein neues Abfragefenster.

1. Klicken Sie mit der rechten Maustaste auf die Datenbank **AdventureWorksLT** und wählen Sie **Neue Abfrage**.  

1. Führen Sie die folgende Abfrage aus, um die klassifizierten Daten und in einigen Fällen die für maskierte Daten gekennzeichneten Spalten zurückzugeben. Wählen Sie **Ausführen** aus, um die Abfrage durchzuführen.

    ```sql
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    ```

    Ihr Ergebnis sollte die ersten 10 Namen ohne Maskierung anzeigen. Warum? Da Sie der Administrator für diesen logischen Azure SQL-Datenbank-Server sind.  

1. In der folgenden Abfrage erstellen Sie einen neuen Benutzer und führen die vorherige Abfrage als dieser Benutzer aus. Außerdem verwenden Sie `EXECUTE AS`, um die Identität von `Bob` anzunehmen. Bei Ausführung der Anweisung `EXECUTE AS` wird der Ausführungskontext der Sitzung auf den Anmeldenamen bzw. Benutzer umgestellt. Dies bedeutet, dass die Berechtigungen für den Anmeldenamen bzw. Benutzer statt für die Person überprüft werden, die den Befehl `EXECUTE AS` ausführt (in diesem Fall Sie). `REVERT` wird dann verwendet, damit die Identität des Anmeldenamens bzw. des Benutzers nicht mehr angenommen wird.  

    Sie werden möglicherweise die ersten Teile der folgenden Befehle erkennen, da sie eine Wiederholung aus einer vorherigen Übung sind. Erstellen Sie eine neue Abfrage mit den folgenden Befehlen, und wählen Sie dann **Ausführen** aus, um die Abfrage auszuführen und die Ergebnisse anzuzeigen.

    ```sql
    -- Create a new SQL user and give them a password
    CREATE USER Bob WITH PASSWORD = 'c0mpl3xPassword!';

    -- Until you run the following two lines, Bob has no access to read or write data
    ALTER ROLE db_datareader ADD MEMBER Bob;
    ALTER ROLE db_datawriter ADD MEMBER Bob;

    -- Execute as our new, low-privilege user, Bob
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;
    ```

    Das Ergebnis sollte jetzt die ersten 10 Namen anzeigen, allerdings mit Maskierung. Bob wurde kein Zugriff auf die unmaskierte Form dieser Daten gewährt.  

    Was geschieht, wenn Bob aus irgendeinem Grund Zugriff auf die Namen benötigt und die Berechtigung erhält, diese abzurufen?  

    Sie können ausgeschlossene Benutzer von der Maskierung im Azure-Portal aktualisieren, indem Sie zum Bereich **Dynamische Datenmaskierung** unter **Sicherheit** wechseln, dies ist aber ebenso mit T-SQL möglich.

1. Klicken Sie mit der rechten Maustaste auf die **AdventureWorksLT**-Datenbank, und wählen Sie **Neue Abfrage** aus, und geben Sie dann die folgende Abfrage ein, damit Bob die Namensergebnisse ohne Maskierung abfragen kann. Wählen Sie **Ausführen** aus, um die Abfrage durchzuführen.

    ```sql
    GRANT UNMASK TO Bob;  
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;  
    ```

    Die Ergebnisse sollten die vollständigen Namen enthalten.  

1. Sie können einem Benutzer auch die Berechtigungen zur Aufhebung der Maskierung entziehen und diese Aktion bestätigen, indem Sie die folgenden T-SQL-Befehle in einer Abfrage ausführen:  

    ```sql
    -- Remove unmasking privilege
    REVOKE UNMASK TO Bob;  

    -- Execute as Bob
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;  
    ```

    Die Ergebnisse sollten die maskierten Namen enthalten.  

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

In dieser Übung haben Sie die Sicherheit einer Azure SQL-Datenbank durch die Aktivierung von Microsoft Defender for SQL verbessert. Außerdem haben Sie auf der Grundlage von Empfehlungen des Azure-Portals klassifizierte Spalten erstellt.
