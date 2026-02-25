# JJL Ranking System - Dokumentation

Ein Google Sheets-basiertes Ranking-System für Teamsportarten mit wechselnden Teamzusammenstellungen, Satz-Erfassung und mehreren Rating-Systemen.
100% vibe-coded mit Claude

---

## Metriken und Rankings verstehen

Das System berechnet verschiedene Metriken, die unterschiedliche Aspekte der Spielstärke messen.

### Skill Rating (Hauptmetrik)

**Was ist das?**  
Das Skill Rating ist die zentrale Metrik zur Bewertung der Spielstärke. Es kombiniert Konzepte aus dem bekannten ELO-System (Schach) mit TrueSkill-Prinzipien (Microsoft Multiplayer Matchmaking).

**Wie wird es berechnet?**  
Jede Person startet mit 1500 Punkten. Nach jedem Match wird das Rating angepasst basierend auf:
- Ob du gewonnen, verloren oder unentschieden gespielt hast
- Wie stark deine Gegner waren (Siege gegen Starke bringen mehr Punkte)
- Wie deutlich das Match war (Dominanz-Faktor, siehe unten)
- Wie sicher das System über deine Stärke ist (Unsicherheit)

**Wie interpretiere ich die Zahlen?**
- **1500:** Durchschnitt (Startwert)
- **1600+:** Überdurchschnittlich stark
- **1700+:** Sehr stark
- **1400-:** Unterdurchschnittlich
- **±100 Punkte:** Deutlicher Unterschied in der Spielstärke

**Wichtig zu wissen:**
- Das Rating passt sich schneller an, wenn du neu bist (hohe Unsicherheit)
- Nach etwa 10 Matches wird das Rating stabiler
- Nach etwa 20 Matches ist es verlässlich
- Das System ist "Zero-Sum": Was du gewinnst, verliert dein Gegner

**Beispiel:**  
Du hast 1550 Punkte und spielst gegen jemanden mit 1450 Punkten. Du wirst erwartet zu gewinnen. Wenn du gewinnst, bekommst du weniger Punkte (+15) als wenn du unerwartet gegen jemanden mit 1650 gewinnst (+35).

### Unsicherheit

**Was ist das?**  
Die Unsicherheit zeigt, wie sicher sich das System über deine wahre Spielstärke ist.

**Wie wird sie berechnet?**  
Alle starten mit 350 Punkten Unsicherheit. Mit jedem Match sinkt dieser Wert um etwa 10% (Faktor 0.9), bis ein Minimum von 50 erreicht ist.

**Wie interpretiere ich die Zahlen?**
- **350:** Brandneu, keine Daten
- **200-250:** 5-10 Matches gespielt, erste Tendenz
- **100-150:** 15-20 Matches, recht stabil
- **50:** 35+ Matches, maximal stabil

**Warum ist das wichtig?**  
Eine hohe Unsicherheit bedeutet, dass dein Rating noch stark schwanken kann. Eine niedrige Unsicherheit bedeutet, dass dein Rating verlässlich ist. Das System nutzt die Unsicherheit, um zu entscheiden, wie stark sich dein Rating nach einem Match ändert.

**Beispiel:**  
Person A: Rating 1600, Unsicherheit 250 → Ein Sieg kann +50 Punkte bringen  
Person B: Rating 1600, Unsicherheit 80 → Ein Sieg bringt nur +25 Punkte

### Conservative Rating

**Was ist das?**  
Das Conservative Rating ist eine vorsichtige Schätzung deiner Mindeststärke.

**Wie wird es berechnet?**  
```
Conservative Rating = Skill Rating - (3 × Unsicherheit)
```

**Wie interpretiere ich die Zahlen?**  
Das Conservative Rating sagt: "Mit 99.7% Wahrscheinlichkeit bist du mindestens so stark."

**Wann ist es nützlich?**
- Beim Vergleichen von Personen mit unterschiedlich vielen Matches
- Für faire Team-Zusammenstellungen (Draft)
- Um "Glücks-Streaks" zu erkennen

**Beispiel:**  
Person A: Skill 1650, Unsicherheit 250 → Conservative 900 (sehr unsicher!)  
Person B: Skill 1580, Unsicherheit 60 → Conservative 1400 (verlässlich stark)  
→ Person B ist nachweislich stärker, obwohl Person A höheres Skill Rating hat

### ELO

**Was ist das?**  
Das klassische ELO-System aus dem Schach, angepasst für Team-Spiele.

**Wie wird es berechnet?**  
Ähnlich wie Skill Rating, aber ohne Unsicherheitsfaktor. Alle starten mit 1500, der K-Faktor ist konstant 32. Auch hier wird die Match-Dominanz berücksichtigt.

