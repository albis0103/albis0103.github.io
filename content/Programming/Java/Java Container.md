
### Collection
: root interface of Set
- **ArrayList**
```java
List<String> list = new ArrayList<>(List.of('a', 'b'));
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
stack.peek(); // is stack.top && pop();
stack.isEmpty();
```

```java
Deque<String> queue = new ArrayDeque<>();
String item;
queue.offer(item);
queue.poll();
queue.peek();
```

- **PriorityQueue**: Min-Heap , Max-Heap
```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());

minHeap.offer(item);
minHeap.poll();
minHeap.peak();
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
// Default sort by key
TreeMap<String, Integer> map = new TreeMap<>();
// otherwise comparator sort
//ex. reverse sort
TreeMap<String, Integer> map = new TreeMap<>(Comparator.reverse);


map.put('a', 1);
map.put('b', 2);
map.put('c', 3);

map.values();

map.firstKey(); // smallest key: 'a'
map.lastKey(); // largest key: 'c'
```


note: TreeMap otherwise sorted
```Java
Map<Person, Integer> = TreeMap<>(new Comparator<Person>){
	public int compare(Student p1, Student p2){
		return p1.score > p2.score ? -1 : 1;
	}
}

class Student{
	public String name;
	public int score;
}
```


note: map for loop

```Java


for(Map.entry<String, Integer> entry : map.entrySet()){
	System.out.println(entry.getKey());
}
// or 
map.foreach((key, value) -> {
	System.Out.println(key);
})
```
note : `Map.Entry(K,V)`
- `K getKey()`
- `V getValue()`
- `V setValue()`

