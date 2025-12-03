# SEP-0000: Group Primitive

- **Status**: Draft 
- **Type**: Standards Track 
- **Created**: 2025-11-28
- **Author(s)**: scottslewis@gmail.com
- **Sponsor**: 
- **PR**: #{0000}

## Abstract

The original MCP core primitives are Tools, Prompts, and Resources (TPRs). These primitives allow MCP servers to communicate with clients about the server-side state of these entities, and provide properties (e.g. description, title, meta, etc) that client-side AI models, developers, tools, and users can use.

The MCP protocol does not currently provide for multiple TPRs to be organized into collections or **groups**. 

Collections of mcp entities have both server-side use cases (e.g. gateways for directing client access to primitives from multiple mcp servers, server-side organization for scaling, categories of primitives for abstraction, static and dynamic workflows with groupings of relevant TPRs, user-or-role-specific grouping of TPRs, etc}, and client-side use (e.g.grouping for model reasoning, choice, sequencing or orchestration; primitive discovery, organization, search and filtering, and presentation in UI or developer tooling, etc).

This proposal introduces a Group as a first class primitive in the schema, to allow Groups
and their associated primitives to be created on an MCP server and then efficiently communicated to MCP clients.

## Motivation

A Group primitive in the MCP schema+protocol would allow mcp server and client designers to communicate collections of MCP entities, and be able to communicate those structures and group/set/category-level meta-data (e.g. description, title) to clients without having to use custom types for each specific use-case or design situation that called for collections.

### Use Cases

Placeholder for a use cases summary from the use cases discussion #1772 and primitive-grouping-wg working group summaries.

## Specification

