

## Overview

Data serialization is the process of converting data structures or objects into a format that can be stored or transmitted across a network and reconstructed later. This allows different applications (often written in different languages) to exchange data.

## JSON (JavaScript Object Notation)

JSON is the most common format used in modern REST APIs. It is human-readable and easy for machines to parse.

### Key Characteristics

- Data is in **name/value pairs** (e.g., `"name": "value"`).
    
- Objects are enclosed in **curly braces** `{ }`.
    
- Arrays (lists) are enclosed in **square brackets** `[ ]`.
    
- Name/value pairs are separated by **commas**.
    
- Names must be enclosed in **double quotes**.
    

### Data Types

- **String:** "GigabitEthernet1"
    
- **Number:** 100 or 10.5
    
- **Boolean:** true or false
    
- **Null:** null
    
- **Object:** { }
    
- **Array:** [ ]
    

## XML (Extensible Markup Language)

XML is a tag-based language similar to HTML but designed to carry data, not display it.

### Key Characteristics

- Uses **tags** to define data (e.g., `<name>value</name>`).
    
- Tags must be **balanced** (every opening tag needs a closing tag).
    
- Tags are **case-sensitive**.
    
- XML must have a single **root element** that contains all other elements.
    

### Example Structure

```
<interface>
    <name>GigabitEthernet1</name>
    <status>up</status>
</interface>
```

## YAML (YAML Ain't Markup Language)

YAML is a human-friendly data serialization language often used for configuration files (like Ansible).

### Key Characteristics

- Relies on **indentation** to show structure (do not use tabs, use spaces).
    
- Data is represented in **key-value pairs** (e.g., `name: value`).
    
- Lists are created using a **dash** `-`.
    
- Very minimal syntax (no braces or tags required).
    

### Example Structure

```
interface:
  name: GigabitEthernet1
  status: up
  vlans:
    - 10
    - 20
```

## Comparison Table

|Feature|JSON|XML|YAML|
|---|---|---|---|
|**Readability**|Good|Moderate|Excellent|
|**Structure**|Braces/Brackets|Tags|Indentation|
|**Common Use**|REST APIs|Older APIs|Config Files (Ansible)|
|**Metadata**|No|Yes (Attributes)|No|

## Configuration Management Tools

The slides briefly mention which tools prefer which formats:

- **Ansible:** Uses YAML.
    
- **SaltStack:** Uses YAML.
    
- **Puppet:** Uses its own DSL (Declarative State Language), but can use JSON.
    
- **Chef:** Uses Ruby-based DSL.
    

## Verification for CCNA

- Be able to identify a syntax error in a JSON snippet (e.g., a missing comma or quote).
    
- Recognize which format is being shown based on the use of tags (XML), braces (JSON), or indentation (YAML).
    

## Related Notes

- [[REST APIs]]
    
- [[Network Automation]]
    
- [[Ansible]]