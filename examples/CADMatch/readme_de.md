# KUKA KRL Beispiel: CADMatchObjekterkennung

## 1. Übersicht

Dieses KUKA KRL-Programm (`CadMatchExample.src`) demonstriert die CAD-basierte Objekterkennung und Pick-and-Place-Operationen mit dem rc_visard oder rc_cube von Roboception unter Verwendung des CADMatch-Softwaremoduls. Der Roboter identifiziert Objekte anhand einer vordefinierten CAD-Vorlage und versucht, sie aufzunehmen und an einem bestimmten Ort abzulegen.

Das Programm führt folgende Hauptschritte aus:
1.  Initialisiert den Roboter und die EKI (EtherNet/KRL Interface) Kommunikation.
2.  Startet den CADMatch-Dienst auf dem Roboception-Gerät.
3.  Tritt in eine Schleife ein, um kontinuierlich Objekte zu erkennen und zu verarbeiten:
    1.  Konfiguriert Erkennungsparameter (z.B. Vorlagen-ID, Pose-Referenzsystem, Region of Interest).
    2.  Löst den Objekterkennungsdienst aus.
    3.  Beinhaltet einen Wiederholungsmechanismus, falls die Erkennung fehlschlägt.
    4.  Wenn ein Objekt erfolgreich erkannt wurde, liest es die Greifinformationen und die zugehörige `gripper_id`.
    5.  Bestimmt die Zielöffnung des Greifers basierend auf der empfangenen `gripper_id`.
    6.  Zusätzlich liest es die `grasp_id` für das erkannte Objekt.
    7.  Wählt eine spezifische `PLACEPOSE` (z.B. `PLACEPOSE1`, `PLACEPOSE2`) basierend auf der `grasp_id` aus.
    8.  Führt eine Pick-and-Place-Sequenz unter Verwendung der ausgewählten `PLACEPOSE` aus:
        1.  Bewegt sich zu einer Vor-Greif-Position über dem Objekt.
        2.  Bewegt sich zur Greifposition.
        3.  Schließt den Greifer (unter Verwendung der ermittelten Öffnung).
        4.  Bewegt sich zu einer Vor-Ablage-Position.
        5.  Bewegt sich zur endgültigen Ablageposition.
        6.  Öffnet den Greifer.
    9.  Kehrt zur Startposition zurück und wiederholt den Zyklus.
4.  Stoppt den CADMatch-Dienst bei Programmbeendigung.

## 2. Voraussetzungen

*   KUKA KRC4 Robotersteuerung.
*   Roboception rc_visard oder rc_cube, ausgestattet mit dem CADMatch-Softwaremodul.
*   Korrekte EKI-Konfiguration auf der KUKA-Steuerung und dem Roboception-Gerät. Die Dienstnamen im KRL-Programm müssen mit der EKI-XML-Konfiguration übereinstimmen.
*   Netzwerkverbindung zwischen der KUKA-Steuerung und dem Roboception-Gerät hergestellt.
*   Die angegebene CAD-Vorlage (`TEMPLATE_ID`) muss im CADMatch-Modul auf dem Roboception-Gerät hochgeladen und konfiguriert sein.
*   Ein geeigneter Greifer am Roboter montiert, wobei die Funktionen `GripperOpen()` und `GripperClose()` in diesem KRL-Programm korrekt implementiert sind.

## 3. Konfigurationsparameter

Die folgenden Parameter am Anfang von `CadMatchExample.src` müssen vor dem Ausführen des Programms konfiguriert werden:

*   **EKI-Dienstnamen:**
    *   `serviceNameDetect[]`: EKI-Dienstname für die Objekterkennung (z.B. `"rc_cadmatch-detect_object"`).
    *   `serviceNameStart[]`: EKI-Dienstname zum Starten des CADMatch-Moduls (z.B. `"rc_cadmatch-start"`).
    *   `serviceNameStop[]`: EKI-Dienstname zum Stoppen des CADMatch-Moduls (z.B. `"rc_cadmatch-stop"`).
*   **Erkennungsparameter:**
    *   `TEMPLATE_ID[]`: Name der CAD-Vorlage, die abgeglichen werden soll (z.B. `"Template_Name"`). Diese muss mit der Vorlagen-ID auf dem rc_visard/rc_cube übereinstimmen.
    *   `POSE_FRAME[]`: Referenzsystem für die gemeldeten Posen (z.B. `"external"` für Posen im Basiskoordinatensystem des Roboters, unter Annahme einer korrekten Hand-Auge-Kalibrierung).
    *   `LOAD_CARRIER_ID[]`: ID eines vorkonfigurierten Ladungsträgers auf dem Roboception-Gerät. Verwenden Sie `" "` (Leerzeichen), wenn kein Ladungsträger verwendet wird.
    *   `ROI_ID[]`: ID einer vorkonfigurierten Region of Interest (ROI) auf dem Roboception-Gerät. Verwenden Sie `" "`, wenn keine spezifische ROI verwendet wird.
