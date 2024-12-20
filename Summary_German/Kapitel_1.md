---
title: "SQL Kapitel 1"
author: "db"
---

# Datenquellen

Es gibt zwei Haupttypen von Daten: **strukturierte und unstrukturierte Daten**. 

*   **Unstrukturierte Daten** liegen in Form von Textdokumenten oder Bildern als einzelne Dateien vor.
*   **Strukturierte Daten** liegen in einem tabellarischen Format vor, wie z. B. Tabellenkalkulationen oder Datenbanktabellen.
*   Weitere Arten von Daten umfassen CSV, JSON, XML, Graphdatenbanken und Schlüsselwertspeicher.

**Beispiele für RDBMS (Relational Database Management System)** sind Oracle, MySQL, MS SQL Server, PostgreSQL, Amazon Redshift, IBM DB2, MS Access, SQLite und Snowflake.

*   Obwohl sich die Syntax geringfügig unterscheidet, sind die SQL-Konzepte konsistent.

# Relationale Datenbanken

Eine Datenbanktabelle ähnelt einer Tabellenkalkulation.

*   Sie verfügt über Zeilenbezeichner und benannte Spaltenüberschriften.

**Eine Entität** ist ein Objekt oder Konzept, das die Tabelle repräsentiert.

*   Betrachten Sie zum Beispiel eine Tabelle mit Büchern. Die Entität ist Bücher.
*   Die Tabelle enthält Informationen wie ISBN-Nummer, Titel und Autor.
*   Manche Leute verwenden die Begriffe Entität und Tabelle synonym.

Eine Zeile wird auch als Datensatz oder Tupel bezeichnet. Eine Spaltenüberschrift wird auch als Feld oder Attribut bezeichnet.

*   In einer Tabelle mit Büchern ist der Wert im Feld Autor für das Buch "SQL für Data Scientists" Renée M. P. Teate.

**Eine Datenbank ist eine Sammlung von verwandten Tabellen**.

*   Ein Datenbankschema speichert Informationen über die Tabellen und deren Beziehungen.

**Beispiel**: Datenbank einer Arztpraxis:

*   Eine Tabelle enthält Patienteninformationen: Name, Geburtsdatum, Telefonnummer.
*   Eine weitere Tabelle enthält Termininformationen: Patientenname, Terminzeit, Arztname.
*   Die Verbindung zwischen diesen Tabellen könnte der Name des Patienten sein.
*   In der Regel wird ein eindeutiger Identifikator verwendet, da zwei Personen denselben Namen haben können.

Die Beziehung zwischen diesen beiden Tabellen wird als **Eins-zu-Viele-Beziehung** bezeichnet.

*   Jeder Patient erscheint einmal in der Patient Tabelle.
*   Jeder Patient kann in der Termintabelle mehrmals erscheinen.
*   Diese Beziehungen werden in einem Entity-Relationship-Diagramm (ERD) dargestellt.

Ein Unendlichkeitszeichen, N oder Krähenfüße zeigen die "Viele"-Seite einer Eins-zu-Viele-Beziehung an.

**Der Primärschlüssel identifiziert eine Zeile eindeutig**.

*   Der Primärschlüssel darf nicht NULL sein.
*   NULL bedeutet kein Wert, anders als ein Leerzeichen.
*   Der Primärschlüssel kann ein eindeutiger Wert sein, wie z. B. eine Studenten-ID, oder von der Datenbank generiert werden.
*   Der Primärschlüssel wird in einer anderen Tabelle als Fremdschlüssel referenziert.

**Beispiel**: Datenbank einer Arztpraxis:

*   Jeder Patientendatensatz in der Patient Tabelle hat einen eindeutigen Primärschlüssel.
*   Jeder Termindatensatz in der Termintabelle hat einen eindeutigen Primärschlüssel.
*   Die Termintabelle verwendet die Patienten-ID, um Termine mit Patienten zu verknüpfen.
*   Der Name des Patienten wird nicht in der Termintabelle gespeichert.

Eine weitere Art von Beziehung in RDBMSs ist die **Viele-zu-Viele-Beziehung**.

