# neo4j_chord_graph
A knowledge graph for relating notes to intervals to scales to chords to appearances in songs

### Create the twelve semitones
```
UNWIND (["C","C#","D","D#","E","F","F#","G", "G#","A", "A#","B"]) AS note
CALL apoc.merge.node(["Semitone", "Note", note], { name: note  }) YIELD node
RETURN node

MERGE (:Semitone:Note:`C#`:Db { alias: "Db" })
MERGE (:Semitone:Note:`D#`:Eb { alias: "Eb" })
MERGE (:Semitone:Note:`F#`:Gb { alias: "Gb" })
MERGE (:Semitone:Note:`G#`:Ab { alias: "Ab" })
MERGE (:Semitone:Note:`A#`:Bb { alias: "Bb" })
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
RETURN s, t
```

### Create the I chord for each Major Scale
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
RETURN scale, i, node AS chord, n AS root, third, fifth
```
### Create the ii chord for each Major Scale
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
RETURN scale, i, node AS chord, n AS root, third, fifth
```
### Create the iii chord for each Major Scale
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
RETURN scale, i, node AS chord, n AS root, third, fifth
```
### Create the IV chord for each Major Scale
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
RETURN scale, i, node AS chord, n AS root, third, fifth
```
### Create the V chord for each Major Scale
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
RETURN scale, i, node AS chord, n AS root, third, fifth
```
### Create the vi chord for each Major Scale
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
RETURN scale, i, node AS chord, n AS root, third, fifth
```