*   **Kollisionsüberprüfungsparameter (Optional):**
    *   `GRIPPER_ID[]`: ID eines auf dem Roboception-Gerät konfigurierten Greifermodells zur Kollisionsüberprüfung. Verwenden Sie `" "` zum Deaktivieren.
    *   `PRE_GRASP_OFFSET_X`, `PRE_GRASP_OFFSET_Y`, `PRE_GRASP_OFFSET_Z`: Offsets (in Metern) vom Greifpunkt, die vom rc_visard zur Kollisionsüberprüfung des Vor-Greif-Ansatzes verwendet werden. Typischerweise ist Z negativ. Auf `0.0` setzen, um einzelne Offsets zu deaktivieren.
*   **Bewegungsparameter (in Millimetern):**
    *   `MOT_PRE_GRASP_Z_OFFSET`: Vertikaler Offset (z.B. `-300` mm) von der Greifpose für die Vor-Greif-Anfahrtsposition des Roboters.
    *   `MOT_PRE_PLACE_Z_OFFSET`: Vertikaler Offset (z.B. `-300` mm) von der Ablagepose für die Vor-Ablage-Anfahrtsposition des Roboters.
*   **Konfiguration der Erkennungswiederholung:**
    *   `MAX_DETECTION_RETRIES`: Maximale Anzahl von Erkennungsversuchen, wenn die Erkennung fehlschlägt (z.B. `3`).
*   **Greiferkonfiguration:**
    *   `defaultOpening`: Standardöffnungswert für den Greifer (z.B. `100.0`). Dieser wird verwendet, wenn keine spezifische `gripper_id` aus den Erkennungsergebnissen übereinstimmt.

## 4. Eingelernte Positionen

Die folgenden Positionen müssen in der zugehörigen `.dat`-Datei (`CadMatchExample.dat`) eingelernt werden:

*   `STARTPOSE`: Die Start-/Heimatposition des Roboters.
*   `PLACEPOSE`: Die Standard-Zielposition, an der das Objekt abgelegt wird, wenn keine spezifische `grasp_id` übereinstimmt.
*   `PLACEPOSE1`, `PLACEPOSE2`: Alternative Zielpositionen, ausgewählt basierend auf der `grasp_id` des erkannten Objekts.

## 5. Greifersteuerung

Das Programm verwendet zwei Funktionen zur Greifersteuerung:

*   `GripperClose()`: Implementieren Sie hier die Logik zum Schließen Ihres spezifischen Greifers.
*   `GripperOpen(targetOpening : REAL)`: Implementieren Sie hier die Logik zum Öffnen Ihres spezifischen Greifers auf den Wert `targetOpening`. Dieser Wert wird basierend auf der vom CADMatch-Modul empfangenen `gripper_id` bestimmt oder verwendet standardmäßig `defaultOpening`.

Sie müssen die Beispielzeilen innerhalb dieser Funktionen (z.B. `SetOutput(...)` oder `SetAnalogOutput(...)`) an die Schnittstelle Ihres Greifers anpassen.

## 6. Funktionsweise

1.  **Initialisierung:** Der Roboter bewegt sich zu `STARTPOSE`, öffnet den Greifer auf `defaultOpening` und initialisiert EKI-Verbindungen für die CADMatch-Dienste. Die CADMatch-Anwendung auf dem Sensor wird über `serviceNameStart` gestartet.
2.  **Erkennungsschleife:**
    *   Das Programm versucht, Objekte mit `PerformDetection` zu erkennen. Diese Funktion sendet die konfigurierten Parameter (Vorlagen-ID, ROI, Kollisionsparameter, aktuelle Roboterflanschpose) an den `serviceNameDetect` EKI-Dienst.
    *   Schlägt die Erkennung fehl, versucht es bis zu `MAX_DETECTION_RETRIES` Mal erneut und zeigt bei jedem Versuch die Fehlermeldung des Bildverarbeitungssystems an.
    *   Ist die Erkennung erfolgreich, analysiert `RC_READRESULT` die Antwort. Das Programm konzentriert sich derzeit auf den ersten erkannten Greifpunkt (`DETECTEDGRASPS[1]`). Es liest die mit diesem Greifpunkt verbundene `gripper_id`.
    *   **Greifer-ID-Logik:** Basierend auf der `RECEIVED_GRIPPER_ID[]` (z.B. "gripper_a", "gripper_b") wird ein `targetOpening`-Wert gesetzt. Ist die ID unbekannt oder "none", wird `defaultOpening` verwendet.
    *   Die `grasp_id` (z.B. "grasp_1") wird ebenfalls in `RECEIVED_GRASP_ID[]` eingelesen und auf dem KCP ausgegeben.
    *   **Ablagepositions-Auswahl:** Basierend auf dem Wert von `RECEIVED_GRASP_ID[]` wird eine `selectedPlacePose` ausgewählt (z.B. `PLACEPOSE1` bei "grasp_1", `PLACEPOSE2` bei "grasp_2", ansonsten die Standard-`PLACEPOSE`).
