# I2H2A Vocabulary

**URI:** https://ultraquamfy.github.io/I2H2A-spec/vocab.md  
**Status:** Draft  
**Version:** 0.1

This document defines the vocabulary terms used by the I2H2A credential type.

## Terms

### I2H2A
URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab.md#I2H2A`  
The I2H2A credential type identifier.

### delegatedBy
URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab.md#delegatedBy`  
The DID of the human holder who delegated authority to the agent.

### parentCredential
URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab.md#parentCredential`  
Reference to a parent I2H2A credential in an H2A2A chain. Null for V1 (H2A).

### delegationDepth
URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab.md#delegationDepth`  
Non-negative integer indicating delegation chain depth. Must be 0 for V1.

### scope
URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab.md#scope`  
Object defining the permitted operations for the delegated agent.

### mcpServers
URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab.md#mcpServers`  
Array of MCP server identifiers the agent is permitted to interact with.

### taskType
URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab.md#taskType`  
String identifying the class of task the agent is permitted to perform.

### constraints
URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab.md#constraints`  
Optional object carrying additional platform-defined constraints on agent behaviour.

### authorization
URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab.md#authorization`  
Opaque platform-specific payload. Generic I2H2A verifiers MUST NOT interpret its contents.
