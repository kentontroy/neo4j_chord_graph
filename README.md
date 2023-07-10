# neo4j_chord_graph
A knowledge graph for relating notes to intervals to scales to chords to appearances in songs

### Create the twelve semitones
```
UNWIND (["C","C#","D","D#","E","F","F#","G", "G#","A", "A#","B"]) AS note
CALL apoc.merge.node(["Semitone", "Note", note], { name: note  }) YIELD node
RETURN node
```
### Create alias for flatted enharmonic tones
```
MERGE (s:Semitone:Note:`C#`)
SET s:Db, s.alias = "Db"

MERGE (s:Semitone:Note:`D#`)
SET s:Eb, s.alias = "Eb"

MERGE (s:Semitone:Note:`F#`)
SET s:Gb, s.alias = "Gb"

MERGE (s:Semitone:Note:`G#`)
SET s:Ab, s.alias = "Ab"

MERGE (s:Semitone:Note:`A#`)
SET s:Bb, s.alias = "Bb"
```
### Create a circularly linked list denoting a Minor2nd interval relationship between the consecutive semitones
```
UNWIND RANGE(0, 10) AS i
WITH i, (["C","C#","D","D#","E","F","F#","G", "G#","A", "A#","B"]) AS notes
    MATCH (s1:Semitone:Note { name: notes[i] }) 
    MATCH (s2:Semitone:Note { name: notes[i+1] }) 
    MERGE (s1)-[:Minor2nd]->(s2)

MATCH (s1:Semitone:Note:B)
MATCH (s2:Semitone:Note:C)
MERGE (s1)-[:Minor2nd]->(s2)
```
### Create more intervals to link the notes  - e.g. Major2nd, PerfectFifth, Minor7th, etc 
```
UNWIND (["C","C#","D","D#","E","F","F#","G", "G#","A", "A#","B"]) AS n
WITH n
    MATCH (s1:Semitone:Note)-[:Minor2nd*0]->(s2:Semitone)
    WHERE s1.name = n
    MERGE (s1)-[:Unison]->(s2)
    
    WITH s1 
    MATCH (s1)-[:Minor2nd*2]->(s2:Semitone)
    MERGE (s1)-[:Major2nd]->(s2)
    
    WITH s1 
    MATCH (s1)-[:Minor2nd*3]->(s2:Semitone)
    MERGE (s1)-[:Minor3rd]->(s2)
    
    WITH s1 
    MATCH (s1)-[:Minor2nd*4]->(s2:Semitone)
    MERGE (s1)-[:Major3rd]->(s2)

    WITH s1 
    MATCH (s1)-[:Minor2nd*5]->(s2:Semitone)
    MERGE (s1)-[:Perfect4th]->(s2)
    
    WITH s1 
    MATCH (s1)-[:Minor2nd*6]->(s2:Semitone)
    MERGE (s1)-[:Augmented4th]->(s2)

    WITH s1 
    MATCH (s1)-[:Minor2nd*7]->(s2:Semitone)
    MERGE (s1)-[:Perfect5th]->(s2)

    WITH s1 
    MATCH (s1)-[:Minor2nd*8]->(s2:Semitone)
    MERGE (s1)-[:Minor6th]->(s2)
    
    WITH s1 
    MATCH (s1)-[:Minor2nd*9]->(s2:Semitone)
    MERGE (s1)-[:Major6th]->(s2)

    WITH s1 
    MATCH (s1)-[:Minor2nd*10]->(s2:Semitone)
    MERGE (s1)-[:Minor7th]->(s2)
    
    WITH s1 
    MATCH (s1)-[:Minor2nd*11]->(s2:Semitone)
    MERGE (s1)-[:Major7th]->(s2)
```
### Create the Major Scale nodes
```
UNWIND (["C","C#","D","D#","E","F","F#","G", "G#","A", "A#","B"]) AS n
WITH n
    CALL apoc.merge.node(["Scale", "MajorScale", "Diatonic", "Heptatonic", (n + "major") ], 
        { name: (n + "major"), root: n }) YIELD node
RETURN node    

MATCH (s:`C#major`)
SET s:Dbmajor, s.alias = "Dbmajor"

MATCH (s:`D#major`)
SET s:Ebmajor, s.alias = "Ebmajor"

MATCH (s:`F#major`)
SET s:Gbmajor, s.alias = "Gbmajor"

MATCH (s:`G#major`)
SET s:Abmajor, s.alias = "Abmajor"

