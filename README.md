# :herb: ng2-tree

[![Travis](https://img.shields.io/travis/valor-software/ng2-tree.svg?style=flat-square)](https://travis-ci.org/valor-software/ng2-tree)
[![Codecov](https://img.shields.io/codecov/c/github/valor-software/ng2-tree.svg](https://codecov.io/github/valor-software/ng2-tree)

<!-- TOC -->

- [:clapper: Usage](#clapper-usage)
- [:eyes: Demo](#eyes-demo)
- [:wrench: API](#wrench-api)
  - [tree](#tree)
  - [[tree]](#tree)
    - [Load children asynchronously](#load-children-asynchronously)
    - [Configure node via TreeModelSettings](#configure-node-via-treemodelsettings)
  - [[settings]](#settings)
  - [`Tree` class](#tree-class)
  - [events (nodeMoved, nodeSelected, nodeRenamed, nodeRemoved, nodeCreated, nodeExpanded, nodeCollapsed)](#events-nodemoved-nodeselected-noderenamed-noderemoved-nodecreated-nodeexpanded-nodecollapsed)
    - [NodeSelectedEvent](#nodeselectedevent)
    - [NodeMovedEvent](#nodemovedevent)
    - [NodeRemovedEvent](#noderemovedevent)
    - [NodeCreatedEvent](#nodecreatedevent)
    - [NodeRenamedEvent](#noderenamedevent)
    - [NodeExpandedEvent](#nodeexpandedevent)
    - [NodeCollapsedEvent](#nodecollapsedevent)
- [Changes that should be taken into account in order to migrate from __ng2-tree V1__ to __ng2-tree V2__](#changes-that-should-be-taken-into-account-in-order-to-migrate-from-__ng2-tree-v1__-to-__ng2-tree-v2__)
- [:bulb: Want to help?](#bulb-want-to-help)

<!-- /TOC -->

## :clapper: Usage
Ok, let's start with an installation - all you need to do is:

`npm install --save ng2-tree`

Now when you have `ng2-tree` installed, you are in a few steps from having tree in your application:

1. Add the `TreeModule` to your application's module `imports` section:

```typescript
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { BrowserModule } from '@angular/platform-browser';
import { TreeModule } from 'ng2-tree';

@NgModule({
  declarations: [MyComponent],
  imports:      [BrowserModule, TreeModule],
  bootstrap:    [MyComponent]
})
export class MyModule {
}
```

2. As soon as the previous step is done we need to give ng2-tree a model to render - this can be accomplished by populating its `[tree]` attribute with an object that conforms to the `TreeModel` interface (see [API](#wrench-api)):

```typescript
// 1 - import required classes and interfaces
import { TreeModel } from 'ng2-tree';

@Component({
  selector: 'myComp',
  // 2 - set [tree] attribute to tree object
  template: `<tree [tree]="tree"></tree>`
})
class MyComponent {
  // 3 - make sure that tree object conforms to the TreeModel interface
  public tree: TreeModel = {
    value: 'Programming languages by programming paradigm',
    children: [
      {
        value: 'Object-oriented programming',
        children: [
          {value: 'Java'},
          {value: 'C++'},
          {value: 'C#'}
        ]
      },
      {
        value: 'Prototype-based programming',
        children: [
          {value: 'JavaScript'},
          {value: 'CoffeeScript'},
          {value: 'Lua'}
        ]
      }
    ]
  };
}
```

3. Apart from that, in order to have usable tree in the browser, you need to add **ng2-tree styles** which you can find in your `node_modules/ng2-tree/styles.css`
4. And finally, I suppose, you'd want to listen to events generated by ng2-tree (for a full list of supported events look at the [API](#wrench-api)). No problem, this is also easy to do - for example let's add a listener for `node was selected` kind of events:

```typescript
// 1 - import required classes and interfaces
import { TreeModel, NodeEvent } from 'ng2-tree';

@Component({
  selector: 'myComp',
  // 2 - listent for nodeSelected events and handle them
  template: `<tree [tree]="tree" (nodeSelected)="logEvent($event)"></tree>`
})
class MyComponent {
  public tree: TreeModel = { ... };

  // 3 - print caught event to the console
  public logEvent(e: NodeEvent): void {
    console.log(e);
  }
}
```
Voila! That's pretty much it - enjoy :blush:

## :eyes: Demo
Feel free to examine the [demo](https://valor-software.github.io/ng2-tree) and its [sources](demo/) to find out how things are wired.
Also there is [another demo built with Angular CLI](https://github.com/rychkog/ng2-tree-demo).

## :wrench: API

Here is the fully stuffed *tree* tag that you can use in your templates:

```html
    <tree
      [tree]="tree"
      [settings]="settings"
      (nodeRemoved)="handleRemoved($event)"
      (nodeRenamed)="handleRenamed($event)"
      (nodeSelected)="handleSelected($event)"
      (nodeMoved)="handleMoved($event)"
      (nodeCreated)="handleCreated($event)"
      (nodeExpanded)="handleExpanded($event)"
      (nodeCollapsed)="handleCollapsed($event)">
    </tree>
```

Let's go through every element of this structure one by one.

### tree

`tree` is the selector for `TreeComponent` which is bundled into `TreeModule`:

### [tree]

`tree` has a `[tree]` attribute which needs to be populated with an object implementing `TreeModel` interface. You can import this interface like below:

```typescript
import { TreeModel } from 'ng2-tree';
```

Here is the definition of the `TreeModel` interface:

```typescript
interface TreeModel {
  value: string | RenamableNode;
  id: string | number;
  children?: Array<TreeModel>;
  loadChildren?: ChildrenLoadingFunction;
  settings?: TreeModelSettings;
}
```

As you can see - object that conforms to this interface has a recursive nature, example can be seen below:

```typescript
{
    value: 'Programming languages by programming paradigm',
    children: [
      {
        value: 'Object-oriented programming',
        children: [
          {value: 'Java'},
          {value: 'C++'},
          {value: 'C#'}
        ]
      },
      {
        value: 'Prototype-based programming',
        children: [
          {value: 'JavaScript'},
          {value: 'CoffeeScript'},
          {value: 'Lua'}
        ]
      }
    ]
  }
```

Property `value` can be of type `string` or `RenamableNode`.
`RenamableNode` gives you an additional control over the way node is renamed and rendered (by rendered I mean its text representation). Here is the definition of the `RenamableNode` interface:

```typescript
interface RenamableNode {
  // This method will be invoked in order to apply new value to this kind of node
  setName(name: string): void;

  // This method will be invoked in order to get a text for rendering as a node value
  toString(): string;
}
```

Here is an example of such a node in the `TreeModel` object:

```typescript
{
    value: 'Programming languages by programming paradigm',
    children: [
      {
        value: 'Object-oriented programming',
        children: [
          {
            // I am a RenamableNode. Yeah, that's me :)
            value: <RenamableNode>{
              name: 'Java',
              setName(name: string): void {
                this.name = name;
              },
              toString(): string {
                return this.name;
              }
            }
          },
          {value: 'C++'},
          {value: 'C#'}
        ]
      },
      {
        value: 'Prototype-based programming',
        loadChildren: (callback) => {
          setTimeout(() => {
            callback([
              {value: 'JavaScript'},
              {value: 'CoffeeScript'},
              {value: 'TypeScript'}
            ]);
          }, 5000);
        }
      }
    ]
  };
```

#### Load children asynchronously

Another worth noting thing is `loadChildren`. This function on `TreeModel` allows you to load its __children asynchronously__.

```typescript
{
  value: 'Prototype-based programming',
  loadChildren: (callback) => {
    setTimeout(() => {
      callback([
        {value: 'JavaScript'},
        {value: 'CoffeeScript'},
        {value: 'TypeScript'}
      ]);
    }, 5000);
  }
}
```

Node that defines this function is collapsed by default. At the moment of clicking 'Expand' arrow it starts loading its children by calling given function.
If `loadChildren` function is given to the node - `children` property is ignored. For more details - have a look at the [Demo](#eyes-demo).

#### Configure node via TreeModelSettings

Apart from that `TreeModel` interface has an optional field called `settings` of type `TreeModelSettings`.

Here is an example of its usage:

```typescript
{
  value: 'Prototype-based programming',
  settings: {
    'static': true,
    'rightMenu': true,
    'leftMenu': true,
    'cssClasses': {
      'expanded': 'fa fa-caret-down fa-lg',
      'collapsed': 'fa fa-caret-right fa-lg',
      'leaf:': 'fa fa-lg',
      'empty': 'fa fa-caret-right disabled'
    },
    'templates': {
      'node': '<i class="fa fa-folder-o fa-lg"></i>',
      'leaf': '<i class="fa fa-file-o fa-lg"></i>',
      'leftMenu': '<i class="fa fa-navicon fa-lg"></i>'
    }
  },
  children: [
    {value: 'JavaScript'},
    {value: 'CoffeeScript'},
    {value: 'Lua'}
  ]
}
```

* `static` - Boolean - This option makes it impossible to drag a tree or modify it in a some way, though you still can select nodes in the static tree and appropriate events will be generated.
* `rightMenu` - Boolean - This option allows you to activate (true, by default) or deactivate (false) right menu when clicking with right button of a mouse.
* `leftMenu` - Boolean - This option allows you to activate (true) or deactivate (false, by default) left menu.
* `cssClasses` - Object:
  * `expanded` - String - It specifies a css class (or classes) for an item which represents expanded state of a node. The item is clickable and it transitions the node to the collapsed state
  * `collapsed` - String - It specifies a css class (or classes) for an item which represents collapsed state of a node. The item is clickable and it transitions the node to the expanded state
  * `leaf` - String - It specifies a css class (or classes) for an item which represents a node without an option to expand or collapse - in other words: a leaf node.
  * `empty` - String - Node is considered empty when it has no children. Once this condition is satisfied - appropriate css class will be applied to the node.
* `templates` - Object:
  * `node` - String - It specifies a html template which will be included to the left of the node's value.
  * `leaf` - String - It specifies a html template which will be included to the left of the leaf's value.
  * `leftMenu` - String - It specifies a html template to the right of the node's value. This template becomes clickable and shows a menu on node's click.

All options that's defined on a `parent` are automatically applied to children. If you want you can override them by `settings` of the child node.

### [settings]

Object that should be passed to `[settings]` must be of type [`Ng2TreeSettings`](src/tree.types.ts). This attribute is __optional__. Right now only one setting is available in there - `rootIsVisible`. This setting allows you to make a root node of the tree _invisible_:

```typescript
const treeSettings: Ng2TreeSettings = {
  rootIsVisible: false
}
```

By default `rootIsVisible` equals to `true`

### `Tree` class

Also in the next section you'll be reading about events generated by the `ng2-tree`. And here [Tree](src/tree.ts) class comes in handy for us, because its instances propagated with event objects. Under the hood `ng2-tree` wraps a `TreeModel` provided by the user in `Tree`. And `Tree` in turn has lots of useful methods and properties (like `parent`, `hasChild()`, `isRoot()` etc.)

### events (nodeMoved, nodeSelected, nodeRenamed, nodeRemoved, nodeCreated, nodeExpanded, nodeCollapsed)

Here is the diagram that shows tree events' hierarchy

![tree events hierarchy](media/tree-events-hierarchy.png)

`NodeEvent` is the root of the tree events' hierarchy. It defines property `node` that contains a receiver of the event action (`node` is an instance of the `Tree` class).

`NodeDestructiveEvent` is the parent for all events that cause changes to the structure of the tree or to the node's value.

#### NodeSelectedEvent

You can subscribe to the `NodeSelectedEvent` by attaching listener to the `(nodeSelected)` attribute

```html
    <tree
      [tree]="tree"
      (nodeSelected)="handleSelected($event)">
    </tree>
```

`NodeSelectedEvent` has just one property `node` which contains a `Tree` object representing selected node.

```typescript
{node: <Tree>{...}}
```

#### NodeMovedEvent

You can subscribe to `NodeMovedEvent` by attaching listener to `(nodeMoved)` attribute

```html
    <tree
      [tree]="tree"
      (nodeMoved)="handleMoved($event)">
    </tree>
```

`NodeMovedEvent` has two properties `node` and `previousParent` both of which contain `Tree` objects:

- `node` contains a moved node;
- `previousParent` contains a previous parent of the moved node;

```typescript
{node: <Tree>{...}, previousParent: <Tree>{...}}
```

#### NodeRemovedEvent

You can subscribe to `NodeRemovedEvent` by attaching listener to `(nodeRemoved)` attribute

```html
    <tree
      [tree]="tree"
      (nodeRemoved)="handleRemoved($event)">
    </tree>
```

`NodeRemovedEvent` has a `node` property, which contains removed node (of type `Tree`).

```typescript
{node: <Tree>{...}}
```

#### NodeCreatedEvent

You can subscribe to `NodeCreatedEvent` by attaching listener to `(nodeCreated)` attribute

```html
    <tree
      [tree]="tree"
      (nodeCreated)="handleCreated($event)">
    </tree>
```

`NodeCreatedEvent` has a `node` property of type `Tree`, which contains a created node.

```typescript
{node: <Tree>{...}}
```

#### NodeRenamedEvent

You can subscribe to `NodeRenamedEvent` by attaching listener to `(nodeRenamed)` attribute

```html
    <tree
      [tree]="tree"
      (nodeRenamed)="handleRenamed($event)">
    </tree>
```

`NodeRenamedEvent` has three properties:

- `node` contains node that was renamed (instance of `Tree`).
- `oldValue` contains a value, that node used to have (it might be `string` or `RenamableNode`)
- `newValue` contains a new value of the node (it might be `string` or `RenamableNode`)

```typescript
{
  node: <Tree>{...},
  oldValue: <string|RenamableNode>{...},
  newValue: <string|RenamableNode>{...}
}
```

#### NodeExpandedEvent

You can subscribe to `NodeExpandedEvent` by attaching listener to `(nodeExpanded)` attribute, this event wont fire on initial expansion

```html
    <tree
      [tree]="tree"
      (nodeExpanded)="handleExpanded($event)">
    </tree>
```

`NodeExpandedEvent` has a `node` property of type `Tree`, which contains an expanded node.

```typescript
{node: <Tree>{...}}
```

#### NodeCollapsedEvent

You can subscribe to `NodeCollapsedEvent` by attaching listener to `(nodeCollapsed)` attribute

```html
    <tree
      [tree]="tree"
      (nodeCollapsed)="handleCollapsed($event)">
    </tree>
```

`NodeCollapsedEvent` has a `node` property of type `Tree`, which contains a collapsed node.

```typescript
{node: <Tree>{...}}
```

## Changes that should be taken into account in order to migrate from __ng2-tree V1__ to __ng2-tree V2__
- Events were reworked:
  - In V1 all events that were inherited from NodeDestructiveEvent used to have property `parent`. It's not the case anymore. If you need a parent you should get it from `node` in event object like `node.parent`;
  - All events used to have `node` property of type `TreeModel`. Now `node` is of type [Tree](#tree-class) (as well as `node.parent`);
  - `NodeMovedEvent` now has property `previousParent`, which contains tree in which moved node used to be.
- CSS styles in __ng2-tree V2__ are distributed as separate file which you can find in `node_modules/ng2-tree/styles.css`. That allows you to override ng2-tree styles more easely.

## :bulb: Want to help?

I am very appreciate for your ideas, proposals and found bugs which you can put in [github issues](https://github.com/valor-software/ng2-tree/issues). Thanks in advance!

**P.S.** If you find it hard going through documentation, please, let me know which parts of it was difficult to grasp and I will improve them.
