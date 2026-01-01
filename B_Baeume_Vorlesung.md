# Implementierung Relationaler Datenbanksysteme: B-Bäume und B*-Bäume
## Umfassende Dokumentation zur Vorlesung

---

## 1. Verzeichnis als Generischer Datentyp

### 1.1 Grundkonzept

Ein **Verzeichnis (Directory)** ist ein generischer abstrakter Datentyp, der Kollektionen von Datenobjekten verwaltet.

**Schlüsseleigenschaften:**
- **Generisch**: Die Datenstruktur ist konzeptuell unabhängig vom konkreten Datentyp
- **Typisiert**: Verwendet Typparameter (z.B. generische Listen in Java)

**Beispiel in Java:**
```java
// Generische Liste mit Typparameter T
ArrayList<String> names = new ArrayList<>();
ArrayList<Integer> ids = new ArrayList<>();
```

### 1.2 Verzeichnis als Datentyp: `directory[S,C]`

**Definition:**
- `S` = Schlüsselwerttyp (muss Vergleichsoperationen unterstützen: `>`, `<`, `=`)
- `C` = Content-Typ (beliebiger Datentyp, auch wieder `directory`)

**Beispiel aus einer Universität:**
```
Matrikelnummer (S) → Studierendendaten (C)
123456            → {Name: "Max Müller", Alter: 21, ...}
123457            → {Name: "Anna Schmidt", Alter: 20, ...}
```

### 1.3 Typische Operationen auf Verzeichnissen

| Operation | Signatur | Beschreibung |
|-----------|----------|-------------|
| **create()** | `create():V` | Leeres Verzeichnis anlegen |
| **insert()** | `insert(V,S,C)` | Eintrag einfügen/überschreiben |
| **delete()** | `delete(V,S)` | Eintrag mit Schlüssel S löschen |
| **read()** | `read(V,S):C` | Inhalt zum Schlüssel S abrufen |
| **next_key()** | `next_key(V,S):S` | Nächstgrößerer Schlüsselwert |
| **read_interval()** | `read_interval(V,S,S):Liste[C]` | Alle Einträge in Intervall |

### 1.4 Effizienzanforderungen

1. **Laufzeiteffizienz**: Minimale Anzahl Speicherzugriffe
2. **Speicherplatzausnutzung**: Nutzfläche > 50% des Gesamtspeicherplatzes

---

## 2. Grundlagen von B-Bäumen

### 2.1 Warum B-Bäume?

**Problem:** Binäre Suchbäume können unbalanciert wachsen
- Speicherplatzausnutzung unzureichend
- Zu viele Speicherzugriffe beim Suchen

**Lösung:** B-Bäume sind
- ✅ Balancierte Suchbäume
- ✅ Mehrweg-Bäume (n-ary trees)
- ✅ Speicher-orientiert
- ✅ Effizienz für Suche, Durchlauf und Intervallabfragen

### 2.2 Von binären zu Mehrweg-Suchbäumen

**Binärer Suchbaum (2-Weg):**
```
       3
      / \
     2   5
```
- 1 Schlüsselwert pro Knoten
- 2 Unterbäume (links, rechts)

**Mehrweg-Suchbaum (n-Weg):**
```
        s₁  s₂  s₃
       /   |   |   \
      T₁  T₂  T₃  T₄  T₅
```
- `n` Schlüsselwerte: s₁, s₂, ..., sₙ
- `n+1` Unterbäume: T₁, T₂, ..., Tₙ₊₁

**Eigenschaft:** Für Schlüsselwert `x` in Teilbaum Tᵢ gilt:
```
s(i-1) < x < s(i)
```

### 2.3 B-Baum Definition

Ein B-Baum der Ordnung `m` erfüllt:

| Eigenschaft | Definition |
|-------------|-----------|
| **Knotenbesetzung** | Jeder Knoten (außer Wurzel): mindestens `m`, maximal `2m` Schlüsselwerte |
| **Wurzelknoten** | 1 ≤ n ≤ 2m Schlüsselwerte |
| **Unterbäume** | Ein Knoten mit n Schlüsselwerten hat genau n+1 nichtleere Unterbäume |
| **Balance** | Alle Blätter haben die gleiche Tiefe |

