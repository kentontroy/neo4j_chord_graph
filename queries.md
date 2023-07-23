### Get the cycle of fifths starting at C
```
MATCH (root:Note { name: "C" })
MATCH (root)-[:Perfect5th*0..12]->(n:Note)
RETURN n AS note

╒═══════════════════════════════════════════════╕
│note                                           │
╞═══════════════════════════════════════════════╡
│(:Semitone:C:Note {name: "C"})                 │
├───────────────────────────────────────────────┤
│(:Semitone:G:Note {name: "G"})                 │
├───────────────────────────────────────────────┤
│(:Semitone:D:Note {name: "D"})                 │
├───────────────────────────────────────────────┤
│(:Semitone:A:Note {name: "A"})                 │
├───────────────────────────────────────────────┤
│(:Semitone:E:Note {name: "E"})                 │
├───────────────────────────────────────────────┤
│(:Semitone:B:Note {name: "B"})                 │
├───────────────────────────────────────────────┤
│(:Semitone:F#:Note:Gb {name: "F#",alias: "Gb"})│
├───────────────────────────────────────────────┤
│(:Semitone:C#:Note:Db {name: "C#",alias: "Db"})│
├───────────────────────────────────────────────┤
│(:Semitone:G#:Note:Ab {name: "G#",alias: "Ab"})│
├───────────────────────────────────────────────┤
│(:Semitone:D#:Note:Eb {name: "D#",alias: "Eb"})│
├───────────────────────────────────────────────┤
│(:Semitone:A#:Note:Bb {name: "A#",alias: "Bb"})│
├───────────────────────────────────────────────┤
│(:Semitone:F:Note {name: "F"})                 │
├───────────────────────────────────────────────┤
│(:Semitone:C:Note {name: "C"})                 │
└───────────────────────────────────────────────┘
```
### Get a Cm7 Chord using flattened, enharmonic alieas
```
MATCH (root:C)-[:Minor3rd]->(third:Note) 
MATCH (third)-[:Minor3rd]->(fifth:Note) 
MATCH (fifth)-[:Major3rd]->(seventh:Note) 
RETURN coalesce(root.alias, root.name) AS root, 
       coalesce(third.alias, third.name) AS third,
       coalesce(fifth.alias, fifth.name) AS fifth,
       coalesce(seventh.alias, seventh.name) AS seventh

╒════╤═════╤═════╤═══════╕
│root│third│fifth│seventh│
╞════╪═════╪═════╪═══════╡
│"C" │"Eb" │"Gb" │"Bb"   │
└────┴─────┴─────┴───────┘
```
### Get a I, IV, V chord from Dmajor
```
MATCH (s:Dmajor)-[i:HAS_CHORD]->(c:Chord)
WHERE i.name IN ["I", "IV", "V"]
RETURN i.name, c 
ORDER BY i.name
```
### Get a I, IV, V chord by sampling a random key center
```
WITH (["C","C#","D","D#","E","F","F#","G", "G#","A", "A#","B"]) AS n
MATCH (s:MajorScale) WHERE s.root = n[toInteger(rand() * 12)] 
MATCH (s)-[i:HAS_CHORD]->(c:Chord)
WHERE i.name IN ["I", "IV", "V"]
RETURN s.root, i.name, c 
ORDER BY i.name
```
