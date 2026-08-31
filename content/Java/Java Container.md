
### Collection
: root interface of Set
- **ArrayList**
```java
List<String> list = new ArrayList<>();
String item;
list.size();
list.add(item); // or list.add(0, item);
list.remove(0); // or list.remove(item);
list.set(0, item);
list.indexOf(item);
list.contain(item);
list.isEmpty();
list.clear();

for(Integer item: list){ }
list.forEach();
```
- **LinkedList**
```java
LinkedList<String> list = new LinkedList<>();
String item;
list.addFirst(item);
list.addLast(item);
list.getFirst(item);
list.getLast(item);
list.removeFirst(item);
list.removeLast(item);
list.peek(item); // is list.head
```

- **Stack/Queue**
```java
Deque<String> stack = new ArrayDeque<>(); // or new Stack<>();
String item;
stack.push(item);
stack.pop();
stack.peek(); // is stack.top
stack.isEmpty();
```

```java
Deque<String> queue = new ArrayDeque<>();
String item;
queue.offer(item);
queue.poll();
queue.peek();
```
- **HashSet**
```java
Set<String> seta = new HachSet<>(List.of('a', 'b', 'c'));
seta.add('a');
seta.remove('a');
seta.contains('a');
set.size();
seta.isEmpty();
for(String item: set){ }
```

**Map**
- **HashMap**
```java
Map<String, Integer> map = new HashMap<>();

map.put('a', 1);// map.putIfAbsent('a', 0);
map.get('a'); 
map.containsKey('a');
map.remove('a');
map.size();
map.isEmpty(); 
```

- **TreeMap**
```java
TreeMap<String, Integer> map = new TreeMap<>();
map.put('a', 1);
map.put('b', 2);
map.put('c', 3);

map.firstKey(); // smallest key: 'a'
map.lastKey(); // largest key: 'c'
```