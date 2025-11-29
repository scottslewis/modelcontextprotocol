# SEP-{0000}: {Grouping Primitive}

- **Status**: Draft 
- **Type**: Standards Track 
- **Created**: 2025-11-28
- **Author(s)**: scottslewis@gmail.com
- **Sponsor**: 
- **PR**: #{0000}

## Abstract

The current MCP core primitives are Tools, Prompts, and Resources (TPRs). These primitives allow MCP
servers to communicate with clients about these entities, and provide meta-data 
(e.g. description, title, meta, etc) that models, developers, tools, and users can use.

There is currently no way in the MCP protocol for multiple TPRs to be grouped (aka categories,
packages, features sets, etc).  Grouping has both server-side use cases (e.g. gateways to multiple mcp servers,
organization for scaling to many TPRs, categories of TPRs for abstraction, representing workflows
as groupings of TPRs, user-specifica organizations of TPRs, etc}, and client-side use (e.g.grouping for model 
reasoning and choice, discovery, organization, and presentation in UI or developer tooling, etc).

This proposal introduces a Group as a first class primitive, to allow TPRGs to be grouped on an MCP server and
communicated to MCP clients without a need for private protocol.

## Motivation

The current protocol/specification has no support for organizing or grouping existing 
primitives (Tool, Resource, Prompt) for either server-side usage (e.g. organizing, managing, scaling a 
large number of primitives) or client-side usage (e.g. description meta-data about sets of primitives, or
dev/tool/UI discovery, organization, and/or presentation).

A Group primitive in the mcp schema+protocol would allow mcp server and client designers to communicate
organizations and use-case driven sets/categories of other mcp entities, and be able to communicate
those structures and group/set/category-level meta-data (e.g. description, title) to clients without
having to use custom types/schema/json for each design situation.

## Specification

This is the proposed schema addition for a new primitive named Group

```json
"Group": {
    "properties": {
         "_meta": {
            "additionalProperties": {},
            "type": "object"
        },
        "description": {
            "type": "string"
        },
        "name": {
            "type": "string"
        },
        "parent": {
            "$ref": "#/defs/Group"
        },
        "title": {
            "type": "string"
        }
    },
    "required": [
        "name"
    ],
    "type": "object"
}
``
Some of the documentation has been removed for clarity.  [Here is the complete addition 
to the current draft/schema.json](https://github.com/scottslewis/modelcontextprotocol/blob/groups/schema/draft/schema.json#L3618)

The declared Group properties are 

```
	Name		Type		Notes
	
	name		string		See specification details
	parent 		Group		See specification details
	description	string		Same declaration as other primitives
	title		string		Same declaration as other primitives
	_meta		object 		Same declaration as other primitives
```

## Rationale

Design decisions made and alternatives that were considered.

## Backward Compatibility

How does this affect existing implementations? (Required for incompatible changes)

## Security Implications

Any security concerns related to this proposal.

## Reference Implementation

Link to or description of a reference implementation (required before Final status).