**Wie interpretiere ich die Zahlen?**  
Die Skala ist identisch zum Skill Rating (1500 = Durchschnitt).

**Warum gibt es ELO zusätzlich?**  
ELO ist einfacher zu verstehen und ein etablierter Standard. Viele kennen es aus Schach oder anderen Sportarten. In der Praxis liegen Skill Rating und ELO meist nah beieinander, wobei das Skill Rating etwas differenzierter ist.

### Wertungspunkte

**Was ist das?**  
Eine einfache Erfolgsquote ohne Berücksichtigung der Gegnerstärke.

**Wie werden sie berechnet?**
- **Sieg:** +1 Punkt
- **Unentschieden:** +0.5 Punkte
- **Niederlage:** 0 Punkte

**Warum ist das wichtig?**  
Wertungspunkte sind die einfachste Metrik. Sie ignorieren zwar die Gegnerstärke, geben aber einen schnellen Überblick über den reinen Erfolg.

**Beispiel:**  
Person A: 14 Punkte aus 20 Matches (70% Erfolgsquote)  
Person B: 12 Punkte aus 20 Matches (60% Erfolgsquote)  
→ Person A hat mehr Erfolg, unabhängig von der Gegnerstärke

### Buchholz-Zahl

**Was ist das?**  
Die Buchholz-Zahl misst die Stärke deines Spielplans durch die Summe der Wertungspunkte aller deiner Gegner.

**Wie wird sie berechnet?**  
```
Buchholz = Summe der Wertungspunkte aller Gegner
```

**Wie interpretiere ich die Zahlen?**  
Bei 20 Matches gegen 15 verschiedene Gegner:
- **150+:** Sehr starker Spielplan (Durchschnittsgegner hat 10+ Punkte)
- **100-150:** Normaler Spielplan (Durchschnittsgegner hat 7-10 Punkte)
- **<100:** Schwacher Spielplan (Durchschnittsgegner hat <7 Punkte)

**Warum ist das wichtig?**  
Buchholz ist ein Tiebreaker bei gleichen Wertungspunkten. Zwei Personen mit je 12 Punkten sind nicht gleich stark, wenn eine gegen deutlich stärkere Gegner gespielt hat.

**Beispiel:**  
Person A: 12 Wertungspunkte, Buchholz 140 → hat gegen starke Gegner gespielt  
Person B: 12 Wertungspunkte, Buchholz 90 → hat gegen schwächere Gegner gespielt  
→ Person A sollte im Ranking höher stehen

### Durchschnittliche Dominanz

**Was ist das?**  
Die durchschnittliche Dominanz zeigt, wie deutlich deine Matches im Schnitt ausgehen.

**Wie wird sie berechnet?**  
Für jeden Satz: `(Eigene Juggs - Gegner Juggs) / (Juggs gesamt)`  
Dann wird über alle Sätze gemittelt.

**Wie interpretiere ich die Zahlen?**  
Der Wert liegt zwischen -1 und +1:
- **+0.20 bis +0.40:** Du dominierst deine Matches klar (z.B. oft 7:3, 7:2)
- **+0.10 bis +0.20:** Du gewinnst meist deutlich oder verlierst knapp
- **-0.05 bis +0.05:** Deine Matches sind sehr ausgeglichen
- **-0.10 bis -0.20:** Du verlierst meist deutlich oder gewinnst knapp
- **-0.20 bis -0.40:** Du verlierst meist deutlich

**Warum ist das wichtig?**  
Die Dominanz zeigt die Qualität deiner Siege/Niederlagen. Sie beeinflusst auch leicht die Rating-Änderungen (deutliche Siege bringen etwas mehr Punkte, knappe Niederlagen kosten etwas weniger).

**Beispiel:**  
Person A: Rating 1520, Dominanz +0.22 → Spielt sehr souverän  
Person B: Rating 1580, Dominanz -0.05 → Gewinnt knapp, verliert deutlich  
→ Person A könnte stärker sein als das Rating vermuten lässt

**Besonderheit:**  
Die Dominanz ist unabhängig von der Satzhöhe. Ein 7:5 Satz (Dominanz +0.167) ist genauso dominant wie ein (hypothetischer) 21:15 Satz (Dominanz +0.167).

### Schiedseinsätze

**Was ist das?**  
Anzahl der Matches, bei denen dein Team Schiedspersonen gestellt hat.

**Wichtig:**  
Schiedseinsätze zählen NICHT als gespielte Matches und beeinflussen KEINE Ratings. Sie verdeutlichen aber die "Care-Arbeit". :D