MATCH (s:`A#major`)
SET s:Bbmajor, s.alias = "Bbmajor"
```
### Build relationships between the Notes and the Major Scales
```
UNWIND (["C","C#","D","D#","E","F","F#","G", "G#","A", "A#","B"]) AS n
WITH n
    MATCH (t:Semitone:Note { name: n })
    MATCH (s:MajorScale { root: n })
    MERGE (s)-[:HAS_TONE { degree: 1, instance: "Tonic" }]->(t)
    WITH t, s
        MATCH (t)-[:Major2nd]->(next:Semitone)
        MERGE (s)-[:HAS_TONE { degree: 2, instance: "Supertonic" }]->(next)
    WITH t, s
        MATCH (t)-[:Major3rd]->(next:Semitone)
        MERGE (s)-[:HAS_TONE { degree: 3, instance: "Mediant" }]->(next)
    WITH t, s
        MATCH (t)-[:Perfect4th]->(next:Semitone)
        MERGE (s)-[:HAS_TONE { degree: 4, instance: "Subdominant" }]->(next)
    WITH t, s
        MATCH (t)-[:Perfect5th]->(next:Semitone)
        MERGE (s)-[:HAS_TONE { degree: 5, instance: "Dominant" }]->(next)
    WITH t, s
        MATCH (t)-[:Major6th]->(next:Semitone)
        MERGE (s)-[:HAS_TONE { degree: 6, instance: "Submediant" }]->(next)
    WITH t, s
        MATCH (t)-[:Major7th]->(next:Semitone)
        MERGE (s)-[:HAS_TONE { degree: 7, instance: "Leading Tone" }]->(next)
```

### Create the I Chord for each Major Scale
```
MATCH (majScale:MajorScale)
UNWIND (majScale) AS scale
WITH scale
  MATCH (n:Semitone { name: scale.root })
  MATCH (n)-[:Major3rd]->(third:Semitone)
  MATCH (n)-[:Perfect5th]->(fifth:Semitone)
  CALL apoc.merge.node(["Chord", "Triad", n.name], { name: n.name, notes: [n.name, third.name, fifth.name] }) YIELD node
  MERGE (node)-[:HAS_TONE { degree: 1 }]->(n)
  MERGE (node)-[:HAS_TONE { degree: 3 }]->(third)
  MERGE (node)-[:HAS_TONE { degree: 5 }]->(fifth)
  MERGE (scale)-[i:HAS_CHORD { degree: 1, name: "I" }]->(node)
```
### Create the ii Chord for each Major Scale
```
MATCH (majScale:MajorScale)
UNWIND (majScale) AS scale
WITH scale
  MATCH (scale)-[:HAS_TONE { degree: 2 }]->(n:Note) 
  MATCH (n)-[:Minor3rd]->(third:Semitone)
  MATCH (n)-[:Perfect5th]->(fifth:Semitone)
  CALL apoc.merge.node(["Chord", "Triad", (n.name + "m")], { name: (n.name + "m"), notes: [n.name, third.name, fifth.name] }) YIELD node
  MERGE (node)-[:HAS_TONE { degree: 1 }]->(n)
  MERGE (node)-[:HAS_TONE { degree: 3 }]->(third)
  MERGE (node)-[:HAS_TONE { degree: 5 }]->(fifth)
  MERGE (scale)-[i:HAS_CHORD { degree: 2, name: "ii" }]->(node)
```
### Create the iii Chord for each Major Scale
```
MATCH (majScale:MajorScale)
UNWIND (majScale) AS scale
WITH scale
  MATCH (scale)-[:HAS_TONE { degree: 3 }]->(n:Note) 
  MATCH (n)-[:Minor3rd]->(third:Semitone)
  MATCH (n)-[:Perfect5th]->(fifth:Semitone)
  CALL apoc.merge.node(["Chord", "Triad", (n.name + "m")], { name: (n.name + "m"), notes: [n.name, third.name, fifth.name] }) YIELD node
  MERGE (node)-[:HAS_TONE { degree: 1 }]->(n)
  MERGE (node)-[:HAS_TONE { degree: 3 }]->(third)
  MERGE (node)-[:HAS_TONE { degree: 5 }]->(fifth)
  MERGE (scale)-[i:HAS_CHORD { degree: 3, name: "iii" }]->(node)
```
### Create the IV Chord for each Major Scale
```
MATCH (majScale:MajorScale)
UNWIND (majScale) AS scale
WITH scale
  MATCH (scale)-[:HAS_TONE { degree: 4 }]->(n:Note) 
  MATCH (n)-[:Major3rd]->(third:Semitone)
  MATCH (n)-[:Perfect5th]->(fifth:Semitone)
  CALL apoc.merge.node(["Chord", "Triad", n.name], { name: n.name, notes: [n.name, third.name, fifth.name] }) YIELD node
  MERGE (node)-[:HAS_TONE { degree: 1 }]->(n)
  MERGE (node)-[:HAS_TONE { degree: 3 }]->(third)
  MERGE (node)-[:HAS_TONE { degree: 5 }]->(fifth)
  MERGE (scale)-[i:HAS_CHORD { degree: 4, name: "IV" }]->(node)
