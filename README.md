# emb_xml

## about the project

The goal of this project is to allow dynamic XML structures to be embedded directly in Rust code using macros.

This crate is supposed to be as orthogonal as possible.

## some examples

### the beginning

```rust
// manual import is too complicated so i've decided to create a macro to help.
emb_xml::import_emb_xml!();

let doc = xml!{
    // Rust commments are allowed
    <root />
};

assert_eq!(doc.to_string(), "<root/>");
```

pretty formatted macro output - please keep in mind that real output is minified:
```xml
<root />
```

### with static attribute

```rust
emb_xml::import_emb_xml!();

let doc = xml!{
    <root attr="value<>" />
};

assert_eq!(doc.to_string(), "<root attr=\"value&lt;&gt;\"/>");
```

```xml
<root attr="value&lt;&gt;" />
```

### with dynamic attribute

```rust
emb_xml::import_emb_xml!();

let doc = xml!{
    <root { "attr" }={ format!("{}", 42) } />
};

assert_eq!(doc.to_string(), "<root attr=\"42\"/>");
```

```xml
<root attr="42" />
```

### with dynamic tag name

```rust
emb_xml::import_emb_xml!();

let doc = xml!{
    <{ "dynamic" } />
};

assert_eq!(doc.to_string(), "<dynamic/>");
```

```xml
<dynamic />
```

### with short closing tag

```rust
emb_xml::import_emb_xml!();

let doc = xml!{
    <tag><sub /></>
};

assert_eq!(doc.to_string(), "<tag><sub/></tag>");
```

```xml
<tag>
    <sub />
</tag>
```

### with processing instruction

```rust
emb_xml::import_emb_xml!();

let doc = xml!{
    <?xml version="1.0"?>
    // also ok:
    // <? { "xml" } { "version" } = "1.0" ?>
    {
        proc_instr!("foo", "bar") // macro version
    }
    <root attr="value" />
};

assert_eq!(doc.to_string(), "<?xml version=\"1.0\"?><?foo bar?><root attr=\"value\"/>");
```

```xml
<?xml version="1.0"?>
<?foo bar?>
<root attr="value" />
```

### with nested xml fragments

```rust
emb_xml::import_emb_xml!();

let frag = xml_frag!{
    <foo />
};

let over = xml_frag!{
    <bar>{ frag }</>
};

let doc = xml!{
    <root>{ over }</>
};

assert_eq!(doc.to_string(), "<root><bar><foo/></bar></root>");
```

```xml
<root>
    <bar>
        <foo />
    </bar>
</root>
```

### with vector of fragments

```rust
emb_xml::import_emb_xml!();

let cum = (0 .. 3).map(|k| {
    xml_frag!{
        <sub>{ k.to_string() }</sub>
    }
}).collect();

let sub = xml_frag!{
    <tag>{ cum }</tag>
};

let doc = xml!{
    {
        proc_instr!("xml", "version=\"1.0\"")
    }
    <root>{sub}</root>
};

assert_eq!(doc.to_string(), "<?xml version=\"1.0\"?><root><tag><sub>0</sub><sub>1</sub><sub>2</sub></tag></root>");
```

```xml
<?xml version="1.0"?>
<root>
    <tag>
        <sub>0</sub>
        <sub>1</sub>
        <sub>2</sub>
    </tag>
</root>
```

### with composite tags and attributes

```rust
emb_xml::import_emb_xml!();

let doc = xml!{
    <the:root:bździągwa the-test:żółć="test" />
};

assert_eq!(doc.to_string(), "<the:root:bździągwa the-test:żółć=\"test\"/>");
```

```xml
<the:root:bździągwa the-test:żółć="test" />
```

### with unescaped text

```rust
emb_xml::import_emb_xml!();

let doc = xml!{
    {
        raw_text!("<!DOCTYPE żółć>")
    }
    <root />
};

assert_eq!(doc.to_string(), "<!DOCTYPE żółć><root/>");
```

```xml
<!DOCTYPE żółć>
<root />
```

### with custom comment

```rust
emb_xml::import_emb_xml!();

let doc = xml!{
    { comment!("sth") }
    <root />
};

assert_eq!(doc.to_string(), "<!-- sth --><root/>");
```

```xml
<!-- sth -->
<root />
```

### with pretty output mode

```rust
emb_xml::import_emb_xml!();

let f1 = xml_frag!{
    <sub/>
};

let f2 = xml_frag!{
    <?my pi?>
    <sub/>
    {
        comment!("here")
    }
};

let doc = xml!{
    { raw_text!("żółć") }
    <?foo bar?>
    <root>
        <?alpha beta?>
        {
            "one"
        }
        <empty />
        {
            vec![
                f1,
                f2,
                comment!("foo")
            ]
        }
        <nested>
            <?deep pi?>
            <sth />
            {
                comment!("deep comment")
            }
        </nested>
        "two"
        {
            comment!("four")
        }
    </>
    {
        comment!("this is")
    }
    {
        comment!("the end")
    }
};

assert_eq!(doc.to_string_pretty(), r###"żółć
<?foo bar?>
<root>
    <?alpha beta?>
    one
    <empty />
    <sub />
    <?my pi?>
    <sub />
    <!-- here -->
    <!-- foo -->
    <nested>
        <?deep pi?>
        <sth />
        <!-- deep comment -->
    </nested>
    two
    <!-- four -->
</root>
<!-- this is -->
<!-- the end -->
"###);
```
