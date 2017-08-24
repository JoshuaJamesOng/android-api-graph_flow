# GraphFlow

[![Build Status](https://travis-ci.org/saantiaguilera/android-api-graph_flow.svg?branch=develop)](https://travis-ci.org/saantiaguilera/android-api-graph_flow) [![Download](https://api.bintray.com/packages/saantiaguilera/maven/com.saantiaguilera.graphflow.core/images/download.svg) ](https://bintray.com/saantiaguilera/maven/com.saantiaguilera.graphflow.core/_latestVersion)

GraphFlow is a lightweight library for designing 'logic based' UI flows. You create a graph of nodes logically connected where each node represents a UI renderable object (fragment/view/whatever).
 
![](https://cdn-images-1.medium.com/max/1200/1*fCdB8ltfNn-OaXzC7ULRDQ.png) 
 
### Minimum api level

Per module:
- :core: 9
- :fragments: 9
- :views: 14
- :conductor: 16

### This library was heavily influenced by a [Reddit Post](https://www.reddit.com/r/androiddev/comments/6geu04/mastering_viewpager_with_directed_acyclic_graph/)

## Setup

Add dependencies in your `build.gradle`
```groovy
dependencies {
  // Core library.
  compile "com.saantiaguilera.graphflow:core:<latest_version>"
  
  // For using it with fragments. Brings support-fragments
  compile "com.saantiaguilera.graphflow:fragments:<latest_version>"
  
  // For using it with views
  compile "com.saantiaguilera.graphflow:views:<latest_version>"
  
  // For using it with conductor
  compile "com.saantiaguilera.graphflow:conductor:<latest_version>"
}
```

## Usage

1. Create a `Graph` with nodes and connections. Lets say we have a signup application where:

    **a.** You set your name
  
    **b.** You set your age
  
    **c.** If you are <18 you validate something, else skips this
  
    **d.** You set your email

```Java
  // For this example I'll be using the View component (using fragments its the same but MView.class -> MFragment.class
  // Create the type of graph, for this example we use a shipped in DirectedAcyclicGraph
  Graph graph = new DirectedAcyclicGraph();

  // Create the name input node
  // Always pick it, its the root?
  Node nameNode = new Node(NameInputView.class, NodeSelector.ALWAYS);
  
  // Create the age input node
  // Always pick it if coming from the name.
  Node ageNode = new Node(AgeInputView.class, NodeSelector.ALWAYS);
  
  // Create the under 18 input node
  Node underEighteenNode = new Node(UnderEighteenView.class, new NodeSelector() {
    @Override
    public boolean select(@Nullable final Bundle args) {
      return args.getInteger("age") < 18;
    }
  });  

  // Create the email input node
  // Always pick, doesnt matter from where you come
  Node emailNode = new Node(EmailInputView.class, NodeSelector.ALWAYS);

  // Add the nodes to the graph, the order doesnt mind, when connected the shape will be formed.
  graph.add(emailNode);
  graph.add(underEighteenNode);
  graph.add(ageNode);
  graph.add(nameNode);

  // Connect them as we want. We will do it like this:
  // Name --> age --> underEighteen --> email
  //              \_________________/
  graph.connect(name, age);
  graph.connect(age, underEighteen);
  graph.connect(age, email);
  graph.connect(underEighteen, email);
  
  // Graph done!
```

2. Create a node switcher! Node switcher is the class in charge of switching a node for another and make them "rendereable"

```Java
  // Following the example, we will use a NodeViewSwitcher. If using fragments it would be the same with NodeFragmentSwitcher
  NodeSwitcher<View> nodeSwitcher = new NodeViewSwitcher(activityContext, R.id.viewgroup_container_id);
```

3. Finally, create a router and start using it!

```Java
  router = Router.<View>create()
    .with(graph)
    .switcher(nodeSwitcher)
    .build();
```

## Router

The router is the class in charge of managing the whole graph flow. The router provides a lot of useful methods:

- _fromRoot_: Starts the graph from the root node. The root node is considered the only point with no incoming edges and >1 outgoing edges.

- _addOnNodeCommitListener_: Adds a listener that will be triggered everytime we move from one node to another.

- _removeOnNodeCommitListener_: Removes a listener, since they are kept as **strong references**.

- _next_: Moves to the immediate node in the graph that fulfills the bundle params

- _back_: Moves to the previous node

- _jump_: Jumps to a given node

## Proguard

This library supports proguard transitively, no need to add extra rules :)

## Contributing

Feel free to submit me issues or fork it and do pull requests with changes!