---

## Den Rankings-Tab verstehen

Das Rankings-Tab zeigt alle Metriken in einer übersichtlichen Tabelle, sortiert nach Skill Rating (absteigend).

### Spalten im Überblick

| Spalte | Bedeutung | Gut ist... |
|--------|-----------|------------|
| Rang | Position im Ranking | Niedriger |
| Skill Rating | Hauptmetrik | Höher |
| Unsicherheit | Wie stabil ist das Rating? | Niedriger |
| Conservative | Garantierte Mindeststärke | Höher |
| ELO | Alternative Metrik | Höher |
| Gesamt | Anzahl Matches | Mehr = verlässlicher |
| Siegquote % | Erfolgsquote | Höher |
| Wertungspunkte | Einfache Erfolgsmetrik | Höher |
| Buchholz | Spielplan-Stärke | Höher bei gleichen Wertungspunkten |
| Ø Dominanz | Wie deutlich gewinnst/verlierst du? | Positiver |
| Juggs +/- | Erzielte/erhaltene Punkte | Positive Differenz |
| Diff | Juggs-Saldo | Positiver |
| Spieltage/Matches/Sätze | Aktivität | Mehr = aktiver |

### Farbcodierung

- **Gold (1. Platz):** Höchstes Skill Rating
- **Silber (2. Platz):** Zweithöchstes Skill Rating
- **Bronze (3. Platz):** Dritthöchstes Skill Rating

---

## Häufige Fragen zur Interpretation

**Warum habe ich nach einem Sieg nur wenige Punkte bekommen?**  
Du hast gegen laut Datenlage schwächere Teams gewonnen. Das System erwartet, dass du gewinnst, daher gibt es weniger Punkte. Umgekehrt: Siege gegen Stärkere bringen mehr. Das muss nicht der Realität entsprechen, aber das System kann nur mit den Daten arbeiten, die es hat.

**Mein Skill Rating ist höher als mein ELO. Warum?**  
Beide Systeme berechnen sich leicht unterschiedlich. Der Unterschied sollte aber klein sein (<50 Punkte). Größere Abweichungen können bei hoher Unsicherheit auftreten.

**Ich habe eine höhere Siegquote als jemand anderes, aber niedrigeres Rating. Warum?**  
Du hast wahrscheinlich gegen laut Datenlage schwächere Teams gespielt. Das Rating berücksichtigt die Gegnerstärke, die Siegquote nicht.

**Was ist ein gutes Conservative Rating?**  
Das hängt von deiner Aktivität ab. Nach 20+ Matches sollte das Conservative Rating etwa 150 Punkte unter deinem Skill Rating liegen. Größere Abstände bedeuten hohe Unsicherheit.

**Meine Dominanz ist negativ, aber mein Rating ist gut. Wie passt das zusammen?**  
Du gewinnst Matches (daher gutes Rating), aber deine Siege sind oft knapp und deine Niederlagen deutlich.

**Wie viele Matches brauche ich für ein verlässliches Rating?**  
Ab 10 Matches: Erste Tendenz  
Ab 20 Matches: Verlässlich  
Ab 35 Matches: Maximal stabil

**Was ist wichtiger: Skill Rating oder Wertungspunkte?**  
Skill Rating, weil es die Gegnerstärke berücksichtigt. Wertungspunkte sind nur ein grober Indikator.

**Kann ich mein Rating "farmen" durch Spiele gegen Schwächere?**  
Nein. Siege gegen Schwächere bringen sehr wenige Punkte, Niederlagen gegen sie kosten aber viele. Das System ist Zero-Sum und versucht Farming-Effekte zu verhindern.

**Warum haben zwei Personen mit gleichen Wertungspunkten unterschiedliche Ratings?**  
Weil sie gegen unterschiedlich starke Gegner gespielt haben. Die Person mit höherem Rating hat gegen stärkere Gegner gespielt (siehe Buchholz).

---

## Welche Metrik für was nutzen?

**Für das Haupt-Ranking:**  
Skill Rating (Standard-Sortierung)

**Für Team-Balance (Draft):**  
Conservative Rating (verhindert Überschätzung neuer Spielender)

**Für Turniere mit wenigen Matches:**  
Wertungspunkte + Buchholz (klassisches Turnier-System)

**Für Motivations-Rankings:**  
Siegquote oder Wertungspunkte (einfach zu verstehen)

**Für Detail-Analysen:**  
Dominanz, Juggs-Differenz (zeigt Spielstil und Entwicklung)

---

**Februar 2025**
