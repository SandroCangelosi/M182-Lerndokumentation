# Sysmon
## Recherche - Sysmon 0-4P:
### Was ist Sysmon? Wofür wird es verwendet
Sysmon wird für die Überwachung von einem Windows Client verwendet werden.  
Es wird über das Terminal verwaltet und arbeitet nach bestimmten Mustern welche in einer XML Datei definiert wurden.   
Dieses Tool überwacht und protokolliert die Systemaktivitäten und loggt diese in der Ereignisanzeige (Bsp. Prozesse, Dienste und Netzwerkverbindungen). 

### Was ist die aktuellste Version von Sysmon?
Die aktuellste Version ist die 14.13m welche am 29.11.2022 erschienen ist.  
Sysmon kann von der offiziellen Microsoft Seite heruntergeladen werden: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon  

### Was sind Beispiel-Anwendungen/Use-Cases bei welchen Sysmon helfen kann?
z B. könnte ein Windows Client mit Sysmon überwacht werden, bei dem ein Malware Verdacht herrscht.  
So können Netzwerkverbindungen und Tasks protokoliert werden und so ein klareres Bild von dem Client zu erhalten.  
Sysmon kann verwendet werden, um Änderungen an der Windows-Registrierung aufzuzeichnen und zu protokollieren.  

## Installation - Sysmon 0-4P:
### Recherchieren und dokumentieren Sie wie man Sysmon manuell auf einem Windows-Server/Client installieren würde
Im ersten Schritt lädt man die Symon Zip Datei von https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon herunter.  
Dann entzipt man die Datei an den gewünschten Speicherort und fügt die Konfigurationsdatei hinzu (Diese kann z. B. von Github sein).  
In Powershell öffnet man den Speicherort von Sysmon und gibt dort folgenden Befehl für das akzeptieren der Eula ein und das starten von Symon:  
```powershell
.\sysmon.exe -accepteula -i sysmonconfig-export.xml
```

### Recherchieren und dokumentieren Sie wie Sysmon auf Ihren Systemen (automatisiert) installiert wurde
Sysmon wurde laut dem Github Repository mittels "OLAF" installiert und konfiguriert.  
Link zum Tool: https://github.com/olafhartong/sysmon-modular  

Unter "Vagrant\scripts" befindet sich das File "install-sysinternals.ps1".  
Dieses Skript Installiert und Konfiguriert Sysmon. 

## Konfiguration - Sysmon 0-4P:
### Dokumentieren Sie das Config-File welches auf Ihrem System für Sysmon installiert ist
Die Config-Datei befindet sich unter `C:\ProgramData\Sysmon` mit dem Namen `sysmonConfig.xml`.
Darin sind einzelne Bedingung, welche nach gewissen Regeln Dienste, Regestry-Einträge, System- und Netzwerkaktivitäten überprüfen.   
Wenn eines der Bedingungnen zutrifft, dann wird eine Aktion danach ausgeführt.  

So sieht der Beginn der Config Datei aus:  
```xml
-<Sysmon schemaversion="4.60">
    <HashAlgorithms>*</HashAlgorithms>
    <!-- This now also determines the file names of the files preserved (String) -->
    <CheckRevocation>False</CheckRevocation>
    <!-- Setting this to true might impact performance -->
    <DnsLookup>False</DnsLookup>
    <!-- Disables lookup behavior, default is True (Boolean) -->
    <ArchiveDirectory>Sysmon</ArchiveDirectory>
    <!-- Sets the name of the directory in the C:\ root where preserved files will be saved (String)-->
    -<EventFiltering>
    <!-- Event ID 1 == Process Creation - Includes -->
    -<RuleGroup groupRelation="or">
    -<ProcessCreate onmatch="include">
        <ParentImage condition="image" name="technique_id=T1546.008,technique_name=Accessibility Features">sethc.exe</ParentImage>
        <ParentImage condition="image" name="technique_id=T1546.008,technique_name=Accessibility Features">utilman.exe</ParentImage>
        <ParentImage condition="image" name="technique_id=T1546.008,technique_name=Accessibility Features">osk.exe</ParentImage>
        <ParentImage condition="image" name="technique_id=T1546.008,technique_name=Accessibility Features">Magnify.exe</ParentImage>
        <ParentImage condition="image" name="technique_id=T1546.008,technique_name=Accessibility Features">DisplaySwitch.exe</ParentImage>
        <ParentImage condition="image" name="technique_id=T1546.008,technique_name=Accessibility Features">Narrator.exe</ParentImage>
        <ParentImage condition="image" name="technique_id=T1546.008,technique_name=Accessibility Features">AtBroker.exe</ParentImage>
        <OriginalFileName condition="is" name="technique_id=T1546.011,technique_name=Application Shimming">sdbinst.exe</OriginalFileName>
        <OriginalFileName condition="is" name="technique_id=T1197,technique_name=BITS Jobs">bitsadmin.exe</OriginalFileName>
    -<Rule groupRelation="and" name="Eventviewer Bypass UAC">
        <ParentImage condition="image" name="technique_id=T1548.002,technique_name=Bypass User Access Control">eventvwr.exe</ParentImage>
        <Image condition="is not">c:\windows\system32\mmc.exe</Image>
    </Rule>
```

### Dokumentieren Sie die Struktur des Config-Files
Das Config-File ist im XML Format aufgebaut.  

So wird in diesem Beispiel bei der "Process Creation" das Event geloggt, wenn der Name mit `virus.exe` oder `malware.exe` endet:
```xml
<Sysmon schemaversion="4.1">
    <HashAlgorithms>*</HashAlgorithms>
    <EventFiltering>
        <ProcessCreate onmatch="include">
            <Image condition="end with">virus.exe</Image>
            <Image condition="end with">malware.exe</Image>
        </ProcessCreate>
    </EventFiltering>
</Sysmon>
```

Dort wird das Auslösende Event angegeben, mit einer / mehreren Bedigungen welche ausgeöst werden.  
Die einzelnen Tags sind mit einem öffnenden- und schliessenden Tag versehrt.  

### Dokumentieren Sie die Hintergründe der Konfiguration. Folgende Blog-Posts sind wichtig:
* Es kann eine vordefinierte Konfiguration genommen werden, diese muss aber evtl. noch auf das spezifische Unternehmen angepasst werden.
* Standartmässig such Sysmon alle 12 Studen nach einer neuen Version.  
* Damit Sysmon am einfachsten Installiert werden kann, soltle man GPOs verwenden.  

Folgende Links hatten wir zur Verfügung:  
https://medium.com/@olafhartong/endpoint-detection-superpowers-on-the-cheap-part-1-e9c28201ac47  
https://medium.com/@olafhartong/endpoint-detection-superpowers-on-the-cheap-part-2-deploy-and-maintain-d06580329fe8  
https://medium.com/@olafhartong/endpoint-detection-superpowers-on-the-cheap-part-3-sysmon-tampering-49c2dc9bf6d9  
