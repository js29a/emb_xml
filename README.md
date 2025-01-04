# emb_xml

## the beginning

```rust
emb_xml::import_emb_xml!();

let doc = xml!{
    <root />
};

assert_eq!(doc.to_string(), "<root/>");
```

## with static attribute

```rust
emb_xml::import_emb_xml!();

let doc = xml!{
    <root attr="value" />
};

assert_eq!(doc.to_string(), "<root attr=\"value\"/>");
```

## with dynamic attribute

```rust
emb_xml::import_emb_xml!();

let doc = xml!{
    <root attr={ format!("{}", 42) } />
};

assert_eq!(doc.to_string(), "<root attr=\"42\"/>");
```

## with processing instruction

```rust
emb_xml::import_emb_xml!();

let doc = xml!{
    <?xml version="1.0"?>
    <root attr="value" />
};

assert_eq!(doc.to_string(), "<?xml version=\"1.0\"?><root attr=\"value\"/>");
```
