A library/module for escaping/unescaping of special HTML characters.

## 1. Announcement to mailing list

  - Proposed editor: _your name_
  - Date proposed: _date of proposal_
  - Link: _link to email_

###  Notes from discussion on mailing list

  - _note_
  - _note_
  - _note_

## 2. Research of standards and techniques

  1. Standard: _standard_
    - _link to docs_
    - ...
  2. Standard: _standard_
    - _link to docs_
    - ...
  1. Technique: _technique_
    - _link to docs_
    - ...
  2. Technique: _technique_
    - _link to docs_
    - ...

### Summary of research on standards and leading techniques
#### Relevant standards and techniques exist?
#### Those intended to follow (and why)
#### Those intended to ignore (and why)

## 3. Research of libraries from other languages

  1. Language: Go
    - [html](http://golang.org/pkg/html/)
      - `EscapeString()` escapes only the 5 characters `< > & ' "`
      - `UnescapeString()` unescapes more characters
  1. Language: PHP
    - [htmlspecialchars()](http://php.net/manual/en/function.htmlspecialchars.php)
      - escapes only the 5 characters `< > & ' "`
    - [htmlspecialchars_decode ()](http://www.php.net/manual/en/function.htmlspecialchars-decode.php)
      - decodes only the characters handled by htmlspecialchars()

### Summary of research from other languages:
#### Structures and functions commonly appearing
#### Variations on implementation seen
#### Pitfalls and hazards associated with each variant
#### Relationship to other libraries and/or abstract interfaces

## 4. Module writing

  - Pull request: _link to bug_

### Additional implementation notes

Question: where to get from the complete list of characters to escape and entities to produce?

- `escape_minimal()` only escapes the necessary 5 characters `< > & ' "` which are necessary for security/forms/URLs
- `escape_full()` escapes all characters
  - We probably should use a table-lookup (binary search), similar to the code in https://github.com/mozilla/rust/blob/incoming/src/libcore/unicode.rs