---
title: API Reference

language_tabs:
- python: Python(Native)

toc_footers:

search: true
---

# Introduction

>For the purposes of this documentation, we assume you have imported PsynthDB’s functionality as such:

```python
import PsynthDB as psynth
```

Welcome to PsynthDB, a fast embeddable graph database with all the features you’ve been wanting.  PsynthDB has the powerful [Boost](http://www.boost.org) package for C++ at it's core, and most of it's heavier functions are run this way to maximize speed.  With PsynthDB you can interact with graphs in a comfortable object-oriented fashion as opposed to having to learn a complex new query language.  It is written with human legibility in mind.

PsynthDB is currently available natively in Python, but as the REST API is finalized, we will build language bindings for Python, Javascript( and node.js), Ruby, Swift, and Java! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.  Tabs will be added for new languages as they become available.

# Graph

>Load an existing graph:

```python
g = psynth.Graph({
    'load_graph': 'filename.gt'
})
```

>Create a new Graph:

```python
#  Pass a dictionary of parameters to the constructor

g = psynth.Graph({
    'name': 'Test Graph',
    'owner': 'me'
})
```

>or

```python
#  Pass a dictionary of parameters to the constructor

g = psynth.Graph({
    'name': 'Test Graph',
    'filename': 'testgraph.gt',
    'owner': 'me'
})
```

  

This is the main Graph class.  It stores Nodes and Links, and contains a variety of analytic functions.  Most of your interactions with the graph model will be through this class.  It is strongly recommended to use the *add_node*, *add_link*, and *add_link_type* methods rather than accessing the Node, Link, and LinkType constructors directly.
 
Graphs can be constructed with a number of parameters, most of which are optional.  

If the “load_graph” parameter is used, no other parameters are required.

Paramaters

Required | Parameter | Description
--------- | ----------- | --------
*optional* | load_graph | A string of the filename, ending in .gt


or

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
required | owner | The owner of this graph |
*optional* | filename | A string of the filename, ending in .gt | UUID ending in .gt
*optional* | name | The name of your graph | “Unnamed Map”
*optional* | time_created | An object containing the date | time of construction
*optional* | users | An object keyed with users, and valued with permissions | {empty}
*optional* | link_types | An array of LinkType objects | [empty]
*optional* | history | An array of transactions performed on the graph. | [empty]
*optional* | backups | An array of paths where this Graph should be saved redundantly | []

## Graph.filename

>Retrieve the filename from the graph

```python
print g.filename
```

>Set a new filename for the graph

```python
g.filename = 'newgraphfile.gt'
```

The *filename* property must be a string ending in .gt, and supports ASCII characters.

## Graph.save

>Save the Graph to disk.

```python
g.save()
```

This writes the Graph to disk, as *Graph.filename*.  It also saves a file by the same name to every location given in the *backups* parameter during Graph Construction.

## Graph.save_image

>Save an SFDP layout as an image, heatmapped by betweeness values:

```python
g.draw.sfdp({
    'set_position': True
})

nb, lb = g.centrality.betweenness()

g.save_image({
    'filename': 'TestMap_sfdp_betweenness.png',
    'node_alpha': nb.normalized(),
    'link_alpha': lb.normalized()
})
```

>Save an ARF layout as an image, filtered for the Minimum Spanning Tree, heatmapped by Pagerank:

```python
g.draw.arf({
    'set_position': True
})

min_tree = g.topology.min_spanning_tree({
    'root': g.nodes[0]
})

pr = g.centrality.pagerank()

g.save_image({
    'filename': 'TestMap_arf_pagerank_tree.png',
    'node_alpha': pr.normalized(),
    'link_alpha': min_tree
})
```

This saves an image of the Graph.  The image will be a transparent .png. Nodes will be the shape and color they are set to.  Links will be colored by their LinkType *color*.  Nodes and Links will be labeled according to their *label*. Links will show directedness, and their thickness will be according to normalized weights by their LinkType.

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
required | filename | The filename to save this image as. | 
*optional* | width | Width of the image in px | sqrt(number_of_nodes) * 400
*optional* | height | Height of the image in px | sqrt(number_of_nodes) * 400

Nodes can be styled in a few ways, in addition to their intrinsic properties like radius, label, and shape.

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
*optional* | node_alpha | A NodeDataMap with values between 0 and 1 to set the opacity of Nodes. Boolean also acceptable. | 1.0 for all Nodes.
*optional* | node_label_alpha | A NodeDataMap with values between 0 and 1 to set the opacity of Node Labels. Boolean also acceptable. | Same as Node Opacity.
*optional* | node_radius | A NodeDataMap with values to uses for Node Radius.  A normalized map can be used along with the *max_node_radius* property to scale Node radius to a set of values automatically. | None
*optional* | max_node_radius | Can be used along with *node_radius* to scale Node Radius with some value. | None

Links can be styled in a few ways, in addition to their intrinsic properties like value, label, directedness, and LinkType.

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
*optional* | link_alpha | A LinkDataMap with values between 0 and 1 to set the opacity of Links.  Boolean also acceptable. | 0.7 for all Links.
*optional* | link_label_alpha | A LinkDataMap with values between 0 and 1 to set the opacity of Links.  Boolean also acceptable. | Same as Link Opacity.
*optional* | link_thickness | A LinkDataMap with values to uses for Link Thickness.  A normalized map can be used along with the *max_link_thickness* property to scale Link Thickness to a set of values automatically. | None
*optional* | max_link_thickness | Can be used along with *link_thickness* to scale Link Thickness with some value. | None


## Graph.name

>Retrieve the name from the graph

```python
print g.name
```

>Set a new name for the graph

```python
g.name = 'My Really Neat Graph'
```

The *name* property must be a string, and supports Unicode characters.

## Graph.owner

>Retrieve the owner from the graph

```python
o = g.owner
```

>Set a new owner for the graph

```python
g.owner = 'Sue Graphly'
```

>Owners can actually be objects

```python
g.owner = {
    'firstname': 'Sue',
    'lastname': 'Graphly',
    'email': 'sue.graphly@psymphonic.com'
}
```

The *owner* property is an object.  This is to accomodate different user management schemes flexibly. You can use whatever type of User object you'd like, including just a simple string.

## Graph.users

>Retrieve users from the graph

```python
for user in g.users:
    print user
    permission = g.users[user]
```

>Set permissions for a user in the graph

```python
g.users[user1] = 'administrator'

g.users[user2] = 'editor'

```

The *users* property is an object, with arbitary User objects as keys, and permissions as values.  The User object can be a simple string, or any User object you feel like using. This is to accomodate different user management schemes flexibly.

## Graph.time_created

>Retrieve the time the Graph was created.

```python
#   This is a datetime.datetime object by default.

year = g.time_created.year

month = g.time_created.month
```

This property is read-only, and can only be set during Graph construction.

## Graph.add_node

>Add a Node object to the Graph.

```python
g.add_node(node_object)
```

>Pass construction parameters for a Node:

```python
dr_zoidberg = g.add_node({
    'label': 'Dr. Zoidberg',
    'image': 'images/zoidberg.png',
    'shape': 1,
    'color': 'static',
    'radius': 55,
    'data': {
        'profession': '"Doctor"',
        'address': 'West 57th Street',
        'city': 'New New York'
    }
})
```

This is the prefered method to add Nodes to the graph.  It can be passed a Node object, or better, it can be passed construction parameters for a Node object. It returns the Node.

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
*optional* | label | Text that will accompany this Node when displayed | 'Unnamed Node'
*optional* | x | The x-coordinate of this Node when displayed | 1.0
*optional* | y | The y-coordinate of this Node when displayed | 1.0
*optional* | uid | The unique identifier of this Node | UUID
*optional* | radius | The radius in pixels of this Node when displayed  | 24.0
*optional* | color | Describes the color of the Node when displayed. Options are 'dynamic', 'static', or an RGB value. See below for details. | 'dynamic'
*optional* | image | When set to a valid image path and shape is set to 1, Node will display an image instead of a shape. | 'none'
*optional* | shape | An integer describing the shape the Node when displayed. 0 for circle, 1 for image, 3+ for n-gon | 6
*optional* | data | An object filled with various key-value pairs associated with the Node | {}
*optional* | comments | A list of any kind of comment objects you feel like using. | [] 

**More about color**:

* 'dynamic' will allow this Node to be colored according to the user-selected color settings.
* 'static' is usually used when Nodes display images.  This ensures that the image is displayed with true coloring, as opposed to having color filters applied to it.
* (r,g,b) RGB value will be a list or tuple, depending on language

## Graph.nodes

>Iterate through all the Nodes in the Graph

```python
for node in g.nodes:
    print node.label
```

A list of all Nodes in the Graph.

## Graph.node

>Get a Node with *uid* '127.0.0.1'

```python
n = g.node('127.0.0.1')
```

Returns a Node by its *uid* property

## Graph.add_link_type

>Add a LinkType object to the Graph.

```python
g.add_link_type(lt_object)
```

>Pass construction parameters for a LinkType:

```python
n = g.add_link_type({
    'name': 'voiced',
    'icon': 'images/voice_act.png',
    'min': 1,
    'max': 1
})
```

This is the prefered method to add LinkTypes to the graph.  It can be passed a LinkType object, or better, it can be passed construction parameters for a LinkType object. It returns the LinkType.

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
required | name | The name of the LinkType. Must be a string. | 
*optional* | min | The minimum acceptable value for Links of this Type | 1.0
*optional* | max | The maximum acceptable value for Links of this Type | 10.0
*optional* | units | What unit these Link values are expressed in. 'km', 'g', 'gigapascals', etc. | 'units'
*optional* | unit_scale | What scale these Link values are expressed in. 1000, 12, 1000000, etc. | 1
*optional* | icon | A valid image path to an icon representing this LinkType | 'icon_def_link'
*optional* | color | Describes the color of the LinkType when displayed. Options are 'dynamic', or an RGB value. See below for details. | 'dynamic'

**More about color**:

* 'dynamic' will allow this LinkType to be colored according to the user-selected color settings.
* (r,g,b) RGB value will be a list or tuple, depending on language

## Graph.link_types

>Iterate through all the LinkTypes in this Graph

```python
for lt in g.link_types:
    print lt.name
```

A list of all LinkTypes in this Graph

## Graph.link_type

>Retrieve a LinkType named "Packages"

```python
lt = g.link_type("Packages")
```

Returns a LinkType by its *name*.

## Graph.add_link

>Add a Link object to the Graph.

```python
g.add_link(link_object)
```

>Pass construction parameters for a Link:

```python
n = g.add_link({
    'origin': billy_west,
    'terminus': dr_zoidberg,
    'link_type': g.link_type('voiced'),
    'data' : {
        'audio': 'audio/zoidberg_quote.mp3'
    }
})
```

This is the prefered method to add Links to the Graph.  It can be passed a Link object, or better, it can be passed construction parameters for a Link object. It returns the Link.

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
required | origin | The Node object this Link should originate from. | 
required | terminus | The Node object this Link should terminate at. |
required | link_type | A LinkType object.  All Links must have a LinkType. |
*optional* | label | The text that will be shown on this Link when displayed | 'Unnamed Link'
*optional* | value | The value of this Link. Must be between the min and max values set by the LinkType | link_type.min
*optional* | directed | 'directed', 'bidrected', 'undirected' | 'directed'
*optional* | uid | A unique identifier for this Link | UUID
*optional* | data | An object filled with various key-value pairs associated with the Link | {}
*optional* | comments | A list of any kind of comment objects you feel like using. | [] 

## Graph.links

>Iterate through all the Links in the Graph

```python
for link in g.links:
    print link.label
```

A list of all Links in the Graph.

## Graph.link

>Retrieve a Link with *uid* '192.168.0.22:4456'

```python
link = g.link('192.168.0.22:4456')
```

Returns a Link by its *uid*.

## Graph.node_data_map

>Get salary information from a graph, and retrieve the salary from a Node object.

```python
salary_map, excluded = g.node_data_map('salary')
print salary_map[billy_west]
```

>Of course, this can also be done this way, but this may fail for Nodes that don't have the corresponding key.

```python
for n in g.nodes:
    print n.data['salary']
```

Returns a NodeDataMap, which is a Node-keyed object with values corresponding to a key in that node's data object.  Map has a key for every Node which has a matching key. It also returns a list of all the nodes which did not have the matching key.

Returned map:

`{
    <Node Object 0>: value,
    <Node Object 1>: different_value
}`

Returned exclusions:

`[<Node Object 2>, <Node Object 3>]`

## Graph.link_data_map

>Get link information from a graph, and retrieve the 'phone_rep' from a Link object.

```python
phone_rep_map, excluded = g.link_data_map('phone_rep')
for l in phone_rep_map:
    print phone_rep_map[l]
```

>Of course, this can also be done this way, but this may fail for Links that don't have the corresponding key.

```python
for l in g.links:
    print l.data['phone_rep']
```



Returns a LinkDataMap, which is a Link-keyed object with values corresponding to a key in that link's data object.  Map has a key for every Link which has a matching key. It also returns a list of all the links which did not have the matching key.

Returned map:

`{
<Link Object 0>: value,
<Link Object 1>: different_value
}`

Returned exclusions:

`[<Link Object 2>, <Link Object 3>]`

## Graph.link_value_map

>Get a raw LinkDataMap of Link values in a Graph:

```python
lvm = g.link_value_map()
```

>Get a normalized LinkDataMap of Link values in a Graph:

```python
normed_lv = g.link_value_map(normalized=True)
```

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | normalized | If True, normalizes all link values between 0 and 1 proportionate to their LinkTypes | False

## Graph.links_of_type

>Finding all links of type 'voiced', and print the actor that did the voicing.

```python
for link in g.links_of_type(g.link_type('voiced')):
    print link.origin.label
```

Returns a list of all Links of a specific LinkType

## Graph.draw

>All of these examples are using this Graph:

```python
g = psynth.price_network({
    'owner': 'me',
    'name': 'Price Network',
    'num_nodes': 300
})
```

PsynthDB contains a number of layout algorithms out of the box.  All of them accept these optional parameters, in addition to various alogorithm-specific settings.  By default, layouts are saved in a Node.data field of the same name. This effectively caches the result, and can eliminate the need for repeated queries.  All of these functions return a NodeDataMap with x,y coordinates.

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | set_position | When True, this will apply the result of the algorithm each Node.position in the graph. | False
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True

### Graph.draw.sfdp

>Set a Graph's position to a SFDP layout, but don't save the layout internally in a Node.data field.

```python
g.draw.sfdp({
    'set_position': True,
    'internalize': False
})
```

>Obtain the SFDP layout of a graph, but don't internalize the result.

```python
sfdp_layout = g.draw.sfdp({
    'internalize': False
})
```

>Obtain the SFDP layout of a Graph, using 'salary' as Node weights, and normalized Link values as Link weights.

```python
sfdp_layout = g.draw.sfdp({
    'weight': g.link_value_map(normalized=True),
    'node_weight': g.node_data_map('salary')
})
```

Obtain the SFDP spring-block layout of the graph.

This algorithm has complexity O(N log N), and is defined [here](http://www.mathematica-journal.com/issue/v10i1/graph_draw.html).


Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | set_position | When True, this will apply the result of the algorithm each Node.position in the graph. | False
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True
*optional* | data | The name of the Node.data field to save the result of this calculation. Requires that *internalize* field be True. | 'sfdp'
*optional* | weight | A LinkDataMap with numerical values to use as link weights in the calculation | None
*optional* | node_weight | A NodeDataMap with numerical values to use as node weights in the calculation | None

Returns a NodeDataMap with [x,y] positions as values.

Returned map:

`{
<Node Object 0>: [x1,y1],
<Node Object 1>: [x2,y2]
}`

![alt text](images/PNet_300_sfdp.png "SFDP layout of a Price network")

### Graph.draw.fruchertman_reingold

>Set a Graph's position to a Fruchterman-Reingold layout, but don't save the layout internally in a Node.data field.

```python
g.draw.fruchertman_reingold({
    'set_position': True,
    'internalize': False
})
```

>Obtain the Fruchterman-Reingold layout of a graph, but don't internalize the result.

```python
fruchertman_reingold_layout = g.draw.fruchertman_reingold({
    'internalize': False
})
```

>Obtain the Fruchterman-Reingold layout layout of a Graph, using normalized Link values as Link weights.

```python
fruchertman_reingold_layout = g.draw.fruchertman_reingold({
    'weight': g.link_value_map(normalized=True)
})
```

Calculate the Fruchterman-Reingold spring-block layout of the graph.

This algorithm has complexity O(n-iter×(N+L)), and is defined [here](http://onlinelibrary.wiley.com/doi/10.1002/spe.4380211102/abstract).

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | set_position | When True, this will apply the result of the algorithm each Node.position in the graph. | False
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True
*optional* | data | The name of the Node.data field to save the result of this calculation. Requires that *internalize* field be True. | 'fruchterman_reingold'
*optional* | weight | A LinkDataMap with numerical values to use as link weights in the calculation | None

Returns a NodeDataMap with [x,y] positions as values.

Returned map:

`{
<Node Object 0>: [x1,y1],
<Node Object 1>: [x2,y2]
}`

![alt text](images/PNet_300_fr.png "Fruchterman-Reingold layout of a Price network")

### Graph.draw.arf

>Set a Graph's position to the ARF layout, but don't save the layout internally in a Node.data field.

```python
g.draw.arf({
    'set_position': True,
    'internalize': False
})
```

>Obtain the ARF layout of a graph, but don't internalize the result.

```python
arf_layout = g.draw.arf({
    'internalize': False
})
```

>Obtain the ARF layout layout of a Graph, using normalized Link values as Link weights.

```python
arf_layout = g.draw.arf({
    'weight': g.link_value_map(normalized=True)
})
```

Calculate the ARF spring-block layout of the graph.

This algorithm has complexity O(N<sup>2</sup>) and is defined [here](http://www.worldscientific.com/doi/abs/10.1142/S0129183107011558)

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | set_position | When True, this will apply the result of the algorithm each Node.position in the graph. | False
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True
*optional* | data | The name of the Node.data field to save the result of this calculation. Requires that *internalize* field be True. | 'arf'
*optional* | weight | A LinkDataMap with numerical values to use as link weights in the calculation | None

Returns a NodeDataMap with [x,y] positions as values.

Returned map:

`{
<Node Object 0>: [x1,y1],
<Node Object 1>: [x2,y2]
}`

![alt text](images/PNet_300_arf.png "ARF layout of Price Network")

### Graph.draw.radial_tree

>Set a Graph's position to a Radial Tree layout, but don't save the layout internally in a Node.data field.

```python
g.draw.radial_tree({
    'root': g.nodes[0],
    'set_position': True,
    'internalize': False
})
```

>Obtain the Radial Tree layout of a graph, but don't internalize the result.

```python
radial_layout = g.draw.radial_tree({
    'root': g.nodes[0],
    'internalize': False
})
```

>Obtain the Radial Tree layout layout of a Graph, using 'salary' data from Nodes..

```python
radial_layout = g.draw.radial_tree({
    'root': g.nodes[0],
    'node_weight': g.node_data_map('salary')
})
```

Computes a radial layout of the graph according to the minimum spanning tree centered at the root vertex.

This algorithm has complexity O(N+L).

Required | Parameter | Description | Default
---------|-----------|-------------|---------
required | root | A Node object to be the Root Node for this layout |
*optional* | set_position | When True, this will apply the result of the algorithm each Node.position in the graph. | False
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True
*optional* | data | The name of the Node.data field to save the result of this calculation. Requires that *internalize* field be True. | 'radial_tree'
*optional* | node_weight | A NodeDataMap with numerical values to use as node weights in the calculation | None

Returns a NodeDataMap with [x,y] positions as values.

Returned map:

`{
<Node Object 0>: [x1,y1],
<Node Object 1>: [x2,y2]
}`

![alt text](images/PNet_300_radial_tree.png "Radial Tree layout of Price Network")

### Graph.draw.random_layout

>Set a Graph's position to a random layout, but don't save the layout internally in a Node.data field.

```python
g.draw.random_layout({
    'set_position': True,
    'internalize': False
})
```

>Obtain the ARF layout of a graph, but don't internalize the result.

```python
random_layout = g.draw.random_layout({
    'internalize': False
})
```

Performs a random layout of the graph.

This algorithm has complexity O(N).

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | set_position | When True, this will apply the result of the algorithm each Node.position in the graph. | False
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True
*optional* | data | The name of the Node.data field to save the result of this calculation. Requires that *internalize* field be True. | 'random_layout'

Returns a NodeDataMap with [x,y] positions as values.

Returned map:

`{
<Node Object 0>: [x1,y1],
<Node Object 1>: [x2,y2]
}`

![alt text](images/PNet_300_random.png "Random layout of Price Network")

## Graph.centrality

>All of the images are generated from this Graph:

```python
g = psynth.price_network({
    'owner': 'me',
    'name': 'Price Network',
    'num_nodes': 400
})

g.draw.sfdp({
    'set_position': True
})
```

PsynthDB contains a number of centrality algorithms out of the box.  All of them accept an optional parameter to internalize the result, in addition to various alogorithm-specific settings.  By default, centrality measures are saved in a Node.data field of the same name. This effectively caches the result, and can eliminate the need for repeated queries.  All of these methods return a NodeDataMap, and a LinkDataMap if applicable.

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True

### Graph.centrality.pagerank

>Obtain the PageRank values of a graph, and internalize the result.

```python
pr_map = g.centrality.pagerank()
```

>Obtain the Pagerank values of a graph, using normalized Link values for weights:

```python
pr_map = g.centrality.pagerank({
    'weight': g.link_value_map(normalized=True)
})
```

>View the Pagerank values as Node Size and Alpha.

```python
pagerank = g.centrality.pagerank().normalized()

g.save_image({
    'filename': 'images/PNet_400_pagerank.png',
    'width': 600,
    'height': 600,
    'max_link_thickness': 4,
    'node_alpha': pagerank,
    'node_radius': pagerank,
    'max_node_radius': 8
})
```

The pagerank algorithm is described [here](https://en.wikipedia.org/wiki/PageRank) and has a topology-dependent running time.


Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True
*optional* | data | The name of the Node.data field to save the result of this calculation. Requires that *internalize* field be True. | 'pagerank'
*optional* | weight | A LinkDataMap with numerical values to use as link weights in the calculation | None

Returns a NodeDataMap.

Returned map:

`{
<Node Object 0>: float_number,
<Node Object 1>: float_number
}`

![alt text](images/PNet_400_pagerank.png "PageRank values of a Price Network")

### Graph.centrality.betweenness

>Obtain the Betweeness values of a graph, and internalize the result.

```python
node_values, link_values = g.centrality.betweeness()
```

>Obtain the Betweeness values of a graph, using normalized Link values for weights:

```python
node_values, link_values = g.centrality.betweenness({
    'weight': g.link_value_map(normalized=True)
})
```

>View the Node Betweenness values as Node Size and Alpha.

```python
node_values, link_values = g.centrality.betweeness()
node_values = node_values.normalized()

g.save_image({
    'filename': 'images/PNet_400_node_betweenness.png',
    'width': 600,
    'height': 600,
    'max_link_thickness': 4,
    'node_alpha': node_values,
    'node_radius': node_values,
    'max_node_radius': 8
})
```

>View the Link Betweenness values as Link Thickness and Alpha.

```python
node_values, link_values = g.centrality.betweeness()
link_values = link_values.normalized()

g.save_image({
    'filename': 'images/PNet_400_link_betweenness.png',
    'width': 600,
    'height': 600,
    'max_link_thickness': 4,
    'link_alpha': link_values,
    'link_thickness': link_values
})
```

>View the Node Betweenness values as Node Size and Alpha, and Link Betweenness as Link Thickness and Alpha.

```python
node_values, link_values = g.centrality.betweeness()
node_values = node_values.normalized()
link_values = link_values.normalized()

g.save_image({
    'filename': 'images/PNet_400_betweenness.png',
    'width': 600,
    'height': 600,
    'max_link_thickness': 4,
    'max_node_radius': 8,
    'node_alpha': node_values,
    'node_radius': node_values,
    'link_alpha': link_values,
    'link_thickness': link_values
})
```

The algorithm used here is defined in [brandes-faster-2001](http://dx.doi.org/10.1080/0022250X.2001.9990249), and has a complexity of O(NL) for unweighted graphs.

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True
*optional* | node_data | The name of the Node.data field to save the result of this calculation. Requires that *internalize* field be True. | 'node_betweenness'
*optional* | link_data | The name of the Link.data field to save the result of this calculation. Requires that *internalize* field be True. | 'link_betweenness'
*optional* | weight | A LinkDataMap with numerical values to use as link weights in the calculation | None

Returns a NodeDataMap, as well as a LinkDataMap.

Returned maps:

`{
<Node Object 0>: float_number,
<Node Object 1>: float_number
}`

`{
<Link Object 0>: float_number,
<Link Object 1>: float_number
}`

Node Betweenness values:
![alt text](images/PNet_400_node_betweenness.png "Node Betweenness values of a Price Network")

Link Betweenness values:
![alt text](images/PNet_400_link_betweenness.png "Link Betweenness values of a Price Network")

Node and Link Betweenness values:
![alt text](images/PNet_400_betweenness.png "Betweenness values of a Price Network")

### Graph.centrality.closeness

>Obtain the Closeness values of a graph, and internalize the result.

```python
closeness_map = g.centrality.closeness()
```

>Obtain the Closeness values of a graph, using normalized Link values for weights:

```python
closeness_map = g.centrality.closeness({
    'weight': g.link_value_map(normalized=True)
})
```

>View the Closeness values of a Graph, using Node Radius and Alpha.

```python
closeness = g.centrality.closeness().normalized()

g.save_image({
    'filename': 'images/PNet_400_closeness.png',
    'width': 600,
    'height': 600,
    'max_link_thickness': 4,
    'node_alpha': closeness,
    'node_radius': closeness,
    'max_node_radius': 8
})
```

The closeness algorithm is discussed [here.](https://en.wikipedia.org/wiki/Centrality#Closeness_centrality)

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True
*optional* | data | The name of the Node.data field to save the result of this calculation. Requires that *internalize* field be True. | 'closeness'
*optional* | weight | A LinkDataMap with numerical values to use as link weights in the calculation | None

Returns a NodeDataMap.

Returned map:

`{
<Node Object 0>: float_number,
<Node Object 1>: float_number
}`

![alt text](images/PNet_400_closeness.png "Closeness values of a Price Network")

### Graph.centrality.eigenvector

>Obtain the Eigenvector values of a graph, and internalize the result.

```python
eigenvalue, eigenvector_map = g.centrality.eigenvector()
```

>Obtain the Eigenvector values of a graph, using normalized Link values for weights:

```python
eigenvalue, eigenvector_map = g.centrality.eigenvector({
    'weight': g.link_value_map(normalized=True)
})
```

>View the Eigenvector values for a Graph as Node Radius and Alpha

```python
eigenvalue, eigenvector = g.centrality.eigenvector()
eigenvector = eigenvector.normalized()

g.save_image({
    'filename': 'images/PNet_400_eigenvector.png',
    'width': 600,
    'height': 600,
    'max_link_thickness': 4,
    'node_alpha': eigenvector,
    'node_radius': eigenvector,
    'max_node_radius': 8
})
```

The eigenvector algorithm is discussed [here.](https://en.wikipedia.org/wiki/Centrality#Eigenvector_centrality)

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True
*optional* | data | The name of the Node.data field to save the result of this calculation. Requires that *internalize* field be True. | 'eigenvector'
*optional* | weight | A LinkDataMap with numerical values to use as link weights in the calculation | None

Returns the largest Eigenvalue of the (weighted) adjacency matrix, as well as a NodeDataMap.

Returned map:

`{
<Node Object 0>: float_number,
<Node Object 1>: float_number
}`

![alt text](images/PNet_400_eigenvector.png "Eigenvector values of a Price Network")


### Graph.centrality.katz

>Obtain the Katz values of a graph, and internalize the result.

```python
katz_map = g.centrality.katz()
```

>Obtain the Katz values of a graph, using normalized Link values for weights:

```python
katz_map = g.centrality.katz({
    'weight': g.link_value_map(normalized=True)
})
```

>View the Katz values for a Graph as Node Radius and Alpha

```python
katz = g.centrality.katz().normalized()

print "Saving..."
g.save_image({
    'filename': 'images/PNet_400_katz.png',
    'width': 600,
    'height': 600,
    'max_link_thickness': 4,
    'node_alpha': katz,
    'node_radius': katz,
    'max_node_radius': 8
})
```

The Katz algorithm is discussed [here.](https://en.wikipedia.org/wiki/Katz_centrality)

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True
*optional* | data | The name of the Node.data field to save the result of this calculation. Requires that *internalize* field be True. | 'katz'
*optional* | weight | A LinkDataMap with numerical values to use as link weights in the calculation | None

Returns a NodeDataMap.

Returned map:

`{
<Node Object 0>: float_number,
<Node Object 1>: float_number
}`

![alt text](images/PNet_400_katz.png "Katz values of a Price Network")


### Graph.centrality.hits

>Obtain the largest Eigenvalue, Hits Authority, and Hits Hub values of a graph, and internalize the result.

```python
eig, auth, hub = g.centrality.hits()
```

>Obtain the largest Eigenvalue, Hits Authority, and Hits Hub values of a graph, using normalized Link values for weights:

```python
eig, auth, hub = g.centrality.eigenvector({
    'weight': g.link_value_map(normalized=True)
})
```

>View the Hits Authority values for a Graph as Node Radius and Alpha

```python
eig, auth, hub = g.centrality.hits()
auth = auth.normalized()

g.save_image({
    'filename': 'images/PNet_400_hits_auth.png',
    'width': 600,
    'height': 600,
    'max_link_thickness': 4,
    'node_alpha': auth,
    'node_radius': auth,
    'max_node_radius': 8
})
```

>View the Hits Hub values for a Graph as Node Radius and Alpha

```python
eig, auth, hub = g.centrality.hits()
hub = hub.normalized()

g.save_image({
    'filename': 'images/PNet_400_hits_auth.png',
    'width': 600,
    'height': 600,
    'max_link_thickness': 4,
    'node_alpha': hub,
    'node_radius': hub,
    'max_node_radius': 8
})
```


The HITS algorithm is discussed [here.](https://en.wikipedia.org/wiki/HITS_algorithm)

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | internalize | When True, this will save the result of the algorithm to a Node.data field | True
*optional* | authority | The name of the Node.data field to save the result of this calculation. Requires that *internalize* field be True. | 'hits_authority'
*optional* | hub | The name of the Node.data field to save the result of this calculation. Requires that *internalize* field be True. | 'hits_hub'
*optional* | weight | A LinkDataMap with numerical values to use as link weights in the calculation | None

Returns the largest Eigenvalue in the Graph, as well as a NodeDataMap each for the Authority and Hub calculation.

Returned maps:

`{
<Node Object 0>: auth_number,
<Node Object 1>: auth_number
}`


`{
<Node Object 0>: hub_number,
<Node Object 1>: hub_number
}`

Hits Authority:
![alt text](images/PNet_400_hits_auth.png "HITS Authority values of a Price Network")

Hits Hub:
![alt text](images/PNet_400_hits_hub.png "HITS Hub values of a Price Network")


## Graph.topology

PsynthDB includes a number of topology functions out of the box.

### Graph.topology.shortest_distance

>Shortest Distance with no parameters

```python
sd_map = g.topology.shortest_distance()

for origin in sd_map:
    for terminus in sd_map[origin]:
        print origin.name +" -> "+terminus.name+": "+sd_map[origin][terminus]
```

>Shortest Distance with an *origin*

```python
origin = g.nodes[0]

sd_map = g.topology.shortest_distance({
    'origin': origin
})

for terminus in sd_map:
    print origin.name +" -> "+terminus.name+": "+sd_map[terminus]
```

>Shortest Distance with an *origin* and a *terminus*

```python
print g.topology.shortest_distance({
    'origin': g.nodes[0],
    'terminus': g.nodes[10]
})
```

```json
4
```

This finds the shortest distance between any two Nodes.  It is the number of Links between them, or the total of the link weights if weights are given (not currently supported).  

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*optional* | origin | A Node object beginning the path | None
*optional* | terminus | A Node object ending the path | None
*optional* | directed | Whether or not the path most obey link direction. | False

If *origin* and *terminus* are omitted, it returns a NodeDataMap expressing the distance from every node to every other node.

`{<Node Object>: <NodeDataMap Object>}`

If *origin* is provided, and *terminus* is omitted, it returns a NodeDataMap expressing the distance from the *origin* to every Node in the Graph

`{<Node Object>: Distance}`

If *origin* and *terminus* are both provided, it returns a Number expressing the distance between the two Nodes.

`<int32>`

### Graph.topology.shortest_path

>Get the shortest path between the first and last Node in the Graph.

```python
nodes, links = g.topology.shortest_path({
    'origin': g.nodes[0],
    'terminus': g.nodes[-1]
})
```

Returns the shortest path between two Nodes.

Required | Parameter | Description | Default
---------|-----------|-------------|---------
required | origin | A Node object beginning the path | None
required | terminus | A Node object ending the path | None

Returns a list of Nodes, and list of Links that makes up the path.

`[<Node Object>], [<Link Object>]`

### Graph.topology.pseudo_diameter

>Calculate the distance and endpoints of the Graph's pseudo-diameter.

```python
dist, ends = g.topology.pseudo_diameter()
```

The pseudo-diameter is an approximate graph diameter. It is obtained by starting from a vertex *origin*, and finds a vertex *terminus* that is furthest away from *origin*.


Required | Parameter | Description | Default
---------|-----------|-------------|---------
*options* | origin | A Node object from which to start the diameter | None

Returns the distance

`<Number>`

and a pair of Nodes which are the endpoints of the calculation

`[<Node Object>, <Node Object>]`

### Graph.topology.max_cardinality_matching

>Find a maximum cardinality matching in the graph.

```python
match = g.topology.max_cardinality_matching()
```

A matching is a subset of the links of a graph such that no two links share a common node. A maximum cardinality matching has maximum size over all matchings in the graph.

This algorithm runs in time O(NL×α(N,L)), where α(m,n) is a slow growing function that is at most 4 for any feasible input. More on the algorithm [here.](http://www.boost.org/doc/libs/1_59_0/libs/graph/doc/maximum_matching.html)

This returns a LinkDataMap with Boolean values expressing whether they are part of the matching.

`{<Link Object>: <Boolean>}`

### Graph.topology.max_independent_node_set

>Find a maximal independent node set in the graph.

```python
vset = g.topology.max_independent_node_set()
```

A maximal independent node set is an independent set such that adding any other node to the set forces the set to contain a link between two nodes of the set.

This implements the algorithm described [here](http://dl.acm.org/citation.cfm?doid=22145.22146), which runs in time O(N+L).

This return a NodeDataMap with Boolean values expressing whether they are part of the set.

`{<Node Object>: <Boolean>}`

### Graph.topology.min_spanning_tree

>Return the minimum spanning tree of a graph.

```python
min_tree = g.topology.min_spanning_tree()
```

Required | Parameter | Description | Default
---------|-----------|-------------|---------
*options* | root | A Node object forming the root of the tree. | None

Returns the minimum spanning tree of the Graph as a LinkDataMap with Boolean values expressing whether they belong to the tree.

The algorithm runs with O(LlogL) complexity, or O(LlogN) if root is specified.

Original Graph:
![alt text](images/triangle_400.png "Original Graph")

Minimum Spanning Tree:
![alt text](images/triangle_min_tree.png "Minimum Spanning Tree")

### Graph.topology.topological_sort

>Return a topological sort of an acyclic graph.

```python
tsort = g.topology.topological_sort()
```

The topological sort algorithm creates a linear ordering of the nodes such that if Link (u,v) appears in the graph, then u comes before v in the ordering. The graph must be a [directed acyclic graph (DAG)](https://en.wikipedia.org/wiki/Directed_acyclic_graph).  [More on topological sorts.](https://en.wikipedia.org/wiki/Topological_sorting)

The time complexity is O(V+E).

Returns an ordered list of Nodes.

`[<Node Object>]`

### Graph.topology.kcore_decomposition

>Get the k-core decomposition of a Graph, by in-degree:

```python
in_deg = g.topology.kcore_decomposition("in")
```

>Get the k-core decomposition of a Graph, by out-degree:

```python
out_deg = g.topology.kcore_decomposition("out")
```

>Get the k-core decomposition of a Graph, by total-degree:

```python
total_deg = g.topology.kcore_decomposition("total")
```

Required | Parameter | Description | Default
---------|-----------|-------------|---------
required | degree | "in", "out", or "total" |

The k-core is a maximal set of vertices such that its induced subgraph only contains vertices with degree larger than or equal to k.

This algorithm is described [here](http://dx.doi.org/10.1007/s11634-010-0079-y) and runs in O(V+E) time.

It returns a NodeDataMap with the degree of each Node.

`{<Node Object>: <int32>}`

![alt text](http://graph-tool.skewed.de/static/doc/_images/netsci-kcore.png "K-core decomposition of a network of network scientists.")

## Graph.as_dict

```python
print g.as_dict
```

>This returns a neatly formatted object ready for JSON encoding and transmission. 

```json
{
    "filename": "b1ec2e90-c3bf-4373-b065-fd7b2240284d.gt",
    "name": "Authorship Web for Protein Folding Papers 2012-2015",
    "owner": "Dr. Allison James",
    "time_created": {
        "year": 2016,
        "month": 7,
        "day": 22,
        "hour": 11,
        "minute": 42,
        "second": 13,
        "microsecond": 978816,
        "tzinfo": "None"
    },
    "link_types": [
        {
            "name": "Authored",
            "min": 1.0,
            "max": 2.0,
            "color": [255, 0, 0],
            "units": "units",
            "unit_scale": 1,
            "icon": "image/pen.png"
        }
    ],
    "nodes": [
        {
            "label": "Robert T. McGibbon",
            "uid": "0d780b67-e6e1-4a73-b0f9-bfa5d97d2eba",
            "x": 1.0,
            "y": 1.0,
            "color": "dynamic",
            "shape": 6,
            "radius": 24.0,
            "image": "none",
            "data": {
                "institution": "Stanford University"
            },
            "comments": []
        },
        {
            "label": "Perspective: Markov models for long-timescale biomolecular dynamics",
            "uid": "f38d625d-5927-40f2-baf9-4c2121d6b502",
            "x": 1.0,
            "y": 1.0,
            "color": "dynamic",
            "shape": 4,
            "radius": 24.0,
            "image": "none",
            "data": {
                "journal": "The Journal of chemical physics"
            },
            "comments": []
        }
    ],
    "links": [
        {
            "label": "2014",
            "uid": "c695170b-6709-4406-adb4-9709bb014c05",
            "directed": "directed",
            "value": 1.0,
            "comments": [],
            "data": {},
            "type": "Authored",
            "origin": "0d780b67-e6e1-4a73-b0f9-bfa5d97d2eba",
            "terminus": "f38d625d-5927-40f2-baf9-4c2121d6b502"
        }
    ]
}
```

Returns a JSON-ready object with all of the Graph's data. This feature exists predominately to ease RESTful exposure.  Editing this dictionary will not edit the actual Graph.  If you have embedded custom object types into the Graph, it is recommended that you add an *as_dict* property to that Class.  If your object has an *as_dict* property, it will be automatically called when generating this output.


# Node

>Constucting a Node object through the constructor:

```python
dr_zoidberg = psynth.Node({
    'graph': g
    'label': 'Dr. Zoidberg',
    'image': 'images/zoidberg.png',
    'shape': 1,
    'color': 'static',
    'radius': 55,
    'data': {
        'profession': '"Doctor"',
        'address': 'West 57th Street'
        'city': 'New New York'
    }
})
g.add_node(dr_zoidberg)
```

>However, by accessing it through the Graph class's *add_node* method directly, you can omit the 'graph' parameter, and perform the creation and addition in a single declaration.

```python
dr_zoidberg = g.add_node({
    'label': 'Dr. Zoidberg',
    'image': 'images/zoidberg.png',
    'shape': 1,
    'color': 'static',
    'radius': 55,
    'data': {
        'profession': '"Doctor"',
        'address': 'West 57th Street'
        'city': 'New New York'
    }
})
```

Nodes are one of the basic pieces of a Graph.  They connect to other Nodes via Links.  Nodes have a variety of permanent properties designed to assist in visualization, as well as a *data* property for storing arbitrary key-value pairs associated with the Node.  For algorithms using weighted Nodes, the values for determing Node weight would be found in the *data* property.  Nodes also have a place to store comments, to assist in collaborative graph editing and analytics.  It's generally not necessary to use the *x* and *y* properties during construction if you intend to run a layout algorithm later.

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
required | graph | A Graph object that the Node will belond to. | 
*optional* | label | Text that will accompany this Node when displayed | ''
*optional* | x | The x-coordinate of this Node when displayed | 1.0
*optional* | y | The y-coordinate of this Node when displayed | 1.0
*optional* | uid | The unique identifier of this Node | UUID
*optional* | radius | The radius in pixels of this Node when displayed  | 24.0
*optional* | color | Describes the color of the Node when displayed. Options are 'dynamic', 'static', or an RGB value. See below for details. | 'dynamic'
*optional* | image | When set to a valid image path and shape is set to 1, Node will display an image instead of a shape. | 'none'
*optional* | shape | An integer describing the shape the Node when displayed. 0 for circle, 1 for image, 3+ for n-gon | 6
*optional* | data | An object filled with various key-value pairs associated with the Node | {}
*optional* | comments | A list of any kind of comment objects you feel like using. | [] 

**More about color**:

* 'dynamic' will allow this Node to be colored according to the user-selected color settings.
* 'static' is usually used when Nodes display images.  This ensures that the image is displayed with true coloring, as opposed to having color filters applied to it.
* (r,g,b) RGB value will be a list or tuple, depending on language

## Node.uid

>Access the Node's uid.

```python
print dr_zoidberg.uid
```

```json
"0d780b67-e6e1-4a73-b0f9-bfa5d97d2eba"
```

A String which uniquely identifies this Node within the Graph.  By default, PsynthDB uses UUID4 Global Unique Identifiers.  There are certainly use cases in which a different uid scheme could be appropriate.  IP addresses, MAC addresses, etc.

## Node.label

>Access the Node's label.

```python
print dr_zoidberg.label
```

```json
"Dr. Zoidberg"
```

>Set the Node's label.

```python
dr_zoidberg.label = "Dr. John E. Zoidberg"
```

A String which labels this Node.

## Node.position

>Access the Node's position.

```python
print dr_zoidberg.position
```

```json
[423.4, 105.6]
```

>Set the Node's position
dr_zoidberg.position = [132, 84]

The x,y-coordinates of this Node, as a List.

## Node.x

>Access the Node's x-coordinate.

```python
print dr_zoidberg.x
```

```json
423.4
```

>Set the Node's x-coordinate.

```python
dr_zoidberg.x = 132
```

The x-coordinate of this Node, as a float.

## Node.y

>Access the Node's y-coordinate.

```python
print dr_zoidberg.y
```

```json
105.6
```

>Set the Node's y-coordinate.

```python
dr_zoidberg.y = 84
```

The y-coordinate of this Node, as a float.

## Node.radius

>Access the Node's radius.

```python
print dr_zoidberg.radius
```

```json
55.0
```

>Set the Node's radius.

```python
dr_zoidberg.radius = 32.0
```

The radius of this Node, as a float.

## Node.color

>Access the Node's color information.

```python
print dr_zoidberg.color
```

```json
"static"
```

>Set the Node's color information

```python
dr_zoidberg.color = (255, 0, 222)
```

PsynthDB has a few options for coloring Nodes.

* 'dynamic' will allow this Node to be colored according to the user-selected color settings.  This feature really works best with geometric node shapes, as opposed to images, but color filters can be applied to images to achieve this effect for images as well.
* 'static' is usually used when Nodes display images.  This ensures that the image is displayed with true coloring, as opposed to having color filters applied to it.
* (r,g,b) RGB value as a list or tuple, depending on language. This feature really works best with geometric node shapes, as opposed to images, but color filters can be applied to images to achieve this effect for images as well.

## Node.red

>Retrieve the red value.

```python
dr_zoidberg.color = 'dynamic'
print dr_zoidberg.red
```

```json
-1.0
```

```python
dr_zoidberg.color = (255, 0, 222)
print dr_zoidberg.red
```

```json
255
```

>Set the red value.

```python
dr_zoidberg.red = 125
```

If *color* is set to 'dynamic' or 'static', this will return -1.0.  Otherwise, it returns a number 0-255 for the red value of the color of this Node.

## Node.green

>Retrieve the green value.

```python
dr_zoidberg.color = 'static'
print dr_zoidberg.green
```

```json
-1.0
```

```python
dr_zoidberg.color = (255, 0, 222)
print dr_zoidberg.green
```

```json
0
```

>Set the green value.

```python
dr_zoidberg.green = 42
```

If *color* is set to 'dynamic' or 'static', this will return -1.0.  Otherwise, it returns a number 0-255 for the green value of the color of this Node.

## Node.blue

>Retrieve the blue value.

```python
dr_zoidberg.color = 'static'
print dr_zoidberg.blue
```

```json
-1.0
```

```python
dr_zoidberg.color = (255, 0, 222)
print dr_zoidberg.blue
```

```json
222
```

>Set the blue value.

```python
dr_zoidberg.blue = 189
```

If *color* is set to 'dynamic' or 'static', this will return -1.0.  Otherwise, it returns a number 0-255 for the blue value of the color of this Node.

## Node.image

>Access a Node's image:

```python
print dr_zoidberg.image
```

```json
"images/zoidberg.png"
```

>Set a Node's image:

```python
dr_zoidberg.image = "image/zoidberg_out.png"
```

This is either 'none', or a valid path to an image.  This value will be accessed by your visualization platform directly, so the path should be accessible from it.  URLs are usually how this is handled for web-based platforms.  Since this is just a path, image format is only relevant so much as it should work with your visualization front-end.  

## Node.shape

>Access a Node's shape:

```python
print dr_zoidberg.shape
```

```json
1
```

>Set a Node's shape:

```python
dr_zoidberg.shape = 6
```

Integers represent shape in PsynthDB, as follows:

Number | Shape
-------|-------
0 | circle
1 | image; requires that the *image* property have a valid image path.
3 | triangle
4 | diamond(square)
5 | pentagon
6 | hexagon
7+ | n-gon

## Node.data

>Retrieve salary information from a Node:

```python
sal = dr_zoidberg.data['salary']
```

>Set the salary information for a Node:

```python
dr_zoidberg.data['salary'] = 100.00
```

>Set random salary information for every Node in a Graph:

```python
from random import randint

for n in g.nodes:
    n.data['salary'] = randint(1000, 5000)
```

Store and retrieve information about Nodes.  Supports fully arbitrary key-value pairing. If the graph is to be exposed RESTfully, it is recommended to use Strings for keys, and JSON encodable values.

## Node.comments

>Leave a plaintext comment on a Node:

```python
dr_zoidberg.comments.append("What a smelly lobster.")
```

>Leave a comment object on a Node:

```python
from datetime import datetime

dr_zoidberg.comments.append({
    'user': 'graph user',
    'time': datetime.now(),
    'content': "What a smelly lobster.",
    'replies': []
})
```

>Retrieve the most recent comment left on a node:

```python
print n.comments[-1]
```

Store and retrieve comments that have been left on this Node.  This is an ordered list.  PsynthDB doesn't include a comment object, but you can define whatever one you'd like and use it here.

## Node.link_to

>Link from one node to another directly.

```python
billy_west.link_to(dr_zoidberg, {
    'link_type': g.link_type('voiced'),
    'data' : {
        'audio': 'audio/zoidberg_quote.mp3'
    }
})
```

This creates a Link from this Node to a terminus Node.

It requires a Node object to use as the terminus, as well a parameters object to define a Link

The parameters for the link are as follows:

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
required | link_type | A LinkType object.  All Links must have a LinkType. |
*optional* | label | The text that will be shown on this Link when displayed | 'Unnamed Link'
*optional* | value | The value of this Link. Must be between the min and max values set by the LinkType | link_type.min
*optional* | directed | 'directed', 'bidrected', 'undirected' | 'directed'
*optional* | uid | A unique identifier for this Link | UUID
*optional* | data | An object filled with various key-value pairs associated with the Link | {}
*optional* | comments | A list of any kind of comment objects you feel like using. | [] 

## Node.link_from

>Link to one node from another directly.

```python
dr_zoidberg.link_from(billy_west, {
    'link_type': g.link_type('voiced'),
    'data' : {
        'audio': 'audio/zoidberg_quote.mp3'
    }
})
```

This creates a Link to this Node from a terminus Node.

It requires a Node object to use as the origin, as well a parameters object to define a Link

The parameters for the link are as follows:

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
required | link_type | A LinkType object.  All Links must have a LinkType. |
*optional* | label | The text that will be shown on this Link when displayed | 'Unnamed Link'
*optional* | value | The value of this Link. Must be between the min and max values set by the LinkType | link_type.min
*optional* | directed | 'directed', 'bidrected', 'undirected' | 'directed'
*optional* | uid | A unique identifier for this Link | UUID
*optional* | data | An object filled with various key-value pairs associated with the Link | {}
*optional* | comments | A list of any kind of comment objects you feel like using. | [] 

## Node.in_neighbors

>Print the label of all of a Node's in-neighbors.

```python
for n in dr_zoidberg.in_neighbors:
    print n.label
```

Returns a list of all Nodes which have a Link that terminates in this Node.  Both 'directed' and 'bidrected' Links can be traversed for this, but not 'undirected'

## Node.out_neighbors

>Print the label of all of a Node's out-neighbors.

```python
for n in dr_zoidberg.out_neighbors:
    print n.label
```

Returns a list of all Nodes which have a Link that originates at this Node.  Both 'directed' and 'bidrected' Links can be traversed for this, but not 'undirected'

## Node.all_neighbors

>Print the label of all of a Node's neighbors.

```python
for n in dr_zoidberg.all_neighbors:
    print n.label
```

Returns a list of all Nodes which have a Link connecting them to this Node.  All links are traversable, regardless of directedness.

## Node.in_links

>Print the value of all Links which terminate in this Node

```python
for l in dr_zoidberg.in_links:
    print l.value
```

Returns a list of all Links which terminate in the Node.  Includes 'directed' and 'bidrected' Links, but not 'undirected'.

## Node.out_links

>Print the value of all Links which originate in this Node

```python
for l in dr_zoidberg.out_links:
    print l.value
```

Returns a list of all Links which originate in the Node.  Includes 'directed' and 'bidrected' Links, but not 'undirected'.

## Node.all_links

>Print the value of all Links connected to this Node

```python
for l in dr_zoidberg.all_links:
    print l.value
```

Returns a list of all Links which connect to the Node.  All links are traversable, regardless of directedness.

## Node.as_dict

```python
print dr_zoidberg.as_dict
```

```json
{
    "label": "Dr. Zoidberg",
    "uid": "0d780b67-e6e1-4a73-b0f9-bfa5d97d2eba",
    "x": 1.0,
    "y": 1.0,
    "color": "static",
    "shape": 1,
    "radius": 55.0,
    "image": "images/dr_zoidberg.png",
    "data": {
        "profession": "'Doctor'",
        "address": "West 57th Street",
        "city": "New New York"
    },
    "comments": []
}
```

Returns a JSON-ready object describing the Node.  Editing this dictionary will not edit the Node.

# LinkType

>Construct a LinkType directly

```python
voiced = psynth.LinkType({
    'graph': g
    'name': 'voiced',
    'icon': 'images/voice_act.png',
    'max': 1
})
g.add_link_type(voiced)
```

>However, by doing this through the Graph.add_link_type method, you can omit the *graph* parameter, and do the whole thing in a single declaration:

```python
voiced = g.add_link_type({
    'name': 'voiced',
    'icon': 'images/voice_act.png',
    'max': 1
})
```

LinkTypes describe categorical information for Links.  All Links must be described by a LinkType.  LinkTypes set the minimum and maximum allowable values for Links of that type, unit type and scale, as well as color and icon information for visualization.

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
required | graph | The Graph this LinkType will be added to. This must be a Graph object. |
required | name | The name of the LinkType. Must be a string. | 
*optional* | min | The minimum acceptable value for Links of this Type | 1.0
*optional* | max | The maximum acceptable value for Links of this Type | 10.0
*optional* | units | What unit these Link values are expressed in. 'km', 'g', 'gigapascals', etc. | 'units'
*optional* | unit_scale | What scale these Link values are expressed in. 1000, 12, 1000000, etc. | 1
*optional* | icon | A valid image path to an icon representing this LinkType | 'icon_def_link'
*optional* | color | Describes the color of the LinkType when displayed. Options are 'dynamic', or an RGB value. See below for details. | 'dynamic'

**More about color**:

* 'dynamic' will allow this LinkType to be colored according to the user-selected color settings.
* (r,g,b) RGB value will be a list or tuple, depending on language

## LinkType.min

>Set minimum value for a LinkType

```python
voiced.min = 2
```

>Retrieve minimum value for a LinkType

```python
print voiced.min
```

```json
2
```

Set or Retrieve the minimum acceptable value for this LinkType.  Links in the Graph will be constrained to the new value. If the value set is higher than *max*, *max* will be set to this value as well.

## LinkType.max

>Set maximum value for a LinkType

```python
voiced.max = 3
```

>Retrieve maximum value for a LinkType

```python
print voiced.max
```

```json
3
```

Set or Retrieve the maximum acceptable value for this LinkType.  Links in the Graph will be constrained to the new value. If the value set is lower than *min*, *min* will be set to this value as well.

## LinkType.name

>Set the name for a LinkType

```python
voiced.name = 'voice acted'
```

>Retrieve minimum value for a LinkType

```python
print voiced.name
```

```json
"voice acted"
```

Set or Retrieve the name for this LinkType.

## LinkType.units

>Set the units for a LinkType

```python
sales = g.link_type('sales')
sales.units = 'USD'
```

>Retrieve units for a LinkType

```python
print sales.units
```

```json
"USD"
```

Set or Retrieve the units for this LinkType.  Examples:

* grams
* school buses
* gigapascals
* lightyears
* hours
* Euros

## LinkType.unit_scale

>Set the unit scale for a LinkType

```python
sales = g.link_type('sales')
sales.unit_scale = 1000
```

>Retrieve unit scale for a LinkType

```python
print sales.unit_scale
```

```json
1000
```

Set or Retrieve the unit scale for this LinkType.  This should be an integer.

## LinkType.icon

>Set the icon for a LinkType

```python
sales = g.link_type('sales')
sales.icon = 'images/usd.png'
```

>Retrieve icon for a LinkType

```python
print sales.icon
```

```json
"images/usd.png"
```

Set or Retrieve the units for this LinkType.

## LinkType.color

>Access the LinkType's color information.

```python
print voiced.color
```

```json
"dynamic"
```

>Set the LinkType's color information

```python
voiced.color = (255, 0, 222)
```

PsynthDB has a couple options for coloring LinkTypes.

* 'dynamic' will allow this LinkType to be colored according to the user-selected color settings.
* (r,g,b) RGB value as a list or tuple, depending on language.

## LinkType.red

>Retrieve the red value.

```python
voiced.color = 'dynamic'
print voiced.red
```

```json
-1.0
```

```python
voiced.color = (255, 0, 222)
print voiced.red
```

```json
255
```

>Set the red value.

```python
voiced.red = 125
```

If *color* is set to 'dynamic', this will return -1.0.  Otherwise, it returns a number 0-255 for the red value of the color of this LinkType.

## LinkType.green

>Retrieve the green value.

```python
voiced.color = 'dynamic'
print voiced.green
```

```json
-1.0
```

```python
voiced.color = (255, 0, 222)
print voiced.green
```

```json
0
```

>Set the green value.

```python
voiced.green = 43
```

If *color* is set to 'dynamic', this will return -1.0.  Otherwise, it returns a number 0-255 for the green value of the color of this LinkType.

## LinkType.blue

>Retrieve the blue value.

```python
voiced.color = 'dynamic'
print voiced.blue
```

```json
-1.0
```

```python
voiced.color = (255, 0, 222)
print voiced.blue
```

```json
222
```

>Set the blue value.

```python
voiced.blue = 15
```

If *color* is set to 'dynamic', this will return -1.0.  Otherwise, it returns a number 0-255 for the blue value of the color of this LinkType.

## LinkType.as_dict

```python
print g.link_type("Authored").as_dict
```

```json
{
    "name": "Authored",
    "min": 1.0,
    "max": 2.0,
    "color": [255, 0, 0],
    "units": "units",
    "unit_scale": 1,
    "icon": "image/pen.png"
}
```

Returns a JSON-ready object describing the LinkType.  Editing this dictionary will not edit the LinkType.

# Link

>Construct a Link and add it to the Graph.

```python
l = psynth.Link({
    'graph': g,
    'origin': billy_west,
    'terminus': dr_zoidberg,
    'link_type': g.link_type('voiced'),
    'data' : {
        'audio': 'audio/zoidberg_quote.mp3'
    }
})
g.add_link(l)
```

>However, if you use the Graph.add_link method directly, you can omit the *graph* parameter, and do the whole thing in a single declaration.

```python
l = g.add_link({
    'origin': billy_west,
    'terminus': dr_zoidberg,
    'link_type': g.link_type('voiced'),
    'data' : {
        'audio': 'audio/zoidberg_quote.mp3'
    }
})
```

>Or, you can go even less verbose and use the Node.link_to method:

```python
l = billy_west.link_to(dr_zoidberg, {
    'link_type': g.link_type('voiced'),
    'data' : {
        'audio': 'audio/zoidberg_quote.mp3'
    }
})
```

Links are a fundamental component of Graphs.  The connect Nodes to each other, and have type, described by LinkType objects.  Links can be directed, undirected, or bidirected.  Links of different directness can co-exist in a single Graph.

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
required | graph | The Graph object this Link will belong to. | 
required | origin | The Node object this Link should originate from. | 
required | terminus | The Node object this Link should terminate at. |
required | link_type | A LinkType object.  All Links must have a LinkType. |
*optional* | label | The text that will be shown on this Link when displayed | ''
*optional* | value | The value of this Link. Must be between the min and max values set by the LinkType | link_type.min
*optional* | directed | 'directed', 'bidrected', 'undirected' | 'directed'
*optional* | uid | A unique identifier for this Link | UUID
*optional* | data | An object filled with various key-value pairs associated with the Link | {}
*optional* | comments | A list of any kind of comment objects you feel like using. | []

## Link.uid

>Access the Link's uid.

```python
print l.uid
```

```json
"4cdb03a6-6c1f-4766-9616-64ec2a9d2dd7"
```

A String which uniquely identifies this Link within the Graph.  By default, PsynthDB uses UUID4 Global Unique Identifiers.  There are certainly use cases in which a different uid scheme could be appropriate.  IP addresses, MAC addresses, Phone Numbers, etc.

## Link.origin

>Get the label of a Link's origin Node

```python
print l.origin.label
```

```json
"Billy West"
```

>Change the origin Node of a Link

```python
l.origin = g.nodes[0]
```

The Node object which is this Link's origin.

## Link.terminus

>Get the label of a Link's terminus Node

```python
print l.terminus.label
```

```json
"Dr. Zoidberg"
```

>Change the terminus Node of a Link

```python
l.terminus = phillip_j_fry
```

The Node object which is this Link's terminus.

## Link.link_type

>Get the LinkType of a Link

```python
print l.link_type.name
```

```json
"voiced"
```

>Set the LinkType of a link

```python
l.link_type = g.link_type('paid')
```

The LinkType object which describes this Link.  Changing the LinkType will set new constraints for the value of this Link.

## Link.directed

>Retrieve directedness of Link

```python
print l.directed
```

```json
"directed"
```

>Set directness of Link

```python
l.directed = "bidirected"
```

The directness of the Link. Links can be 'directed', 'undirected', or 'bidirected'.  Links are 'directed' by default.

Type | Description
-----|-------------
directed | This Link can be traversed in only one direction, from Origin to Terminus.
undirected | This Link expresses a connection between Nodes for which direction is not a useful concept.
bidirected | This Link can be traversed in both directions, and represents a symmetrical directed relationship between Origin and Terminus, and between Terminus and Origin.

## Link.value

>Set a Link's value

```python
l.value = 2
```

>Retrieve a Link's value

```python
print l.value
```

```json
2.0
```

The numerical value of the Link, such that Link.link_type.min <= Link.value <= Link.link_type.max.  If a value is set that is outside of the boundaries set by the LinkType, it will be constrained appropriately.

## Link.label

>Set a Link's label

```python
l.label = "really hammed it up"
```

>Retrieve a Link's label

```python
print l.label
```

```json
"really hammed it up"
```

This is text that will accompany the Link when displayed.

## Link.data

>Retrieve audio information from a Link:

```python
clip = l.data['audio']
```

>Set the audio information for a Link:

```python
l.data['audio'] = 'audio/zoidberg_quote.mp3'
```

Store and retrieve information about Links.  Supports fully arbitrary key-value pairing. If the graph is to be exposed RESTfully, it is recommended to use Strings for keys, and JSON encodable values.

## Link.comments

>Leave a plaintext comment on a Link:

```python
l.comments.append("Really stellar job here.")
```

>Leave a comment object on a Link:

```python
from datetime import datetime

l.comments.append({
    'user': 'graph user',
    'time': datetime.now(),
    'content': "Really stellar job here.",
    'replies': []
})
```

>Retrieve the most recent comment left on a Link:

```python
print l.comments[-1]
```

Store and retrieve comments that have been left on this Link.  This is an ordered list.  PsynthDB doesn't include a comment object, but you can define whatever one you'd like and use it here.

## Link.parallel

>Get the label of every link parallel to *l*

```python
for p in l.parallel:
    print p.label
```

Returns a list of all Links which are parallel to this Link. This is direction independent, so it will include all Links which share the Origin and Terminus Nodes of this Link.

## Link.as_dict

```python
print l.as_dict
```

```json
{
    "label": "2014",
    "uid": "c695170b-6709-4406-adb4-9709bb014c05",
    "directed": "directed",
    "value": 1.0,
    "comments": [],
    "data": {},
    "type": "Authored",
    "origin": "0d780b67-e6e1-4a73-b0f9-bfa5d97d2eba",
    "terminus": "f38d625d-5927-40f2-baf9-4c2121d6b502"
}
```

Returns a JSON-ready object describing the Link.  Editing this dictionary will not edit the Link.

# NodeDataMap

The NodeDataMap is essentially a dictionary that only accepts Nodes as keys, and has a couple extra features.

## NodeDataMap.graph

>Get a NodeDataMap's Graph

```python
print data_map.graph.name
```

This returns the Graph that this NodeDataMap describes.

## NodeDataMap.normalize

>See the Pagerank of a Graph as Node alpha:

```python
pr = g.centrality.pagerank()

g.save_image({
    'filename': 'TestMap_sfdp_pr_1.png',
    'node_alpha': pr.normalized()
})
```

This returns a version of this NodeDataMap normalized with values between 0 and 1.

## NodeDataMap.uid_keyed

>Get the node betweenness of a Graph...

```python
n_between, l_between = g.centrality.betweenness()
```

>And return it as a JSON string

```python
return json.dumps(n_between.uid_keyed())
```

Returns this NodeDataMap as a regular dictionary keyed with Node UIDs.  This assists in RESTful exposition.

# LinkDataMap

The LinkDataMap is essentially a dictionary that only accepts Links as keys, and identifies the Graph it belongs to.

## LinkDataMap.graph

>Get a LinkDataMap's Graph

```python
print data_map.graph.name
```

This returns the Graph that this LinkDataMap describes.

## LinkDataMap.normalize

>See the Betweenness of a Graph as Node and Link alpha:

```python
nb, lb = g.centrality.betweenness()

g.save_image({
    'filename': 'TestMap_sfdp_pr_1.png',
    'node_alpha': nb.normalized(),
    'link_alpha': lb.normalized()
})
```

This returns a version of this LinkDataMap normalized with values between 0 and 1.

## LinkDataMap.uid_keyed

>Get the link betweenness of a Graph...

```python
n_between, l_between = g.centrality.betweenness()
```

>And return it as a JSON string

```python
    return json.dumps(l_between.uid_keyed())
```

Returns this LinkDataMap as a regular dictionary keyed with Link UIDs.  This assists in RESTful exposition.

# random_graph

>Generate a random Graph with 200 Nodes and 6000 Links of 2 LinkTypes.

```python
node_size = 3
link_thickness = 3

g = psynth.random_graph({
    'owner': 'me',
    'name': 'random_graph',
    'num_nodes': 200,
    'num_link_types': 2,
    'num_links': 600
})

for n in g.nodes:
    n.radius = node_size

g.draw.sfdp({
    'set_position': True
})

g.save_image({
    'filename': 'images/random_graph_200_600.png',
    'width': 600,
    'max_link_thickness': link_thickness
})
```

Generates a random Graph.  Accepts all of a Graph's normal parameters, as well as:

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
required | num_nodes | The number of Nodes to add to the Graph |
required | num_link_types | The number of LinkTypes to add to the Graph |
required | num_links | The number of Links to add to the Graph |

![alt text](images/random_graph_200_600.png "A random Graph with an SFDP layout")

# price_network

>Generate a Price Network with 300 Nodes.

```python
node_size = 3
link_thickness = 3

g2 = psynth.price_network({
    'owner': 'me',
    'name': 'random_graph',
    'num_nodes': 200
})

for n in g2.nodes:
    n.radius = node_size

g2.draw.sfdp({
    'set_position': True
})

g2.save_image({
    'filename': 'images/price_network_200.png',
    'width': 600,
    'max_link_thickness': link_thickness
})
```

Generates a Price Network.  Accepts all of a Graph's normal parameters, as well as:

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
required | num_nodes | The number of Nodes to add to the Graph |

![alt text](images/price_network_200.png "A Price Network with an SFDP layout")

# triangulation

>Generate a delauney triangulation graph with 200 Nodes.

```python
node_size = 3
link_thickness = 3

g3 = psynth.triangulation({
    'owner': 'me',
    'name': 'random_graph',
    'num_nodes': 200,
    'type': 'delaunay'
})

for n in g3.nodes:
    n.radius = node_size

g3.save_image({
    'filename': 'images/delaunay_200.png',
    'width': 600,
    'max_link_thickness': link_thickness
})
```

>The same Graph with the SFDP layout

```python
g3.draw.sfdp({
    'set_position': True
})

g3.save_image({
    'filename': 'images/delaunay_sfdp_200.png',
    'width': 600,
    'max_link_thickness': link_thickness
})
```

Generates a Triangulation Graph.  Accepts all of a Graph's normal parameters, as well as:

Required | Paramater | Description | Default
--------- | ----------- | -------- | --------
*optional* | num_nodes | The number of Nodes to add to the Graph |
*optional* | type | The type of triangulation to do. "simple" or "delaunay" | "simple"

Delaunay Triangulation on random points
![alt text](images/delaunay_200.png "A Delaunay Triangulation Graph")

The same Graph with an SFDP Layout.
![alt text](images/delaunay_sfdp_200.png "A Delaunay Triangulation Graph with an SFDP layout")


