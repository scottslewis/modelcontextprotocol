# SEP-0000: Group Primitive

- **Status**: Draft 
- **Type**: Standards Track 
- **Created**: 2025-11-28
- **Author(s)**: Primitive Groups Working Group
- **Sponsor**: 
- **PR**: #{0000}

## Abstract

The original MCP core primitives are Tools, Prompts, and Resources (TPRs). These primitives allow MCP servers to communicate with clients about the server-side state of these entities, and provide properties (e.g. description, title, meta, etc) that client-side AI models, developers, tools, and users can use.

The MCP protocol does not currently provide for multiple TPRs to be organized into collections or **groups**. 

Collections of MCP entities have both server-side use cases (e.g. gateways for directing client access to primitives from multiple MCP servers, server-side organization for scaling, categories of primitives for abstraction, static and dynamic workflows with groupings of relevant TPRs, user-or-role-specific grouping of TPRs, etc}, and client-side use (e.g.grouping for model reasoning, choice, sequencing or orchestration; primitive discovery, organization, search and filtering, and presentation in UI or developer tooling, etc).

This proposal introduces a Group as a first class primitive in the schema, to allow Groups
and their associated primitives to be created on an MCP server and then communicated to MCP clients.

NOTE: This version of the proposal focuses on non-hierarchical groupings, simple collections of TPRs with no group hierarchy.  

Future enhancements likely will be needed for 

Group hierarchy and namespaces 

Json serialization efficiency

Filtering and search for Groups

Server Dynamics and Group Updates

Additional request/responses (e.g. listGroups)

Other enhancements required by the community

## Motivation

A Group primitive in the MCP schema+protocol would allow mcp server and client designers to communicate collections of MCP entities, and be able to communicate those structures and group/set/category-level meta-data (e.g. description, title) to clients without having to use custom types for each specific use-case or design situation that calls for collections/grouping.  

To address a variety of collection use cases without multiple duplicate protocol enhancements, it makes sense to introduce a single Group primitive that is able to provide collections for all the other primitives (e.g. collection of tools) or combined (a single group that includes tools, resources, and prompts).

Another advantage of having groups as a new primitive is that it allows the MCP server developer to provide descriptions of Groups for processing by models.  For example, a Group could be defined on an MCP server that has multiple tools, prompts, and resources that are relevant to a specific workflow, and provide a description for the group, in addition to any/all of the descriptions created for TPRs.  The Group.description can provide information about the collection of TPRs (e.g. a workflow), along with the descriptions of each of the TPRs.

### Use Cases

Placeholder for a use cases summary from the community [use cases discussion](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/1772) and primitive-grouping-wg working group summaries.

## Specification

### Json Schema

```json
"Group": {
 "properties": {
   "name": {
     "description": "Uniquely identifies Group for programmatic access. May be used as a display name or fallback if title isn't present. Must be unique within server-defined context.",
     "type": "string"
   },
   "parent": {
        "$ref": "#/defs/Group"
    },
   "title": {
     "description": "Intended for UI and end-user contexts — optimized to be human-readable and easily understood.",
     "type": "string"
   },
   "description": {
     "description": "A full, human-readable description of the group.",
     "type": "string"
   },
   "_meta": {
     "additionalProperties": {},
     "description": "See [General fields: `_meta`](/specification/2025-11-25/basic/index#meta) for notes on `_meta` usage.",
     "type": "object"
   },
   "annotations": {
     "$ref": "#/$defs/Annotation",
     "description": "Optional additional group information.\n\nDisplay name precedence order is: title, annotations.title, then name."
   },
   “icons": {
     "description": "Optional set of icons that can display in a user interface. See similar usage by Tools, Resources, Prompt",
     "items": {
       "$ref": "#/$defs/Icon"
     },
     "type": "array"
   }
 },
 "required": [
   "name"
 ],
 "type": "object"
}
```
### Group Properties


| Property Name| Type| Required|Notes|
| --- | --- | --- | --- |
| name					| string		| yes			| See Group.name Property below|
| parent				| Group			| no			| See Group.parent Property below|
| title					| string		| no			| Same as for other TPRs|
| description      		| string		| no			| Same as for other TPRs|
| annotations      		| Annotation	| no			| Same as for other TPRs|
| icons					| Icon			| no            | Same as for other TPRs|
| _meta					| object		| no			| Same as for other TPRs

