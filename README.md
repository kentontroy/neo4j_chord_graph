# neo4j_chord_graph
A knowledge graph for relating notes to intervals to scales to chords to appearances in songs

### Create the twelve semitones
```
UNWIND (["C","C#","D","D#","E","F","F#","G", "G#","A", "A#","B"]) AS note
CALL apoc.merge.node(["Semitone", "Note", note], { name: note  }) YIELD node
RETURN node

MATCH (n:Semitone:Note:`C#`)
SET n:Db, n.name = "`C#`", n.alias = "Db"
MATCH (n:Semitone:Note:`D#`)
SET n:Eb, n.name = "`D#`", n.alias = "Eb"
MATCH (n:Semitone:Note:`F#`)
SET n:Gb, n.name = "`F#`", n.alias = "Gb"
MATCH (n:Semitone:Note:`G#`)
SET n:Ab, n.name = "`G#`", n.alias = "Ab"
MATCH (n:Semitone:Note:`A#`)
SET n:Bb, n.name = "`A#`", n.alias = "Bb"
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
