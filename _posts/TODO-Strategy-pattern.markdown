---
layout: post
title:  "Stategy Pattern"
date:   ?? 
categories: OO Pattern development
---

A brain dump of the Strategy pattern and a real world use case. Since Java is one of the more used OO Languages that's what the example is in, however it applies to other OO languages as well. This pattern was introduced in the original GOF Design Patterns book. 

## Why?

The Strategy Pattern is a way for a program to select a piece of code to execute at runtime, rather than being explicitly tied to the implementation at compile time. Generally this is useful where an algorithm can be selected for use based on some property of a piece of data or data structure.

## Example

This example uses the strategy pattern to apply logic based on differing attributes of an Object, which enables the use of a singular algorithm implmentation. The requirement is to synchronise the attributes of an **Account** or **Header** Object, which implement a shared interface **Child**, I'll refer to both of these implmentations as a Child. The Child attributes may have become stale in a dependant downstream system (regardless of this being a bad design decision). Account and Header objects make up a tree structure together, where each node is an account or header. Header Objects can include references to Child Accounts or Headers, and Accounts cannot include references to other nodes.

**Child**

```
public interface Child {

  String getType();

  boolean isAccount();

  Stream<Child> flatten();

  void normalize();
}
```

**Account**

```
public Class Account implements Child {
  private String id;
  
  public String getId() {
    return this.id;
  }
}
```

**Header**

```
public Class Header implements Child {
  private String number;
  
  public String getNumber() {
    return this.number;
  }
}
```

The strategy pattern is implemented as a way to lookup a Child from a structure in order to synchronise the downstream system. Because each implementation of the Child uses a different attribute as an identifier, the lookup strategy is determined by the Implementation. The high level process for synchronisation is:

1. Retrieve existing tree structure to synchronise with the Upstream System
2. Retrieve upstream data that will be used to synchronise
3. Iterate through the tree and synchronise each node. The node will select which attribute to check the synchronisation source at runtime based on the node class
4. Synchronise the node attributes

The Strategy Implementation uses the following Interface which each lookup Strategy implements:

**LookupStrategy**

```
public interface LookupStrategy {
  String get(Child child);
}
```

**ById**

```
public class ById implements LookupStrategy {
  @Override public String get(Child child) {
    return account.getId();
  }
}
```

**ByNumber**
```
@Override public String get(Child child) {
  return account.getNumber();
}
```

Based on this implementation, the Interfact method can be used to inject, at runtime, the correct Child's lookup attribute. Here is the usage of this in the final synchronisation method. In the `updateView` method, the `accountsView` argument represents the structure to be synchronised, while the `accounts` is the source data to synchronise the target.

```
public void updateView(AccountsView accountsView, AccountList accounts) {

  Map<String, AccountInfo> accountIdToInfoMap = accounts != null ? accounts.createIdToAccountsMap() : new HashMap<>();

  accountsView.getChildren().forEach(child -> accountInfoSynchronizer.synchronizeNode(child, accountIdToInfoMap, LookupStrategy.get(child)));

  accountClassificationSynchronizer.removeInvalidAccountsFromClassification(accountsView, accountIdToInfoMap);
  addMissingAccountsToView(accountsView, accountIdToInfoMap);
  checkAndAddHeaderIDs(accountsView);
}
```
