# neo4j_chord_graph
A knowledge graph for relating notes to intervals to scales to chords to appearances in songs

### Create I chord for each Major Scale
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