### 2.4 Verzweigungsgrad berechnen

Ziel: Einen Baumknoten optimal in einen Speicherblock füllen.

**Rechenbeispiel:**
```
Blockgröße b = 2048 Bytes
Verweis t = 8 Bytes
Schlüsselwert k = 8 Bytes
Dateninhalt i = 110 Bytes

Speicherplatz pro Eintrag = k + i + t = 126 Bytes
```

**Formel für maximale Schlüsselwerte:**
```
n = (b - t) / (k + i + t)
n = (2048 - 8) / (8 + 110 + 8) = 2040 / 126 ≈ 16
```

Der Knoten kann also 16 Schlüsselwerte speichern.
Verzweigungsgrad = 16 + 1 = 17 Unterbäume

### 2.5 Suchaufwand in B-Bäumen

**Suchaufwand analysieren:**

Mit Höhe `h` und minimaler Knoten-besetzung:

```
Anzahl Knoten in Teilbaum (Höhe 1): 1 + m·(m+1)⁰
Anzahl Knoten in Teilbaum (Höhe 2): 1 + m·(m+1)¹
Anzahl Knoten allgemein:            1 + m·[(m+1)^h - 1] / m
                                   = 1 + (m+1)^h - 1
                                   = (m+1)^h
```

**Minimale Schlüsselwerte bei Höhe h:**
```
n ≥ m · [(m+1)^h - 1] / m = (m+1)^h - 1
```

**Maximalhöhe bei n Schlüsselwerten:**
```
h ≤ log_(m+1)((n+1)/2)
```

**Beispiel:**
```
n = 1.000.000 Einträge
m = 8 (Ordnung)

h ≤ log₉(1.000.000 + 1)/2 ≈ 5,97 ≤ 6

→ Maximum 7 Speicherzugriffe (6 Ebenen + 1)
```

### 2.6 Beispiel: B-Baum der Ordnung m=2

```
              34
           /      \
          /        \
      [12]        [76]
      /    \      /    \
    [2,5] [17,19] [42,50,59,70] [83,102]
```

**Eigenschaften dieses Baums:**
- Ordnung m = 2
- Min. Einträge pro Knoten (außer Wurzel) = 2
- Max. Einträge pro Knoten = 4
- Alle Blätter auf gleicher Ebene

---

## 3. Operationen auf B-Bäumen

### 3.1 Suchen (READ-Operation)

**Algorithmus `read(bb, s, c):`**

1. Starte im Wurzelknoten
2. Durchlaufe Schlüsselwerte von links nach rechts
   - Wenn `s = s'`: Gebe Inhalt c zurück
   - Wenn `s < s'`: Gehe in linken Unterbaum
   - Wenn `s > s'`: Versuche nächsten Schlüsselwert
   - Wenn `s > letztem Schlüsselwert`: Gehe in rechten Unterbaum
3. Falls kein Blatt mit Schlüssel gefunden: Schlüssel existiert nicht

**Zeitkomplexität:** O(log n) Speicherzugriffe

**Beispiel - Suche nach 50:**
```
Start: 34                  (50 > 34 → rechts)
       ├─ 76              (50 < 76 → links in [42,50,59,70])
Blatt: 42,50,59,70        (50 gefunden!)
```

### 3.2 Einfügen (INSERT-Operation)

**Grundalgorithmus:**
1. Finde Blattknoten, wo Eintrag eingefügt werden soll
2. Füge Eintrag hypothetisch ein
3. Falls Knoten überläuft (> 2m Einträge) → Überlaufbehandlung

### 3.3 Überlaufbehandlung (Split)

**Problem:** Knoten hat 2m+1 Einträge (zu viele)

**Lösung: Knoten in der Mitte teilen**

```
Übergelaufener Knoten mit 2m+1 Einträgen:
[s₁ ... sₘ | sₘ₊₁ | sₘ₊₂ ... s₂ₘ₊₁]

                    ↓ Split bei sₘ₊₁

Linker Knoten: [s₁ ... sₘ]  (m Einträge)
Mittlerer Schlüssel: sₘ₊₁   (wandert nach oben)
Rechter Knoten: [sₘ₊₂ ... s₂ₘ₊₁]  (m Einträge)
```

