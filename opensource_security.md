---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## CVE-2024-3094: Die XZ Backdoor
  Eine Aufarbeitung des beinahe-GAUs.
drawings:
  persist: false
transition: slide-left
title: CVE-2024-3094 - Die XZ Backdoor
---

<div class="absolute inset-0 bg-black/50 -z-10"></div>

# [CVE-2024-3094](https://nvd.nist.gov/vuln/detail/cve-2024-3094)
## Der Feind in Open-Source Repository! <span style="color: #3b82f6; font-weight: 800;">Die XZ-Backdoor</span>

Wie das Internet durch Zufall einer Katastrophe entging

<div class="mt-10 opacity-80 text-xl">
  Ingo Siebert
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.next" class="px-2 py-1 rounded hover:bg-white/10">
    Starten <carbon:arrow-right />
  </button>
</div>

---

# Was ist passiert?

Am 29. März 2024 veröffentlichte Andres Freund (Microsoft) seine Entdeckung einer Backdoor in den `xz-utils` (Datenkompression, LZMA).

- **Die Schwachstelle:** Remote Code Execution (RCE) via SSH.
- **Der Mechanismus:** Schadcode wurde in Login-Zertifikaten versteckt; nur aktiv in Verbindung mit `systemd`.
- **Der Zufall:** Gefunden durch Micro-Benchmarking von Postgres – SSH-Logins verbrauchten plötzlich ungewöhnlich viel CPU.
- **Betroffene:** Vor allem "Bleeding Edge" Distributionen wie Debian Unstable, Fedora Rawhide und openSUSE Tumbleweed.


---

# Live-Check der Version

So hätte die Abfrage auf einem infizierten System ausgesehen:

<div class="mt-8 shadow-xl rounded-lg overflow-hidden">
  <div class="bg-gray-800 px-4 py-2 flex items-center gap-2">
    <div class="w-3 h-3 rounded-full bg-red-500"></div>
    <div class="w-3 h-3 rounded-full bg-yellow-500"></div>
    <div class="w-3 h-3 rounded-full bg-green-500"></div>
    <span class="text-gray-400 text-xs ml-2">bash — 80x24</span>
  </div>
  <div class="bg-black p-4 font-mono text-green-400 leading-relaxed">
    <div class="flex gap-2">
      <span class="text-blue-400">ingo@throne:~$</span>
      <span class="text-white">xz --version</span>
    </div>
    <div class="text-gray-300">
      xz (XZ Utils) 5.6.1<br>
      liblzma 5.6.1<br>
      <span class="text-red-500 font-bold"># WARNUNG: Diese Version ist kompromittiert!</span>
    </div>
    <div class="flex gap-2 mt-2">
      <span class="text-blue-400">ingo@throne:~$</span>
      <span class="animate-pulse w-2 h-5 bg-white"></span>
    </div>
  </div>
</div>

---

# Exkurs: Wie funktioniert Open Source?

Um den Angriff zu verstehen, muss man die Rollen in der OSS-Welt kennen:

* **Maintainer**: Haben die Kontrolle über das Projekt, machen Releases und verwalten das Setup.
* **Contributor**: Steuern Code-Änderungen (Patches, Pull Requests/Merge Requests) oder Dokumentation bei.
* **User**: Nutzen die Software, melden Fehler oder wünschen sich Features.

**Das Problem:** Viele kritische Projekte werden von einer einzelnen Person als Hobby betrieben ("Single Point of Failure").

---

# Die Akteure des Angriffs

Der Angriff war kein technischer Exploit, sondern brillantes Social Engineering.

* **Lasse Collin**: Langjähriger Maintainer von `xz-utils`. Er litt unter psychischen Belastungen ("mental health issues").
* **Jia Tan (JiaT75)**: Der Angreifer. Erschlich sich über Jahre das Vertrauen als fleißiger Contributor.
* **Sockpuppets (Jigar Kumar, Dennis Ens)**: Schein-Accounts, die Druck auf Lasse Collin ausübten 
  * "Warum geht es nicht schneller?"
  * "Brauchen wir einen Fork?"

---

# Timeline des Angriffs (1/3)

Ein "Long Con" über mehrere Jahre:

