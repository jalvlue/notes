
|Name|Store Operation(s)|Primary Retrieval Operation|Retrieve By|
|---|---|---|---|
|List|`add(key)`, `insert(key, index)`|`get(index)`|index|
|Map|`put(key, value)`|`get(key)`|key identity|
|Set|`add(key)`|`containsKey(key)`|key identity|
|PQ|`add(key)`|`getSmallest()`|key order (aka key size)|
|Disjoint Sets|`connect(int1, int2)`|`isConnected(int1, int2)`|two integer values|