*   Sie verbindet Entitäten, bei denen Datensätze auf jeder Seite mit mehreren Datensätzen auf der anderen Seite verknüpft werden können.
*   **Beispiel**: Bücher und Autoren haben eine Viele-zu-Viele-Beziehung.
    *   Jeder Autor kann mehrere Bücher schreiben.
    *   Jedes Buch kann mehrere Autoren haben.
*   Es wird eine **Junction-Tabelle** benötigt, um die Paare verwandter Zeilen zu erfassen.

**In Abbildung 1.5**:

*   ISBN ist der Primärschlüssel in der Tabelle Bücher.
*   Autoren-ID ist der Primärschlüssel in der Tabelle Autoren.
*   Beide sind Fremdschlüssel in der Junction-Tabelle Bücher-Autoren.
*   Jedes Paar aus ISBN und Autoren-ID ist eindeutig.
*   Das Paar ist ein mehrspaltiger Primärschlüssel in der Junction-Tabelle Bücher-Autoren.
*   Diese Einrichtung vermeidet mehrere Zeilen pro Buch oder Autor in der Tabelle Bücher.
*   Sie reduziert redundante Daten und verdeutlicht Beziehungen.

**Datenbanknormalisierung** bedeutet, dass keine redundanten Daten gespeichert werden.

*   In der Buchdatenbank wird der Name jedes Autors einmal gespeichert.
*   In der Datenbank der Arztpraxis wird die Telefonnummer eines Patienten einmal im Patientenverzeichnis gespeichert.
*   Die Normalisierung reduziert den Speicherplatz und vereinfacht Aktualisierungen.
*   Weitere Informationen finden Sie unter Relationales Datenbankdesign.

# Dimensionale Data Warehouses

**Data Warehouses** enthalten Daten aus mehreren Quellen.

*   Sie können in Form einer normalisierten relationalen Datenbank oder in anderen Designs entworfen werden.
*   Sie können Rohdaten und Summierungstabellen enthalten.
*   Summierungstabellen sind transformierte Versionen von Rohdaten.
*   Data Warehouses können historische Datenprotokolle, in Echtzeit aktualisierte Tabellen oder Daten-Snapshots speichern.

**Dimensionale Modellierungstechniken** werden häufig in Data Warehouses verwendet.

*   Ein gängiges Design ist das **Sternschema**.
*   Es teilt Daten in Fakten und Dimensionen auf.

Eine **Faktentabelle** enthält die Metadaten einer Entität und numerische Kennzahlen, die verfolgt werden sollen.

*   **Beispiel**: Ein Kaufdatensatz enthält einen Zeitstempel, die Filialnummer, die Bestellnummer, die Kundennummer und den gezahlten Betrag.

Eine **Dimension** ist eine Eigenschaft einer Entität, nach der Faktdatensätze gruppiert werden können.

*   **Beispiel**: Das Geschäft in einem Kaufdatensatz ist eine Dimension.
*   Die Dimensionstabelle des Geschäfts enthält Informationen wie den Namen des Geschäfts.

**Beispiel**: Datenbank einer Arztpraxis in einem Sternschema.

*   Die Faktentabelle Termine erfasst die Details jedes Termins.
*   Die Datumsdimension und die Zeitdimension speichern Eigenschaften von Termin-Datumsangaben und -Zeiten.

Dieses Design ermöglicht eine einfache Gruppierung und Zusammenfassung von Daten.

*   **Beispiel**: Zählen von Terminen pro Zeitraum oder Finden von Spitzenzeiten für die Terminbuchung.

**Abbildung 1.6** zeigt ein dimensionales Data Warehouse-Design.

*   Das Design ähnelt einem Stern, daher der Name Sternschema.

Es könnte ein Terminverlaufsprotokoll im Data Warehouse geben.

*   Es zeichnet jedes Mal auf, wenn ein Termin geändert wurde.
*   Sie können sehen, wann der Termin stattfinden soll.
*   Sie können sehen, wie oft er geändert wurde.
*   Sie können sehen, ob er ursprünglich einem anderen Arzt zugewiesen war.

Ein dimensionales Modell speichert mehr Informationen als eine normalisierte relationale Datenbank.

*   Termindatensätze erscheinen mehrmals in einer Terminprotokolltabelle.
*   Die Datumsdimensionstabelle kann einen Datensatz für jedes Kalenderdatum enthalten.
*   Die Datumsangaben können sich über Jahrzehnte in die Zukunft erstrecken.