3.  **Pick and Place (`ExecutePickAndPlace`):**
    *   **Vor-Greifen:** Der Roboter bewegt sich zu `PREGRASPPOSE`, die durch Anwenden von `MOT_PRE_GRASP_Z_OFFSET` auf die `Z`-Koordinate der erkannten `GRASPPOSE` berechnet wird.
    *   **Greifen:** Der Roboter bewegt sich linear zur `GRASPPOSE`.
    *   **Greifer schließen:** `GripperClose()` wird aufgerufen.
    *   **Anheben:** Der Roboter bewegt sich zurück zu `PREGRASPPOSE`.
    *   **Vor-Ablegen:** Der Roboter bewegt sich zu `PREPLACEPOSE`, berechnet durch Anwenden von `MOT_PRE_PLACE_Z_OFFSET` auf die `Z`-Koordinate der eingelernten `PLACEPOSE`.
    *   **Ablegen:** Der Roboter bewegt sich linear zu `PLACEPOSE`.
    *   **Vor-Ablegen:** Der Roboter bewegt sich zu `PREPLACEPOSE`, berechnet durch Anwenden von `MOT_PRE_PLACE_Z_OFFSET` auf die `Z`-Koordinate der `selectedPlacePose`.
    *   **Ablegen:** Der Roboter bewegt sich linear zur `selectedPlacePose`.
    *   **Greifer öffnen:** `GripperOpen(targetOpening)` wird aufgerufen, wobei der ermittelte Öffnungswert verwendet wird.
    *   **Zurückziehen:** Der Roboter bewegt sich zurück zu `PREPLACEPOSE`.
    *   **Heimfahrt:** Der Roboter bewegt sich zu `STARTPOSE`.
4.  **Schleifenfortsetzung:** Das Programm kehrt zum Anfang der Schleife zurück, um eine weitere Erkennung zu versuchen.
5.  **Beendigung:** Schlägt die Erkennung nach allen Wiederholungsversuchen fehl, hält das Programm an und verlässt die Schleife. Die Funktion `RC_STOP` wird aufgerufen, um den CADMatch-Dienst auf dem Roboception-Gerät zu stoppen, bevor das KRL-Programm endet.

## 7. Anpassung

*   **Mehrere Objekte:** Um mehr als ein erkanntes Objekt pro Zyklus zu verarbeiten, ändern Sie die Schleife in `DEF CadMatchExample()`, um von `1` bis `NUMDETECTEDGRASPS` zu iterieren und `ExecutePickAndPlace` für jeden gültigen Greifpunkt aufzurufen.
*   **Greiferlogik:** Passen Sie `GripperOpen()` und `GripperClose()` für Ihre spezifische Hardware an.
*   **Fehlerbehandlung:** Verbessern Sie die Fehlerbehandlung bei Bedarf für Ihre Anwendung.
*   **Bewegungsparameter:** Passen Sie Geschwindigkeiten, Beschleunigungen und Überschleifparameter in den PTP- und LIN-Bewegungsbefehlen für einen reibungsloseren und sichereren Betrieb an.
*   **Zuordnung von Greifer-ID zu Öffnung:** Ändern Sie den `IF STRCOMP(...)`-Block in der Hauptschleife, um den von Ihrem CADMatch-Setup zurückgegebenen `gripper_id`-Strings zu entsprechen und geeignete `targetOpening`-Werte festzulegen. 
*   **Zuordnung von Greif-ID zu Ablageposition:** Ändern Sie den `IF STRCOMP(...)`-Block bezüglich `RECEIVED_GRASP_ID[]`, um den von Ihrem Vision-System zurückgegebenen Greif-IDs zu entsprechen und stellen Sie sicher, dass entsprechende `PLACEPOSE1`, `PLACEPOSE2`, etc. in der `.dat`-Datei definiert und eingelernt sind. 