**Rekursion:** Falls Elternknoten auch überläuft, wiederholen

**Besonderheit: B-Bäume wachsen nach oben!**
```
Normalerweise: Bäume wachsen nach unten (neue Blätter)
B-Baum: Überlauf in Wurzel → neue Wurzel (Höhe nimmt zu)
```

**Beispiel: Einfügen von 5 in B-Baum der Ordnung m=2**

```
Schritt 1: Einfügen in Blatt
[19,34,50,102] → [5,19,34,50,102] (Überlauf!)

Schritt 2: Split bei 34
Linkes Blatt: [5,19]
Mittlerer Schlüssel: 34 (nach oben)
Rechtes Blatt: [50,102]

Ergebnis:
        [34]
       /    \
    [5,19]  [50,102]
```

### 3.4 Löschen (DELETE-Operation)

**Grundalgorithmus:**
1. Finde Knoten `N` mit Schlüsselwert `s`
2. Falls `N` ein innerer Knoten:
   - Ersetze `s` durch nächstgrößeren Schlüsselwert `s'` aus rechtem Unterbaum
   - Lösche `s'` aus Blattknoten
3. Falls `N` ein Blattknoten:
   - Entferne `s` direkt
4. Falls `N` nur noch m-1 Einträge hat → Unterlaufbehandlung

### 3.5 Unterlaufbehandlung

**Problem:** Knoten hat nur m-1 Einträge (zu wenig)

**Lösung Option 1: Ausgleich (Rebalancing)**

Falls Nachbarknoten `R` hat ≥ m+1 Einträge:
```
Knoten N:  [... sₘ₋₁]  (m-1 Einträge)
Nachbar R: [s₁, s₂, s₃ ...]  (m+k Einträge, k≥1)

         ↓ Ausgleich

Knoten N:  [... x, s₁ ...]
Nachbar R: [s₂, s₃ ...]
Parent:    x ersetzt den alten Trennschlüssel
```

**Lösung Option 2: Verschmelzung (Merge)**

Falls beide Nachbarknoten haben ≤ m Einträge:
```
Knoten N:    [... sₘ₋₁]        (m-1 Einträge)
Nachbar R:   [...]             (m Einträge)
Trennwert x: (aus Elternknoten)

         ↓ Merge

Neuer Knoten: [... sₘ₋₁, x, ...]  (2m Einträge, vollbesetzt)
Nachbar R entfernt
```

**Rekursion:** Falls Elternknoten unterläuft, wiederholen

**Spezialfall: Wurzel unterläuft**
- Wenn Wurzel nur noch 1 Kind hat: Kind wird neue Wurzel
- Baumhöhe nimmt ab

---

## 4. Praktische Beispiele: Operationen auf B-Bäumen

### 4.1 Einfügen: Schritt für Schritt (m=2)

**Schritt 1-4: Einfügen von 50, 102, 34, 19**
```
50               [50]              [50,102]          [34,50,102]
  ↓              ↓                  ↓                 ↓
[50]         [50,102]          [34,50,102]       [19,34,50,102]
```

**Schritt 5: Einfügen von 5 → Überlauf**
```
[5,19,34,50,102] (5 Einträge, aber max = 4)
      ↓ Split bei 34
      
      [34]
     /    \
  [5,19]  [50,102]
```

**Schritt 6-8: Einfügen von 76, 42, 2**
```
        [34]              [34]
       /    \            /    \
    [5,19] [50,102]  [5,19] [42,50,76,102]
      ↓                ↓
    ...              [2,5,19,42,50,76,102]
```

**Schritt 9: Einfügen von 83 → Überlauf und Rekursion**
```
Vorher:          [2,5,19,42,50,76,102] (7 Einträge - Überlauf!)
                       ↓ Split bei 50
    
Nachher:              [34,50]
                     /   |   \
                 [2,5,19] [42,76] [83,102]
```

### 4.2 Löschen: Schritt für Schritt (m=2)

**Initial:** 
```
                [34,50]
               /   |   \
           [2,5] [42,76] [83,102]
```

