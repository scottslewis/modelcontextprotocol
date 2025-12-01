# SEP-0000: Group Primitive

- **Status**: Draft 
- **Type**: Standards Track 
- **Created**: 2025-11-28
- **Author(s)**: scottslewis@gmail.com
- **Sponsor**: 
- **PR**: #{0000}

## Abstract

The original MCP core primitives are Tools, Prompts, and Resources (TPRs). These primitives allow MCP servers to communicate with clients about the state of these entities, and provide properties (e.g. description, title, meta, etc) that AI models, developers, tools, and 
users can use.

The MCP protocol does not currently provide for multiple TPRs to be organized, collected, or
abstracted into collections/categories/features or **groups**. 

Such collections have both server-side use cases (e.g. gateways for multiple mcp servers, server-side organization for scaling,, categories of TPRs for TPR abstraction, static and dynamic workflows with groupings of TPRs, user-specific grouping of TPRs, etc}, and 
client-side use (e.g.grouping for model reasoning and choice, discovery, organization, filtering, and presentation in UI or developer tooling, etc).

This proposal introduces a Group as a first class primitive, to allow TPRGs to 
be dynamically created on an MCP server and then efficiently communicated to MCP clients.

## Motivation

A Group primitive in the mcp schema+protocol would allow mcp server and client designers to communicate collections of other mcp entities, and be able to communicate
those structures and group/set/category-level meta-data (e.g. description, title) to clients without having to use custom types/schema/json for each specific use-case or design situation.

### Use Cases

Placeholder for a use cases summary from the use cases discussion #1772 and primitive-grouping-wg working group summaries.

## Specification

###Json Schema for Group Primitive

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
```
The json schema description for each property has been omitted above for clarity.  [Here is the complete addition 
to the current draft/schema.json](https://github.com/scottslewis/modelcontextprotocol/blob/groups/schema/draft/schema.json#L3618)

### Group Properties


| Property Name			| Type			| Required		|Notes								|
|:---------------------:|:-------------:|:-------------:|:---------------------------------:|
| name					| string		| yes			| Unique identifier for the group|
| parent				| Group			| no			| See Group.parent section below|
| description      		| string		| no			| Same as for TPRs|
| title					| string		| no			| Optional human-readable name of the group for display purposes|
| _meta					| object		|no				|Same as for other TPRs


### Group.name property

The Group.name property should be assumed to have the same syntax and uniqueness requirements
 as specified by the [Tool.name property](https://modelcontextprotocol.io/specification/2025-11-25/server/tools#tool-names). Group.name is the only required property in the Group schema definition.
 
As Groups represent collections of Tools, Prompts, and/or Resources, the presence of the Group.parent property (see below), along with a hierarchy of Groups makes it possible for TPRGs be made unique via 'fully-qualified' name (i.e. the Group hierarchy names, combined with the TPR name).  See the
examples under Group.parent property below for examples.

Note: This allows but does not require the usage of pre-existing or new namespaces as Group.names.
 
### Group.parent property

NOTE: The Group.parent property could be eliminated from this proposal completely, or its
introduction could deferred to a subsequent proposal. It was included here because the notion
of grouping/collections are very often associated with hierarchy (and namespaces)...e.g.
file systems (directories) or class namespaces. An alternative approach could be to leave out the optional Group.parent property from the schema/specification until a later time. 

The Group.parent property provides an optional reference to a hierarchical set of groups, where 
the top of the Group hierarchy is specified by Group.parent == null or not present.  

The notion of a recursive optional parent reference supports the creation of trees of Groups of arbitrary
depth. Since Group.names are assumed to be unique withing a given mcp server (Group.name property above),
the Group.parent reference implies a full parent<->child relationship...i.e. a given Group.parent reference
implies a Group.child relationship.

### Remaining Group Properties

The description, title, and _meta properties have exactly the same purpose and definition as the 
existing TPRs.


```
Example 1

	{	
		name: "topgroup",
		title: "Animals",
		description:  "..."
	}

	{
		name: "group1",
		parent: "topgroup",
		title: "Mammals",
		description: "..."
	}

	Defines the hierarchy: topgroup -> group1
	
	Fully Qualified Names: topgroup, topgroup.group1

Example 2

	{
		name: "group2"
		parent: "group1"
		title: "Reptiles"
	}

	Defines the hierarchy: topgroup -> group1 -> group2
	
	Fully Qualified Names: topgroup, topgroup.group1, topgroup.group1.group2
	
Example 3

	{ 
		name: "group3"
		parent: "group1"
		title: "Cats"
	}
	
	Defines the hierarchy: topgroup -> group1 -> group2, group3
	
	Fully Qualified Names: topgroup, topgroup.group1, topgroup.group1.group2, topgroup.group1.group3
```

The recursive definition of Group.parent (of type Group)in combination with the has some important implications for
runtime serialization to json for inclusion in the request/response protocol.  See [Rationale(#Rationale) below for design alternatives considered.

### Associating Tools, Prompts, and Resources with Groups

There are multiple ways to associating Group instances with Tools, Resources, and Prompts (and
other MCP entities). See [Rationale(#Rationale) below for design alternatives.

One concise way to associate Primitives with Groups is to add an optional 'groups' properties to
the other primitives.  For example, adding an optional 'groups' property to the schema definition of Tool:

```json
	"Tool": {
// properties deleted for brevity
                "name": {
                    "type": "string"
                },
//               List of groups may be specified identifying the Groups that
//               this Tool is *in*
                "groups": {
                    "items": {
                        "$ref": "#/defs/Group"
                    },
                    "type": "array"
                }
                
   }

```
See [here](https://github.com/scottslewis/modelcontextprotocol/blob/groups/schema/draft/schema.json#L3743) for a complete version of Tool, with a new Tool.groups property.

A Tool.groups property allows a given Tool to be added to any number (list) of Groups.

Similar optional 'groups' properties can be [added to Prompt definition](https://github.com/scottslewis/modelcontextprotocol/blob/groups/schema/draft/schema.json#L3743) and [Resource definition](https://github.com/scottslewis/modelcontextprotocol/blob/groups/schema/draft/schema.json#L3743).

NOTE: The addition of an optional

## Rationale

### Group.name

Placeholder for summarized version of technical discussion from [here](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/1567#discussioncomment-15078444)

### Group.parent

Placeholder for summarized version of technical discussion  from [here](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/1567#discussioncomment-15091079)

### Serialization of Groups/trees

Placeholder for summarized version of technical discussion  from [here](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/1567#discussioncomment-15078444)

### Associating Primitives with Groups

Placeholder for summarized version of technical discussion  from [here](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/1567#discussioncomment-15091079)

## Backward Compatibility

The use of an optional 'groups' property for Tools, Resources, Prompts and other types allows the Group
schema to be introduced without requiring any changes to servers or clients.

However, adding the Group primitive type and ways to easily use it in existing protocol (e.g. listTools, listResources, listPrompts) could quickly require existing servers and clients to securely support it.

## Security Implications

The main security concern is that it does introduce a new Group primitive type, meaning servers and clients will have to properly deal with the type (probably via the language-specific sdk) in order to avoid
malicious server exploitation.

## Reference Implementation

Reference implementation (in Java) exists [here](https://github.com/scottslewis/mcp-java-sdk).  [Here](https://github.com/scottslewis/mcp-java-sdk/blob/groups/mcp-core/src/main/java/io/modelcontextprotocol/spec/McpSchema.java#L1348) is the Java + Jackson json serialization implementation of the Group type as specified above.