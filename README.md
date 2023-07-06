# neo4j_chord_graph
A knowledge graph for relating notes to intervals to scales to chords to appearances in songs

### Create the twelve semitones
```
UNWIND (["C","C#","D","D#","E","F","F#","G", "G#","A", "A#","B"]) AS note
CALL apoc.merge.node(["Semitone", "Note", note], { name: note  }) YIELD node
RETURN node

MATCH (n:Semitone:Note:`C#`)
SET n:Db, n.alias = "Db"

MATCH (n:Semitone:Note:`D#`)
SET n:Eb, n.alias = "Eb"

MATCH (n:Semitone:Note:`F#`)
SET n:Gb, n.alias = "Gb"

MATCH (n:Semitone:Note:`G#`)
SET n:Ab, n.alias = "Ab"

MATCH (n:Semitone:Note:`A#`)
SET n:Bb, n.alias = "Bb"
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

### Create the I chord for each Major Scale
```
MATCH (majScale:MajorScale)
UNWIND (majScale.name) AS scale
WITH scale
  MATCH (s:MajorScale { name: scale })-[i:HAS_TONE { degree: 1 }]->(n:Semitone)
  MATCH (n)-[:Major3rd]->(third:Semitone)
  MATCH (n)-[:Perfect5th]->(fifth:Semitone)
  CALL apoc.merge.node(["Chord", "Triad", n.name], { name: n.name, notes: [n.name, third.name, fifth.name] }) YIELD node
  MERGE (node)-[:HAS_TONE { degree: 1 }]->(n)
  MERGE (node)-[:HAS_TONE { degree: 3 }]->(third)
  MERGE (node)-[:HAS_TONE { degree: 5 }]->(fifth)
RETURN node AS chord, n AS root, third, fifth
```