- **2009**: xz-utils 5.0.0 erstes Github release
- **2010-2020**: unregelmäßige Release-Zyklen (1-5 releases, teilweise nur Alpha-Versionen)
- <span class="year2021">Januar 2021</span>: GitHub Account `JiaT75` wird erstellt
- <span class="year2022">2022</span>: Jia Tan beginnt mit kleinen Patches
  - "Jigar Kumar" baut gleichzeitig Druck auf Lasse Collin auf.  
    [Archiv der E-Mails von Jigar Kumar](https://www.mail-archive.com/search?l=xz-devel%40tukaani.org&q=from:%22Jigar+Kumar%22&o=newest)
- <span class="year2022">Juni</span>: Lasse Collin spricht von “mental health issues”  
  - Lasse Collin macht Jia Tan zum Co-Maintainer
- <span class="year2022">Sept</span>: Jia Tan kündigt Release 5.4 und dessen Inhalte an
- <span class="year2022">Dez</span>: Jia Tan pusht erstmals Commits direkt  
  - Ein Commit mit Namen "Jia Cheong Tan"


---

# Timeline des Angriffs (2/3)

Ein "Long Con" über mehrere Jahre:

- <span class="year2023">2023</span>: Jia Tan übernimmt immer mehr Kontrolle
- <span class="year2023">März</span>: Jin Tan ersetzt Kontaktmail bei google/oss-fuzz (Security-Tests) 
- <span class="year2023">Juni</span>: Patches ovn "Hans Jansen", merged von Jian Tan, Vorbereitung für Backdoor. [ifunc hook](https://sourceware.org/glibc/wiki/GNU_IFUNC)
- <span class="year2023">Juli</span>: Jia Tan schaltet ifunc support in oss-fuzz ab
- <span class="year2024">2024</span>: Jahr des Angriffs
- <span class="year2024">Feb</span>: Jia Tan merged backdoor code, versteckt in angeblichen Test-Files (compressed binary files)
  - Jia Tan erstellt xz-utils release v5.6.0
  - v5.6.0 enthält neues build-script -> in deb/rpm Paketen wird der Schadcode aus den “test”-binaries mit ins Installationspaket verpackt
- **Angriffsvorbereitung ist abgeschlossen**

---

# Timeline des Angriffs (3/3)

- <span class="year2024">März</span>: In RedHat Valgrind (Instrumentation Framework) tauchen Fehler im Zusammenhang mit xz-utils v5.6.0 auf
  - Jia Tan committed angeblichen Valgrind fix
  - Jia Tan committed neue binary test files (echter Fix für die Valgrind Probleme)
  - Jia Tan tagged v5.6.1 - mit Backdoor
  - “Hans Jansen" eröffnet Debian bug report und wirbt für v5.6.1. Zwei weitere neue unbekannte Mail-Accounts “unterstützen” diesen Request
  - Jia Tan eröffnet Ubuntu bug report und wirbt für v5.6.1 update
- <span class="year2024">28. März</span>: Andres Freund findet die Schwachstelle beim Debuggen eines Performance-Problems
  - Andres Freund informiert Debian und RedHat “privat”
  - RedHat vergibt CVE-2024-3094
- <span class="year2024">28. März</span>: Andres Freund veröffentlicht Schwachstelle
  - GitHub sperrt die Repos von tukaani und die Accounts von Lasse Collin und Jia Tan
- Die Analyse beginnt, breites Unterstützungsangebot an Lasse Collin

---
layout: two-cols
---

# Wer ist Jia Tan?

Hinter dem Pseudonym wird ein hochprofessioneller Akteur vermutet.

- **Arbeitsweise**: Agiert wie eine APT (Advanced Persistent Threat)
  - staatliche Akteure sind wahrscheinlich.
- **Commit-Zeiten**: Meist UTC+8 (Asien) oder UTC+2 (Europa). Die Zeiten entsprechen europäischen Bürozeiten (9-18 Uhr).
- **Urlaubszeiten**: Die Aktivität pausierte an europäischen Feiertagen, nicht an asiatischen.
- **IP-Adressen**: Spuren führten u.a. zu VPNs und IPs aus Singapur.

::right::

# IRC
![Jia Tan Timeline](/images/lia_backdoor.jpg)

---

# Was heißt das für uns?

### Für uns als Firma
* **Maintainer unterstützen**: Regelmäßige finanzielle Zuwendungen, Maintainer einstellen oder "Features gegen Rechnung"  
* **Zeit einräumen**: Eigene Entwickler ermutigen, während der Arbeitszeit an genutzten OSS-Projekten mitzuarbeiten
* **Supply Chain**: Dependency-Upgrades genauer prüfen und in Audits investieren

<br>

### Für uns als Entwickler
* **Versionmanagement**: Betreibe keine Server mit unstable/testing Distribution
* **Tools nutzen**: Automatisierte Scanner (Dependency Scanner, Renovate, Dependabot) einsetzen
* **Wachsamkeit**: "Hobby-Projekte", die kritisch für uns sind, identifizieren und professionalisieren
* **Wertschätzung**: Einfach mal "Danke" sagen, um den Stress für Maintainer zu senken

---

# Fazit & Diskussion

> "If your hobby project is used by thousands of people, it is no longer a hobby." 

<br>

- Wie sicher sind unsere eigenen Abhängigkeiten?
- Könnten wir einen solchen Angriff in unseren Libraries entdecken?

**Vielen Dank für eure Aufmerksamkeit!**  

Fragen?

<br>

Quellen:
- "Wie kommen eigentlich Hintertürchen in Open Source Software?" Christian Kühn, dmTech  
  Weitere Quellen in seinen Folien
- https://research.hisolutions.com/2024/04/xz-backdoor-eine-aufarbeitung/
- https://boehs.org/node/everything-i-know-about-the-xz-backdoor