# rdoc

### rdoc (Replicated DOCument): Build better decentralized and offline-first applications in Go

[![Build Status](https://travis-ci.org/gpestana/rdoc.svg?branch=master)](https://travis-ci.org/gpestana/rdoc) [![Package Version](https://img.shields.io/github/v/tag/gpestana/rdoc)](https://img.shields.io/github/v/tag/gpestana/rdoc)

[![Go Reference](https://pkg.go.dev/badge/github.com/gpestana/rdoc.svg)](https://pkg.go.dev/github.com/gpestana/rdoc)

rdoc is a native go implementation of a conflict-free replicated JSON data
structure, as introduced by Martin Kleppmann and Alastair R. Beresford in their
seminal work [[1]](https://arxiv.org/abs/1608.03960). A JSON CRDT is "[...] an
algorithm and formal semantics for a JSON data structure that automatically
resolves concurrent modifications such that no updates are lost, and such that
all replicas converge towards the same state (a conflict-free replicated
datatype or CRDT)."  [[1]](https://arxiv.org/abs/1608.03960).

Do you want to learn more about the JSON CRDT data type? [This youtube video](https://www.youtube.com/watch?v=TRvQzwDyVro) is a good introduction to the original paper [1] by Martin Kleppmann and Alastair R. Beresford.

## Features 

- Simple API; One API call allows the application logic to update and manage coverging JSON replicas in decentralized settings;  
- Supports JSON Patch notation as defined in [RFC6902](https://tools.ietf.org/html/rfc6902);
- Supports [cbor serialization](https://tools.ietf.org/html/rfc7049) [WIP; v1.1.0 milestone];

## Examples

```go

// starts a new replicated JSON document with an unique ID (in the context of the replicas sample)
doc := Init("doc_replica_1")

// updates the document state with a JSON patch operation:
patch := []byte(`{"op": "add", "path": "/", "value": "user"`)
err := doc.Apply(patch)
if err != nil {
    panic(err)
}

// update the document state with remote operations (i.e. operations executed by a remote replica); 
// remote operations will update the state of the document iif all its dependencies have been applied.  
remotePath := []byte(`[
 {"op": "add", "path": "/", "value": "user", "id":"1.380503024", "deps": [] },
 {"op": "add", "path": "/name", "value": "Jane", "id":"2.1", "deps": ["1.1"] },
 {"op": "add", "path": "/name", "value": "Jane", "id":"2.380503024", "deps": ["1.380503024"] }
]`)

err := doc.Apply(remotePath)
if err != nil {
    panic(err)
}

// Get Doc operations to send over the wire; these operations can be used by
// remote replicas to converge state with `doc1`
doc1Operations := doc.Operations()

// ... apply state from doc1 into doc2, in order for replicas `doc1` and `doc2` to converge

doc2.Apply(doc1Operations)

// ...

// Native Go marshaling/unmarshaling supported 
buffer, err := json.Marshal(*doc)
if err != nil {
    panic(err)
}
```

## References

1. [A Conflict-Free Replicated JSON Datatype](https://arxiv.org/abs/1608.03960) (Martin Kleppmann, Alastair R. Beresford)