**Schritt 1: Löschen von 83 → Unterlauf**
```
[83,102] → [102] (1 Eintrag, aber min = 2)

Ausgleich mit Nachbar [42,76]:
    [42,76] hat 2 Einträge, kann Eintrag abgeben
    
Nachher:
            [34,42]
           /   |   \
       [2,5] [50,76] [102]
```

**Schritt 2: Löschen von 2 → Unterlauf → Merge**
```
[2,5] → [5] (1 Eintrag, Ausgleich nicht möglich)

Merge mit Nachbar [50,76] über Trennwert 34:
    
Nachher:
        [34,50]
       /       \
   [5,42]   [76,102]
```

---

## 5. B*-Bäume

### 5.1 Motivation: Warum B*-Bäume?

**Ziel:** Noch effizientere Speichernutzung und weniger Speicherzugriffe

**Probleme bei B-Bäumen:**
- Innere Knoten speichern vollständige Einträge (Schlüssel + Inhalt)
- Platzvergeudung: Viele Inhalte in inneren Knoten, die nur zur Navigation dienen

**Idee:**
- Innere Knoten: nur **Schlüsselwerte** (Indexfunktion)
- Blattknoten: nur **vollständige Einträge** (Daten)

### 5.2 Struktur eines B*-Baums

```
        [34' | 76']           Innere Knoten (nur Schlüssel)
        /     |      \
    [5,19]  [34,50]  [83,102]  Blattknoten (Schlüssel + Inhalt)
    
34' = Schlüssel ohne Inhalt (nur für Navigation)
34 = Schlüssel mit Inhalt (vollständiger Eintrag)
```

### 5.3 Vorteile von B*-Bäumen

| Vorteil | Erklärung |
|---------|-----------|
| **Höherer Verzweigungsgrad** | Innere Knoten enthalten mehr Verweise (weniger Inhalte) |
| **Flacherer Baum** | Geringere Höhe → weniger Speicherzugriffe |
| **Bessere Speicherausnutzung** | Innere Knoten kompakter |
| **Weniger innere Knoten** | Mehr Platz für Hauptspeicher-Caching |

### 5.4 Nachteile von B*-Bäumen

| Nachteil | Erklärung |
|----------|-----------|
| **Immer Bläter erreichen** | Suche muss immer bis zu Blatt gehen (kein vorzeitiger Abbruch) |
| **Redundanz** | Schlüsselwerte doppelt gespeichert (innere Knoten + Blätter) |
| **Komplexere Implementierung** | Unterschiedliche Logik für innere und Blattknoten |

### 5.5 Speicherplatz-Vergleich: B-Baum vs. B*-Baum

**Annahmen:**
```
b = 2048 Bytes (Blockgröße)
t = 8 Bytes (Verweis)
k = 8 Bytes (Schlüsselwert)
i = 110 Bytes (Dateninhalt)
```

**B-Baum (vollständige Einträge in allen Knoten):**
```
n = (b - t) / (k + i + t) = (2048 - 8) / 126 ≈ 16 Einträge
Verzweigungsgrad ≈ 17
Maximalhe für 1 Mio. Einträge: h ≤ log₁₇((10⁶+1)/2) ≈ 5,97 → 6 Ebenen
```

**B*-Baum (nur Schlüssel in inneren Knoten):**
```
Innere Knoten: n_inner = (b - t) / (k + t) = (2048 - 8) / 16 = 127 Einträge
Blattknoten: n_leaf = (b - t) / (k + i + t) = 16 Einträge
Verzweigungsgrad innerer Knoten ≈ 128
Maximalhöhe für 1 Mio. Einträge: h ≤ log₁₂₈((10⁶+1)/2) ≈ 2,8 → 3 Ebenen!
```

**Resultat:** B*-Baum ist deutlich flacher (3 Ebenen statt 6)!

---

## 6. Unterschied: Primärindex vs. Sekundärindex

### 6.1 Primärindex (B-Baum)

**Eigenschaft:**
- **Bestandteil der Primärdaten** (integriert in die Datenbank)
- **Sortierung** nach Ordnungsrelation auf Schlüsselwert
- **Eindeutigkeit:** Maximal **ein** Primärindex pro Relation
- **Primärschlüssel:** Der durch Primärindex unterstützte Suchschlüssel