### Group.name Property

Group.name is the only required property in the Group schema definition.

The Group.name property should be assumed to have the same syntax and uniqueness requirements
 as specified by the [Tool.name property](https://modelcontextprotocol.io/specification/2025-11-25/server/tools#tool-names). 
 
As Groups are intended to represent collections of primitives, the presence of the Group.parent property (see below), and the resulting hierarchy of Groups makes it possible for TPRGs be guaranteed unique via a 'fully-qualified' name (i.e. the Group hierarchy names combined via some separator with the TPR name). See  under Group.parent property below for examples.

Note: This allows (but does not require) the usage of namespaces as Group.names.  Whether to associate Group instances in the protocol with namespace (or to use Groups at all) is a design choice for the MCP server developer.
 
### Group.parent Property

The Group.parent property provides an optional reference to a hierarchical set of groups, where the top of the hierarchy is specified by Group.parent == null (or property not present).  

The recursive optional parent reference supports the creation of hierarchies of Groups of arbitrary depth. 

Since Group.names are to be unique within a given mcp server (Group.name property above),
the Group.parent reference implies a full parent<->child relationship...i.e. a given Group.parent reference
establishes a 1-1 Group.child relationship in the opposite (parent -> child) direction.

NOTE: The Group.parent property could be eliminated from this proposal. It was included here because the notion of grouping/collections are very often associated with hierarchy (and namespaces)...e.g. file systems (directories) or object-oriented class namespaces. Given the ubiquity and utility of hierarchical
structures, the Group.parent property seems to be a reasonable addition to this proposal.

An alternative approach could be to omit the Group.parent property in this proposal, and introduce the optional Group.parent property in a future/later proposal.

### Group Title, Description, Icon, Annotation, and Meta Properties

The Group description, title, and _meta properties have exactly the same purpose and definition as the same-named properties in other MCP primitive definitions so will not be
discussed further here.

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

The recursive definition of Group.parent, in combination with the has some important implications for runtime serialization to and from json.  See Serialization of Hierarchical Groups below for exploration of
these issues.

### Defining Non-Hierarchical Group Membership

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

With many Groups, and/or many TPRs in a given group, this naive copy-per-reference strategy (currently used by most
json serialization libraries) could result in many copies of the same Group meta-data being added to the stream multiple times, with much duplication of other Group data (title, description, _meta), and much wasted bandwidth.

#### Efficient Serialization of object references via Json Pointers

TBD.  See [experiments with integrating into python sdk](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/1567#discussioncomment-15555163).

### Design Alternatives for Collecting Tools, Prompts, and Resources into Groups

As per the section above, one way to collect TPRs into groups is by adding a list of groups (optional property named 'groups' to Tool, Prompt, Resource and other types) that
each TPR is in/contained by.  

Another alternative considered would be to add a new standard property to the _meta property for TPRs (e.g. 'x-mcp-groups'. The value would assumed to be of type list<Group>.  An advantage of such a use of _meta is that it would not require any schema change for the existing TPRs (i.e. no need for TPR.groups property).  

Such a use of _meta has disadvantages. One is that the typing information in the schema for the 
groups property (type list of Groups) is lost and so typing would have to be enforced by each sdk 
implementation rather than by specification.  

## Backward Compatibility

Adding a Group primitive type does not by itself create backward incompatibility. It would have broad
implications, however, for protocol efficiency and scalability (e.g. Scalable Serialization via Object
Graphs).

## Security Implications

The main security concern is that this SEP introduces a new primitive type, meaning MCP servers and clients will have to be correctly implemented to create Groups, add/remove TPRGs to them, serialize them to json, deserialized them from json via the language-specific sdk.

## Reference Implementation

Reference implementation (Java) exists in [a branch of the mcp-java-sdk](https://github.com/scottslewis/mcp-java-sdk/tree/groups). [Here is the Java + Jackson json  implementation of the McpSchema.Group type](https://github.com/scottslewis/mcp-java-sdk/blob/groups/mcp-core/src/main/java/io/modelcontextprotocol/spec/McpSchema.java#L1348).

There are also reference implementations of the Group primitive for [python](https://github.com/scottslewis/python-sdk/blob/groups/src/mcp/types.py#L215) and [javascript](https://github.com/scottslewis/typescript-sdk/blob/main/src/types.ts#L253).
