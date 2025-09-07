

Sometimes we might need to override our priority queue, to use in in a custom data type


For example, Top K most frequent elements, we might want to, create a Pair with 
{elem,count} and insert in pq, where we compare it based on value

pq.top() -> element with most frequency


How to overload, best option


Define a custom class, and a overload function
also make that function friend, so others can use it


```c++
class elements {
public:
    int key;
    int value;
    elements(const int &key, const int &value) {
        this->key = key;
        this->value = value;
    }
    friend bool operator <(const elements& a, const elements& b) {

        // Overloading the operator, now we gat min heap
        return a.value > b.value; // opposite of actual operator: min heap


        return a.value < b.value; //Max Heap, same direction as function defination
        
    }
};

```


One interesting thing, if we want Top K elements in the stream, we can use a min heap, so when a new element comes, we add it first, left heapify() and pop the top (minimum of k+1), so by this we ensure Kth elements remained, and all the minimums are pooped out, hence giving us Top K elements

Complexity:  NlgN for maxHeap with N size, NlgK with minHeap 
Problem: https://leetcode.com/problems/top-k-frequent-elements/description/



If we are using first class data types, like int, double, and we want a minHeap, we can also define 
the comparator while creating the heap


```c++
 std::priority_queue<int, std::vector<int>, std::greater<int>> min_heap;
```