```
### Create the V Chord for each Major Scale
```
MATCH (majScale:MajorScale)
UNWIND (majScale) AS scale
WITH scale
  MATCH (scale)-[:HAS_TONE { degree: 5 }]->(n:Note) 
  MATCH (n)-[:Major3rd]->(third:Semitone)
  MATCH (n)-[:Perfect5th]->(fifth:Semitone)
  CALL apoc.merge.node(["Chord", "Triad", n.name], { name: n.name, notes: [n.name, third.name, fifth.name] }) YIELD node
  MERGE (node)-[:HAS_TONE { degree: 1 }]->(n)
  MERGE (node)-[:HAS_TONE { degree: 3 }]->(third)
  MERGE (node)-[:HAS_TONE { degree: 5 }]->(fifth)
  MERGE (scale)-[i:HAS_CHORD { degree: 5, name: "V" }]->(node)
```
### Create the vi Chord for each Major Scale
```
MATCH (majScale:MajorScale)
UNWIND (majScale) AS scale
WITH scale
  MATCH (scale)-[:HAS_TONE { degree: 6 }]->(n:Note) 
  MATCH (n)-[:Minor3rd]->(third:Semitone)
  MATCH (n)-[:Perfect5th]->(fifth:Semitone)
  CALL apoc.merge.node(["Chord", "Triad", (n.name + "m")], { name: (n.name + "m"), notes: [n.name, third.name, fifth.name] }) YIELD node
  MERGE (node)-[:HAS_TONE { degree: 1 }]->(n)
  MERGE (node)-[:HAS_TONE { degree: 3 }]->(third)
  MERGE (node)-[:HAS_TONE { degree: 5 }]->(fifth)
  MERGE (scale)-[i:HAS_CHORD { degree: 6, name: "vi" }]->(node)
```
### Create the vii(dim) Chord for each Major Scale
```
MATCH (majScale:MajorScale)
UNWIND (majScale) AS scale
WITH scale
  MATCH (scale)-[:HAS_TONE { degree: 7 }]->(n:Note) 
  MATCH (n)-[:Minor3rd]->(third:Semitone)
  MATCH (n)-[:Augmented4th]->(fifth:Semitone)
  CALL apoc.merge.node(["Chord", "Triad", (n.name + "dim")], { name: (n.name + "dim"), notes: [n.name, third.name, fifth.name] }) YIELD node
  MERGE (node)-[:HAS_TONE { degree: 1 }]->(n)
  MERGE (node)-[:HAS_TONE { degree: 3 }]->(third)
  MERGE (node)-[:HAS_TONE { degree: 5 }]->(fifth)
  MERGE (scale)-[i:HAS_CHORD { degree: 7, name: "vii(dim)" }]->(node)
```
### Create the Minor Scale nodes
```
UNWIND (["C","C#","D","D#","E","F","F#","G", "G#","A", "A#","B"]) AS n
WITH n
  CALL apoc.merge.node(["Scale", "MinorScale", "NaturalMinorScale", "Diatonic", "Heptatonic", (n + "minor") ], 
        { name: (n + "minor"), root: n }) YIELD node
RETURN node

MATCH (s:`C#minor`)
SET s:Dbminor, s.alias = "Dbminor"

MATCH (s:`D#minor`)
SET s:Ebminor, s.alias = "Ebminor"

MATCH (s:`F#minor`)
SET s:Gbminor, s.alias = "Gbminor"

MATCH (s:`G#minor`)
SET s:Abminor, s.alias = "Abminor"

MATCH (s:`A#minor`)
SET s:Bbminor, s.alias = "Bbminor"
```
### Build relationships between the Notes and the Minor Scales
```
UNWIND (["C","C#","D","D#","E","F","F#","G", "G#","A", "A#","B"]) AS n
WITH n
  MATCH (t:Semitone:Note { name: n })
  MATCH (s:NaturalMinorScale { root: n })
  MERGE (s)-[:HAS_TONE { degree: 1 }]->(t)
  WITH t, s
    MATCH (t)-[:Major2nd]->(next:Semitone)
    MERGE (s)-[:HAS_TONE { degree: 2 }]->(next)
  WITH t, s
    MATCH (t)-[:Minor3rd]->(next:Semitone)
    MERGE (s)-[:HAS_TONE { degree: 3 }]->(next)
  WITH t, s
    MATCH (t)-[:Perfect4th]->(next:Semitone)
    MERGE (s)-[:HAS_TONE { degree: 4 }]->(next)
  WITH t, s
    MATCH (t)-[:Perfect5th]->(next:Semitone)
    MERGE (s)-[:HAS_TONE { degree: 5 }]->(next)
  WITH t, s
    MATCH (t)-[:Minor6th]->(next:Semitone)
    MERGE (s)-[:HAS_TONE { degree: 6 }]->(next)
  WITH t, s
    MATCH (t)-[:Minor7th]->(next:Semitone)
    MERGE (s)-[:HAS_TONE { degree: 7 }]->(next)
