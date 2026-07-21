# XML — What It Is and How Polaris Uses It

*Part of: [[Summer Project Overview]] | [[ASD Config Overview]]*

---

## XML in One Sentence

XML = a way to store structured data as text, using nested tags with attributes.

---

## The Basics

```xml
<person name="Alice" age="30">
    <address city="Reykjavik" country="Iceland"/>
    <job title="Engineer" company="Tern"/>
</person>
```

Rules:
- **Tags** wrap content: `<tagname>...</tagname>`
- **Self-closing** if no children: `<tagname attr="val"/>`
- **Attributes** live inside the opening tag: `name="Alice"`
- Tags must be **properly nested** — no overlapping
- One **root** element contains everything

---

## Namespaces — The Confusing Part

In Polaris XML you'll see things like:

```xml
<instances xmlns:tlabel="http://tern.is/polaris-asd/tracklabel">
    <tlabel:tracklabel-line> ... </tlabel:tracklabel-line>
</instances>
```

The `tlabel:` prefix = a **namespace**. It's just a way to avoid naming conflicts when you mix schemas.

- `xmlns:tlabel="http://tern.is/..."` declares that `tlabel:` = this schema
- The URL is just an identifier — it doesn't have to be a real webpage
- **Read `tlabel:field` as just `field` in the tracklabel namespace**

Python analogy: `import numpy as np` — then you write `np.array()`. Same idea.

---

## The Polaris `<instances>` Pattern

Every Polaris config file looks like this:

```xml
<instances xmlns:tlabel="http://tern.is/polaris-asd/tracklabel">

    <instance id="polaris-asd-tracklabel"
              class-mapping-reference="TrackLabelConfig">

        <tlabel:configuration>
            <!-- actual config -->
        </tlabel:configuration>

    </instance>

</instances>
```

| Part                      | Meaning                                                           |
| ------------------------- | ----------------------------------------------------------------- |
| `<instances>`             | Root container, holds one or more instances                       |
| `<instance>`              | One configurable component                                        |
| `id`                      | Unique name (how Polaris finds this config block)                 |
| `class-mapping-reference` | Which C++ class in Polaris reads this config                      |
| `<tlabel:configuration>`  | The actual config content, validated by the tracklabel XSD schema |

---

## Property Substitution `${variable.name}`

Many values look like `${pasd.unit.colorscheme}` instead of a literal value.

```xml
<col:color name="emerald" color="#${pasd.colors.emerald}"/>
```

These are **placeholders** resolved from `.properties` files:

```properties
# unit-configuration-app.properties
pasd.unit.colorscheme=Isavia
pasd.colors.emerald=3CB371
```

At startup, Polaris replaces `${pasd.colors.emerald}` → `3CB371`.

**Why?** Same XML, different `.properties` file = different customer deployment. One file for Iceland, different colors/settings for Hungary.

**For my editor**: I need to either:
- Show the resolved value (read properties + substitute)
- Show the property key with the resolved value as a tooltip
- Let the user edit the property key or the resolved value

---

## Real Example — Track Label Field

```xml
<tlabel:field field-name="callsign"
              font-adjustment="+4"
              toggleable="false"
              visible-in-holding="true">

    <tlabel:context-menu-item
        context-menu="CallsignMenu.Correlated"
        mouse="left" />

    <tlabel:context-menu-item
        context-menu="CallsignMenu.Correlated"
        mouse="right"/>

</tlabel:field>
```

Reading this like Python:
```python
field = {
    "field_name": "callsign",      # what data to show
    "font_adjustment": +4,          # bigger text
    "toggleable": False,            # can't hide this field
    "visible_in_holding": True,
    "context_menus": [
        {"menu": "CallsignMenu.Correlated", "mouse": "left"},
        {"menu": "CallsignMenu.Correlated", "mouse": "right"},
    ]
}
```

---

## Real Example — Flight List Field

```xml
<tlist:field column="0" row="0"
             header="CALLSIGN"
             header-detailed="Callsign"
             display-role="callsign"
             font-size="22" font-size-xl="24"
             width="85" width-xl="95"
             column-width="85" column-width-xl="95">
    <tlist:context-menu-item
        context-menu="CallsignMenu.Correlated"
        mouse="right"/>
</tlist:field>
```

This = column 0 in the flight list, showing callsign data, 85px wide, font size 22.

---

## What My Editor Must Do With XML

1. **Parse** — read XML into a data structure (Qt has `QXmlStreamReader` / `QDomDocument`)
2. **Display** — show the config in a user-friendly GUI
3. **Modify** — user edits via GUI (not raw XML)
4. **Serialize** — write modified data back to valid XML
5. **Preserve** — don't destroy comments, `${property}` placeholders, or unknown attributes

Step 5 is tricky — most XML libraries don't preserve comments. This is a design challenge.

---

## Next Steps to Understand

- [[XSD Schema — What It Is]] — how Polaris *validates* its XML
- [[Qt XML Parsing]] — how C++ code reads this XML
- [[Track Label Editor]] — applying this knowledge to the first editor

---

*Related: [[ASD Config Overview]] | [[XSD Field Generation]] | [[Summer Project Overview]]*
