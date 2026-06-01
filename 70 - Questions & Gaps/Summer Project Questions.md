# Summer Project — Open Questions

*Active question backlog for the summer project*

---

## Config Structure Questions

- [ ] Where are the XSD schema files for tracklabel / flightlist? (needed for Phase 3.2)
- [ ] Full list of valid `field-name` values for track labels — is there a reference doc or enum in source?
- [ ] Full list of valid `display-role` values for flight lists?
- [ ] What are all valid `tracklabel-type` values (beyond Uncorrelated/Correlated)?
- [ ] How does `${property.name}` substitution work exactly — is it done by Polaris at startup, or by the editor?
- [ ] What is `class-mapping-reference`? Is it a Java/C++ class name? Where is the mapping defined?

## Editor Architecture Questions

- [ ] How does the existing **Aeronautical Data Editor** handle schema→widget mapping? (reference impl)
- [ ] How does the existing **Map Layer Editor** handle XML import/export? (reuse this)
- [ ] What is the structure of the `polaris-asd-editor` repo? (folders, build system)
- [ ] What build system does the editor use? (CMake? qmake?)

## Technology Questions

- [ ] What version of Qt is used?
- [ ] What C++ standard?
- [ ] Is there a dev environment setup guide?
- [ ] How do I run the existing editors locally?

## Domain Questions

- [ ] What does a real ATCO workflow look like when they need to change track label config? (who changes it, how often, what's the pain?)
- [ ] What is the `-app` vs `-fds` distinction exactly in the properties files?

---

*When answered: create a note, link it here, strike out the question.*