```
### Relate the scales to each other via Relative Majors and Relative Minors
```
MATCH (majorScale:MajorScale)
WITH majorScale
  MATCH (n:Note { name: majorScale.root })-[:Major6th]->(relativeMinor:Note)
  MATCH (minorScale:NaturalMinorScale { root: relativeMinor.name })
  MERGE (minorScale)-[:IS_RELATIVE_MINOR_OF]->(majorScale)
  MERGE (majorScale)-[:IS_RELATIVE_MAJOR_OF]->(minorScale)
```
### Create the i Chord for each Minor Scale
```
MATCH (minScale:NaturalMinorScale)
UNWIND (minScale) AS scale
WITH scale
  MATCH (n:Semitone { name: scale.root })
  MATCH (n)-[:Minor3rd]->(third:Semitone)
  MATCH (n)-[:Perfect5th]->(fifth:Semitone)
  CALL apoc.merge.node(["Chord", "Triad", (n.name + "m")], { name: (n.name + "m"), notes: [n.name, third.name, fifth.name] }) YIELD node
  MERGE (node)-[:HAS_TONE { degree: 1 }]->(n)
  MERGE (node)-[:HAS_TONE { degree: 3 }]->(third)
  MERGE (node)-[:HAS_TONE { degree: 5 }]->(fifth)
  MERGE (scale)-[i:HAS_CHORD { degree: 1, name: "i" }]->(node)
RETURN scale, i, node AS chord, n AS root, third, fifth
```
### Create the ii(dim) Chord for each Minor Scale
```
MATCH (minScale:NaturalMinorScale)
UNWIND (minScale) AS scale
WITH scale
  MATCH (scale)-[:HAS_TONE { degree: 2 }]->(n:Note) 
  MATCH (n)-[:Minor3rd]->(third:Semitone)
  MATCH (n)-[:Augmented4th]->(fifth:Semitone)
  CALL apoc.merge.node(["Chord", "Triad", (n.name + "dim")], { name: (n.name + "dim"), notes: [n.name, third.name, fifth.name] }) YIELD node
  MERGE (node)-[:HAS_TONE { degree: 1 }]->(n)
  MERGE (node)-[:HAS_TONE { degree: 3 }]->(third)
  MERGE (node)-[:HAS_TONE { degree: 5 }]->(fifth)
  MERGE (scale)-[i:HAS_CHORD { degree: 2, name: "ii(dim)" }]->(node)
RETURN scale, i, node AS chord, n AS root, third, fifth
```
### Create the bIII Chord for each Minor Scale
```
MATCH (minScale:NaturalMinorScale)
UNWIND (minScale) AS scale
WITH scale
  MATCH (scale)-[:HAS_TONE { degree: 3 }]->(n:Note) 
  MATCH (n)-[:Major3rd]->(third:Semitone)
  MATCH (n)-[:Perfect5th]->(fifth:Semitone)
  CALL apoc.merge.node(["Chord", "Triad", n.name], { name: n.name, notes: [n.name, third.name, fifth.name] }) YIELD node
  MERGE (node)-[:HAS_TONE { degree: 1 }]->(n)
  MERGE (node)-[:HAS_TONE { degree: 3 }]->(third)
  MERGE (node)-[:HAS_TONE { degree: 5 }]->(fifth)
  MERGE (scale)-[i:HAS_CHORD { degree: 3, name: "bIII" }]->(node)
RETURN scale, i, node AS chord, n AS root, third, fifth
```
### Create the iv Chord for each Minor Scale
```
MATCH (minScale:NaturalMinorScale)
UNWIND (minScale) AS scale
WITH scale
  MATCH (scale)-[:HAS_TONE { degree: 4 }]->(n:Note) 
  MATCH (n)-[:Minor3rd]->(third:Semitone)
  MATCH (n)-[:Perfect5th]->(fifth:Semitone)
  CALL apoc.merge.node(["Chord", "Triad", (n.name + "m")], { name: (n.name + "m"), notes: [n.name, third.name, fifth.name] }) YIELD node
  MERGE (node)-[:HAS_TONE { degree: 1 }]->(n)
  MERGE (node)-[:HAS_TONE { degree: 3 }]->(third)
  MERGE (node)-[:HAS_TONE { degree: 5 }]->(fifth)
  MERGE (scale)-[i:HAS_CHORD { degree: 4, name: "iv" }]->(node)
RETURN scale, i, node AS chord, n AS root, third, fifth
```
### Create the v Chord for each Minor Scale