Um eine Datenbank oder ein Data Warehouse zu entwerfen, müssen Sie diese Konzepte im Detail verstehen.

Um die Datenbank nach einem analytischen Datensatz abzufragen:

*   Verstehen Sie die Tabellenkörnung (Detaillierungsgrad).
*   Wissen Sie, welche Spaltengruppe eine Zeile eindeutig macht.
*   Verstehen Sie, wie die Tabellen miteinander verbunden sind.

Die Abfrage eines dimensionalen Data Warehouse mit SQL ähnelt der Abfrage einer relationalen Datenbank mit SQL.

# Fragen zur Datenquelle stellen

Verstehen Sie die Datenquelle, das Schemadesign und die Tabellenbeziehungen, bevor Sie SQL schreiben.

Kommunizieren Sie mit Fachexperten (SMEs), um Einblicke zu gewinnen.

*   Zu den SMEs gehören Datenbankadministratoren (DBAs), ETL-Ingenieure und Dateneingabepersonal.
*   Verwenden Sie Datenwörterbücher, falls vorhanden.

**Beispielfragen für SMEs**:

*   Welche Tabellen sollte ich für bestimmte Daten abfragen?
*   Welche Felder bilden den Primärschlüssel?
*   Werden Datensätze direkt importiert oder transformiert?
*   Ist diese Tabelle statisch oder wird sie regelmäßig aktualisiert? Wie häufig wird sie aktualisiert?
*   Werden die Daten automatisch erfasst oder von Personen eingegeben?

Überprüfen Sie die Werteverteilungen in Feldern.

*   Visualisieren Sie Daten mithilfe von Histogrammen während der explorativen Datenanalyse (EDA).
*   Analysieren Sie Daten nach Zeiträumen, um Veränderungen zu beobachten.

Kennen Sie den Typ der Datenbank für eine effiziente Abfrage.

*   Unterschiedliche Datenbanken haben unterschiedliche SQL-Syntaxen und Leistungsmerkmale.

# Einführung in die Farmer's Market Datenbank

Unsere Beispiel-MySQL-Datenbank ist für einen fiktiven Bauernmarkt.

*   Sie verfolgt Verkäufer, Produkte, Kunden und Verkäufe.
*   Sie enthält Daten zu Markttagen wie Datum, Uhrzeiten, Wochentag und Wetter.
*   Sie enthält Verkäuferdetails wie Standzuweisungen, Produkte und Preise.
*   Die Verkäufer verwenden vernetzte Kassen, um Artikel zu kassieren.
*   Die Kunden scannen bei jeder Transaktion ihre Treuekarten.
*   Wir haben detaillierte Protokolle der Einkäufe.
    *   Wir wissen, wer welche Artikel gekauft hat und wann genau.

Die angeforderten Informationen zu "Tools for Connecting to Data Sources and Editing SQL" sowie zu "A Note on Machine Learning Dataset Terminology" sind in den bereitgestellten Quellen nicht enthalten. 

# Übungen

1.  Was denken Sie, wird in der beschriebenen Bücher- und Autorendatenbank, die in Abbildung 1.5 dargestellt ist, passieren, wenn ein Autor seinen Namen ändert? Welche Datensätze könnten hinzugefügt oder aktualisiert werden, und welche Auswirkungen könnte dies auf die Ergebnisse zukünftiger Abfragen haben, die auf diesen Daten basieren?
2.  Denken Sie an etwas in Ihrem Leben, das Sie mit einer Datenbank verfolgen könnten.
    *   Eins-zu-Viele-Beziehungen:
        *   Beispiel: Eine Person kann mehrere E-Mail-Adressen haben.
    *   Viele-zu-Viele-Beziehungen:
        *   Beispiel: Studenten und Kurse. Ein Student kann sich in mehrere Kurse einschreiben, und ein Kurs kann mehrere Studenten haben.

Bitte beachten Sie, dass dies nur eine Zusammenfassung der wichtigsten Punkte in den Quellen ist. Es wird empfohlen, die Quellen selbst zu lesen, um ein vollständigeres Verständnis der Themen zu erhalten.