**Beispiel:**
```
Tabelle: Studenten
Primärschlüssel: Matrikelnummer
→ B-Baum organisiert alle Datensätze nach Matrikelnummer
```

### 6.2 Sekundärindex (separate Struktur)

**Eigenschaft:**
- **Separater Index**, nicht integriert in Primärdaten
- **Mehrere möglich** pro Relation
- **Verweise** auf Primärdaten
- Unterstützen alternative Suchschlüssel

---

## 7. Praktische Anwendung: Speicherplatz-Optimierung

### 7.1 Szenario: Universitätsdatenbank

**Daten:**
```
Blockgröße: 4096 Bytes
Verweis pro Datensatz: 8 Bytes
Schlüsselwert (ID): 8 Bytes
Studentendaten (Name, Alter, Adresse, etc.): 250 Bytes
```

**Berechnung B-Baum der Ordnung m=3:**
```
Speicher pro Eintrag = 8 + 250 + 8 = 266 Bytes
Max. Einträge pro Knoten = (4096 - 8) / 266 ≈ 15

Für 100.000 Studenten:
h ≤ log₄(100.001/2) ≈ 8,2 Ebenen
```

**Optimierung mit B*-Baum:**
```
Innere Knoten: nur Schlüssel (8 Bytes)
Max. Einträge = (4096 - 8) / (8 + 8) = 256

Für 100.000 Studenten:
h ≤ log₂₅₇(100.001/2) ≈ 2,5 Ebenen  ← Drastisch besser!
```

---

## 8. Checkliste: Wann wähle ich welche Struktur?

| Anforderung | B-Baum | B*-Baum |
|-------------|--------|---------|
| Häufige Suche nach einzelnen Einträgen | ✅ Gut | ✅ Besser (cache-friendly) |
| Häufige Range-Queries | ⚠️ OK | ✅ Optimal (Blätter verlinkt) |
| Begrenzte Cache-Größe | ⚠️ OK | ✅ Besser |
| Einfache Implementierung | ✅ | ❌ |
| Häufige Updates | ✅ Einfacher | ⚠️ Komplexer |
| Datensätze klein | ✅ | ✅✅ (profitiert mehr) |
| Datensätze groß | ✅ | ✅ |

---

## 9. Zusammenfassung der Komplexität

### 9.1 Zeitkomplexität

| Operation | Beste Fall | Schlimmste Fall |
|-----------|-----------|-----------------|
| **Suchen** | O(log n) | O(log n) |
| **Einfügen** | O(log n) | O(log n) |
| **Löschen** | O(log n) | O(log n) |
| **Range-Abfrage** | O(log n + k) | O(log n + k) |

(n = Anzahl Einträge, k = Einträge im Ergebnis)

### 9.2 Speicherkomplexität

| Struktur | Speicher | Bemerkung |
|----------|----------|-----------|
| B-Baum | O(n) | Alle Einträge im Baum |
| B*-Baum | O(n) | Alle Einträge (teilweise redundant in inneren Knoten) |

---

## 10. Häufige Fehler und Lösungen

### 10.1 Fehler bei Überlaufbehandlung

**❌ Falsch: Knoten mit 2m Einträgen teilen**
```
Das ist noch OK! Erst bei 2m+1 Einträgen splitten.
```

**✅ Richtig: Nur bei 2m+1 Einträgen (nach Insert) teilen**
```
Normalzustand: ≤ 2m Einträge
Nach Insert: möglicherweise 2m+1 → Split
```

### 10.2 Fehler bei Unterlaufbehandlung

**❌ Falsch: Immer verschmelzen**
```
Könnte innere Knoten auch überlasten!
Erst prüfen: Kann Nachbar Eintrag abgeben?
```

**✅ Richtig: Ausgleich vor Merge**
```
1. Prüfe Nachbarknoten
2. Falls Nachbar ≥ m+1 Einträge → Ausgleich
3. Sonst → Merge
```

### 10.3 Fehler bei Einfügen in innere Knoten

**❌ Falsch: Eintrag in beliebigen inneren Knoten einfügen**
```
Suchbaum-Eigenschaft würde verletzt!
```

**✅ Richtig: Immer in Blattknoten einfügen**
```
1. Finde richtigen Blattknoten
2. Einfügen dort
3. Ggf. Split propagiert nach oben
```

