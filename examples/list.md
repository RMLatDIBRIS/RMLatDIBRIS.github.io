---
layout: default
title: Verification of lists
show_downloads: false
---
# Verification of lists

The specifications are based on the following basic event types:

* `insert(index,elem)` element `elem` has been inserted in the list at index `index`;
* `remove(index,elem)` element `elem` has been removed from the list at index `index`;
* `get(index,elem)` element `elem` has been retrieved (but not removed) from the list at index `index`;
* `size(size)` `size` has been returned as the size of the list.

Indexes are assumed to start from 0.

## A simple specification which only checks correct use of indexes

```js
// Additional events for CheckIndex
insert_in_bounds(size) matches insert(index,_) with index >= 0 && index <= size;
remove_in_bounds(size) matches remove(index,_) with index >= 0 && index < size;
get_in_bounds(size) matches get(index,_) with index >= 0 && index < size;
get_size(size) matches size(size)|get_in_bounds(size);

CheckIndex<size> =
    get_size(size)* (insert_in_bounds(size) CheckIndex<size+1> \/ remove_in_bounds(size) CheckIndex<size-1>);
```

## Full specification

```js
// Additional events for Main
add_rm_get matches insert(_,_)|remove(_,_)|get(_,_);

// Additional events for CheckIndex
insert_in_bounds(size) matches insert(index,_) with index >= 0 && index <= size;
remove_in_bounds(size) matches remove(index,_) with index >= 0 && index < size;
get_in_bounds(size) matches get(index,_) with index >= 0 && index < size;
get_size(size) matches size(size)|get_in_bounds(size);

// Additional events for CheckElem
not_insert not matches  insert(_,_);

// Additional events for GetElem
increased(i) matches insert(index,_) with index <= i;
decreased(i) matches remove(index,_) with index < i;
irrelevant_modification(i) matches insert(index,_) | remove(index,_) with index > i;
irrelevant_get(i) matches get(index,_)  with index != i;
irrelevant(i) matches irrelevant_modification(i) | irrelevant_get(i);
					
Main = (CheckIndex<0> /\ add_rm_get >> CheckElem)!;
CheckIndex<size> =
    get_size(size)* (insert_in_bounds(size) CheckIndex<size+1> \/ remove_in_bounds(size) CheckIndex<size-1>);
CheckElem =
    not_insert* {let index,elem; insert(index,elem) (GetElem<index,elem> /\ CheckElem)};
GetElem<index,elem> =
    (irrelevant(index) \/ get(index,elem)) GetElem<index,elem> \/ 
    increased(index) GetElem<index+1,elem> \/
    decreased(index) GetElem<index-1,elem> \/
    remove(index,elem) all;
```

### Credits
[Luca Ciccone](https://www.dibris.unige.it/ciccone-luca) collaborated to the development of these examples