### Group Primitive Json Schema

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
The schema description for each property has been omitted above for clarity. [Here is the complete addition to the current draft/schema.json](https://github.com/scottslewis/modelcontextprotocol/blob/groups/schema/draft/schema.json#L3618)

### Group Properties


| Property Name| Type| Required|Notes|
| --- | --- | --- | --- |
| name					| string		| yes			| See Group.name Property below|
| parent				| Group			| no			| See Group.parent Property below|
| description      		| string		| no			| Same as for TPRs|
| title					| string		| no			| Optional human-readable name of the group for display purposes|
| _meta					| object		|no				|Same as for other TPRs


### Group.name Property

The Group.name property is assumed to have the same syntax and uniqueness requirements
 as specified by the [Tool.name property](https://modelcontextprotocol.io/specification/2025-11-25/server/tools#tool-names). Group.name is the only required property in the Group schema definition.
 
As Groups are intended to represent collections of primitives, the presence of the Group.parent property (see below), and the resulting hierarchy of Groups makes it possible for TPRGs be guaranteed unique via a 'fully-qualified' name (i.e. the Group hierarchy names combined via some separator with the TPR name). See the examples under Group.parent property below for examples.

Note: This allows (but does not require) the usage of pre-existing namespaces as Group.names.  Whether to associate Group instances in the protocol with namespace is a 
design decision for the MCP server/TPR developer.
 
### Group.parent Property

The Group.parent property provides an optional reference to a hierarchical set of groups, where the top of the hierarchy is specified by Group.parent == null (or property not present).  

The recursive optional parent reference supports the creation of trees of Groups of arbitrary depth. 

Since Group.names are to be unique within a given mcp server (Group.name property above),
the Group.parent reference implies a full parent<->child relationship...i.e. a given Group.parent reference
implies a 1-1 Group.child relationship in the opposite (parent -> child) direction.

NOTE: The Group.parent property could be eliminated from this proposal. It was included here because the notion of grouping/collections are very often associated with hierarchy (and namespaces)...e.g. file systems (directories) or object-oriented class namespaces. Supporting (via the Group.parent property) a hierarchical organization of Groups seems 
a reasonable addition to this proposal from the outset.

An alternative approach could be to omit the Group.parent property in this proposal, and introduce the optional Group.parent property in a future/later proposal.

### Group Title, Description, and Meta Properties

The Group description, title, and _meta properties have exactly the same purpose and definition as the same-named properties in other MCP schema definitions.

### Example: Hierarchical Groups

Here are some simple example serialized-to-json Group instances to show how hierarchies of Groups can be easily represented.

```
Example 1

	{	
		name: "topgroup",
		title: "Animals",
		description:  "All types of animals are in this group. Plants, however, are not in this group"
	}

	{
		name: "group1",
		parent: "topgroup",
		title: "Mammals",
		description: "This group has all mammal species as part of this group"
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

The recursive definition of Group.parent, in combination with the has some important implications for runtime serialization to json.  See Serialization of Hierarchical Groups below for design alternatives considered.

### Associating Tools, Prompts, and Resources into Groups

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
See [here for a complete version of Tool](https://github.com/scottslewis/modelcontextprotocol/blob/groups/schema/draft/schema.json#L3743), with the Tool.groups property.

A Tool.groups property allows a given Tool to be added to any number of Groups.

```txt
Example adding myToolName tool to group3 from example defined above:

tool1 = Tool(name = "mytoolName", groups = [ group3 ])
tool2 = Tool(name = "secondToolName", groups = [ group3 ])

// serialize tool1 and tool2 instances for listTools() response

```

Additional 'groups' properties could be similarly [added to the Prompt definition](https://github.com/scottslewis/modelcontextprotocol/blob/groups/schema/draft/schema.json#L3743) and [the Resource definition](https://github.com/scottslewis/modelcontextprotocol/blob/groups/schema/draft/schema.json#L3743).   Grouping of additional entities in the MCP protocol (e.g. Role, Task, etc) could be added now or in future by adding a 'groups' field to any entity in the specification.

## Rationale

### Group.name Design Alternatives Considered

As a first class primitive, it seemed most appropriate to have the same uniqueness requirements for Group.name as any of the other primitives.

An alternative considered was that an 'id' property could be added to the Group schema definition to represent a unique identifier for the Group instance.

### Group.parent Design Alternatives Considered

Omit: One alternative would be to omit the Group.parent property entirely from the definition. This would mean that only a single level of grouping would be available (no hierarchy of groups). 

Defer: A second alternative would be introduce the Group primitive without the parent
property, and plan to add the parent property in a future SEP. 

### Serialization of Hierarchical Groups

One consequence of having the Group.parent property defined recursively in the schema, is that at serialization time (e.g. in response to a listTools() client request), the entire tree (connected object graph) would be serialized.  

```text
For example:

Tool("myToolName") -> Group("group3") -> Group("group1") -> Group("topgroup")
Tool("secondToolName") -> Group("group3") -> Group("group1") -> Group("topgroup")
```
serialization (generating json) for myToolName would (for most json libraries) result in myToolNaame in json form, as well as group3, group1, and topgroup.

If other tools were included, and they also referenced group2, group1 or topgroup, then 
for most json generation libraries (e.g. pydantic - Python, Jackson - Python, JSON lib -> javascript) a **separate copy** of each instance is added to the json stream. 

With many Groups, and/or many TPRs in a given group, this copy-per-reference strategy could result in many copies of the same Group being added to the stream multiple times, with much duplication of other Group data (title, description, _meta), and much wasted bandwidth.

#### Scalable Serialization via Object Graphs

One way to prevent this problem is to serialize the first Group instance in the stream normally, at serialization time assign that instance a unique "@id" field value, and then use the "@id" value for all subsequent serializations of that object in the stream. 

The Jackson library (popular enterprise Java libary) has an implementation of this strategy called [@JsonIdentityInfo](https://www.javadoc.io/static/com.fasterxml.jackson.core/jackson-annotations/2.8.0/com/fasterxml/jackson/annotation/JsonIdentityInfo.html). @JsonIdentityInfo is currently
being used in the [Java reference impl here](https://github.com/scottslewis/mcp-java-sdk/blob/groups/mcp-core/src/main/java/io/modelcontextprotocol/spec/McpSchema.java#L1347).

There is
also an open source multi-language implementation of this strategy called [JavaScript Object Graph](https://github.com/jsog/jsog).  

Given these implementations, it will be straightforward to introduce a language-interoperable approach for each compliant MCP sdk that efficiently serializes/deserializes the object graph of Group instances.

## Backward Compatibility

Adding a Group primitive type does not by itself create backward incompatibility. It would have broad
implications, however, for protocol efficiency and scalability (e.g. Scalable Serialization via Object
Graphs).


## Security Implications

The main security concern is that this SEP introduces a new primitive type, meaning MCP servers and clients will have to be correctly implemented to create Groups, add/remove TPRGs to them, serialize them to json, deserialized them from json via the language-specific sdk.

## Reference Implementation

Reference implementation (Java) exists in [a branch of the mcp-java-sdk](https://github.com/scottslewis/mcp-java-sdk/tree/groups). [Here is the Java + Jackson json  implementation of the McpSchema.Group type](https://github.com/scottslewis/mcp-java-sdk/blob/groups/mcp-core/src/main/java/io/modelcontextprotocol/spec/McpSchema.java#L1348).