---

## 11. Implementierungs-Tipps

### 11.1 Datenstruktur für B-Baum-Knoten

```python
class BTreeNode:
    def __init__(self, t, leaf=False):
        self.keys = []           # Schlüsselwerte
        self.children = []       # Verweise auf Unterbäume
        self.values = []         # Zugehörige Inhalte
        self.leaf = leaf         # Ist es ein Blatt?
        self.t = t               # Ordnung m
        
    def is_full(self):
        return len(self.keys) == 2 * self.t - 1
    
    def has_few_keys(self):
        return len(self.keys) < self.t
```

### 11.2 Such-Algorithmus

```python
def search(node, key):
    i = 0
    # Finde den ersten Schlüssel >= key
    while i < len(node.keys) and key > node.keys[i]:
        i += 1
    
    if i < len(node.keys) and key == node.keys[i]:
        return node.values[i]  # Gefunden!
    
    if node.leaf:
        return None  # Nicht im Baum
    
    # Rekursiv im richtigen Kind suchen
    return search(node.children[i], key)
```

### 11.3 Einfügen ohne Präventivem Split (einfacher)

```python
def insert(root, key, value):
    if root.is_full():
        # Root ist voll - neuer Root nötig
        new_root = BTreeNode(root.t, leaf=False)
        new_root.children.append(root)
        split_child(new_root, 0)
        insert_non_full(new_root, key, value)
        return new_root
    else:
        insert_non_full(root, key, value)
        return root

def insert_non_full(node, key, value):
    i = len(node.keys) - 1
    
    if node.leaf:
        # Einfügen im Blatt
        node.keys.append(None)
        node.values.append(None)
        while i >= 0 and key < node.keys[i]:
            node.keys[i+1] = node.keys[i]
            node.values[i+1] = node.values[i]
            i -= 1
        node.keys[i+1] = key
        node.values[i+1] = value
    else:
        # Kind suchen
        while i >= 0 and key < node.keys[i]:
            i -= 1
        i += 1
        
        if node.children[i].is_full():
            split_child(node, i)
            if key > node.keys[i]:
                i += 1
        
        insert_non_full(node.children[i], key, value)
```

---

## 12. Zusammenfassung: B-Baum vs. B*-Baum

| Kriterium | B-Baum | B*-Baum |
|-----------|--------|---------|
| **Komplexität (Such)** | O(log n) | O(log n) |
| **Komplexität (Insert)** | O(log n) | O(log n) |
| **Komplexität (Delete)** | O(log n) | O(log n) |
| **Verzweigungsgrad** | m bis 2m+1 | deutlich höher bei inneren |
| **Baumhöhe** | h = log₍ₘ₊₁₎(n) | h ≈ log₁₂₈(n) (viel flacher) |
| **Speichereffizienz** | ~50-80% | Ähnlich, aber innere Knoten voller |
| **Cache-Effizienz** | Gut | Besser (innere Knoten kleiner) |
| **Range-Queries** | OK | Sehr gut (verkettete Blätter) |
| **Implementierung** | Einfacher | Komplexer |

---

## 13. Prüfungsfragen

1. **Warum wachsen B-Bäume nach oben, nicht nach unten?**
   - Wenn Wurzel überläuft, entsteht neue Wurzel (Höhe +1)

2. **Wie viele Speicherzugriffe für 1 Million Einträge mit m=8?**
   - h ≤ log₉(1.000.001/2) ≈ 6 → max 7 Zugriffe

3. **Wann wende ich Ausgleich, wann Merge an?**
   - Ausgleich: wenn Nachbarknoten > m Einträge hat
   - Merge: wenn beide Nachbarn genau m Einträge haben

4. **Was ist der Unterschied B*-Baum?**
   - Innere Knoten: nur Schlüssel (keine Inhalte)
   - Blätter: vollständige Einträge
   - Resultat: höherer Verzweigungsgrad, flacherer Baum

5. **Wie viele Ebenen spart B*-Baum in der Praxis?**
   - B-Baum: ~6 Ebenen für 1 Mio. Einträge
   - B*-Baum: ~3 Ebenen (50% Reduktion!)

