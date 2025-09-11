

# Sequence Containers


## Dynamic Array Vector

```c++
#include<vecrot>
//empty vec
vector<typename>vec;

//Vec with n items of x value

vector<int>vec(10, 5)  ->{5,5,5,5,5,5,5,5,5,5}

vec.reserve(1000) //Important, avoids doubling
                  //always set this to possible max size

vec.clear(); //empties all element, size = 0

vec.emplace_back(value); //Better than push back
vec.pop_back(); //remove last value

vec.front()  //access front ref
vec.back()   //access back ref
vec.at(index);
vec[index];

//delete 2nd element
//based index means pos:1

vec.erase(vec.begin()+1);

//Delete 2nd to 5th elemet

vec.erase(vec.begin()+1, vec.begin()+4);

//use range loops to reverse print
//
#include<range>
using namespace range;
for (const auto& num : reverse_view(vec)) {
        //Do stuffs
    }


rbegin(), rend() -> reverse iterator

reverse(vec.begin(), vec.end());


//----------Sort----------------

class Car {
public:
    int speed;
    int year;
    string name;
    Car(int speed, int year, string name) {
        this->speed = speed;
        this->year = year;
        this->name = move(name);
    }

};

sort(cars.begin(), cars.end(), [](const Car& a , const Car& b){return a.speed > b.speed;});

// a > b -> descending
// a < b -> ascending
```

## List 

### Constant element removal at iterator (need to know)

```c++
#include <list>

std::list<int> lst;
lst.emplace_front(10)        // O(1)
lst.emplace_back(20)        // O(1)

lst.front()  //-> front elem ref
lst.back()  //-> back element ref


lst.pop_front() //O(1)
lst.pop_back() //O(1)

list.erase(iterator); //example: LRU cache

```


## Double Ended Queue

Indexed Sequence container, Constant Access. Push/Pop from 
both front and back with O(1)

Not contiguous in memory but Okay!

```c++
#include<deque>

deque<type>dq;
dq.emplace_front(10);  // O(1)
dq.emplace_back(20);   // O(1)
dq.pop_front();     // O(1)
dq.pop_back();      // O(1)

dq.front(); ->fromt element ref
dq.back(); -> back element ref


dq.clear() //empty
dq.erase(dq.begin() + pos). //Zero index based position

//Delete 2nd to 5th elemet

dq.erase(vec.begin()+1, vec.begin()+4);


//delete last 4 elements
dq.erase()

```


# Associative Containers

## Set / Unordered Set 
Store unique elements, find them, add them

Constant time lookUp/Add for unordered set, lg lookup for ordered

```c++
#include<set>
#include<unordered_set>

set<type>st; //RBT

unordered_set<type>st; //HasTable

st.reserve() //Only possible for unordered, cause its a hashtable

st.insert(element); //O(1) for unordered, lgn for unordered
st.contains(element) -> bool //return true if found or false
st.find(element) -> iterator, st.end() if not found
st.erase(element/iterator) //->erases the element, return 0 or 1
                             //when we do value/element overload, 0 means not found, 1 means found & erased

st.clear() //empty the set

//Do not use custom comparator for ordered set, because Sets uniqueness is guranteed by the comparator. FFS

//Example
// PROBLEMATIC: Comparator only uses age for ordering
struct CompareByAge {
    bool operator()(const Person& a, const Person& b) const {
        return a.age < b.age;
    }
};

int main() {
    std::set<Person, CompareByAge> people;
    
    people.insert({"Alice", 25, 1});   // Inserted
    people.insert({"Bob", 30, 2});     // Inserted  
    people.insert({"Charlie", 25, 3}); // NOT inserted! Same age as Alice
    people.insert({"David", 25, 4});   // NOT inserted! Same age as Alice
    
    std::cout << "Set size: " << people.size() << "\n"; // Only 2!
    
    // We lost Charlie and David because they have the same age as Alice
}
```


## Map/Unordered Map

```c++
#include<map>
#include<unordered_map>
map<key_type, value_type>mp;

unordered_map<key_type, value_type>ump;
ump.reserve().  //HashTable initial allocation, only for unordered map

//insert
mp[key]=balue;
mp.insert({key,value});

//Checking Element
mp.contains(key) //-> boolean //check if element exist
mp.find(key) //-> iterator / mp.end() if doesn't exist

//Accessing element
mp.at(key) - //>throws exception of not found
mp[key]  //-> reference to the value

//remove single element
mp.erase(key)
mp.erase(iterator) //-> throws exception of not found
//Clear all amp elements
mp.clear()
//Major Map Bug, 

    cout << mp.contains(69) << endl;; //false
    cout << mp.at(69) << endl; //bug, inserts item using default constructor
                            //If no default empty constructor for custom data types, throws error
    cout << mp.contains(69) <<endl; //true now

//Iteration over a map

for (const auto& [key,value] : ump) {
        cout << key << ":" << value << endl;
    }

//sorted Map for custom object
//Lambda with decl type

#include <map>
#include <iostream>

struct Person {
    std::string name;
    int age;
};

int main() {
    // Define lambda
    auto comp = [](const Person& a, const Person& b) {
        return a.age < b.age;
    };
    
    // Use decltype to get the lambda's type
    std::map<Person, std::string, decltype(comp)> peopleMap(comp); //Pass the lambda on constructor
 
    peopleMap.insert({{{"Alice", 30}, "Engineer"}});
    peopleMap.insert({{{"Bob", 25}, "Designer"}});
    peopleMap.insert({{{"Charlie", 35}, "Manager"}});
    
    for (const auto& [person, job] : peopleMap) {
        std::cout << person.name << " (" << person.age << ") - " << job << "\n";
    }
}
```


# Containers Adapters

## Queue -> FIFO

```c++
#include <queue>

std::queue<int> q // Default: deque
q.emplace(val)// O(1)

q.front() //Returns REF to front element
q.back()  //Returns REF to last element

q.pop()  //Removes the first element

q.empty() //-> check if its empty, return bool
q.size() //-> current no of elements
```

## Stack -> LIFO

```c++
#include <stack>

std::stack<int> stk // Default: deque
std::stack<int, std::vector<int>> vec_stack;

stk.push(10)
stk.top()
stk.pop()

stk.empty() //-> check if its empty, return bool
stk.size()  //-> current no of elements
```


## Priority Queue

Constant time lookup of largest/smallest element, defined by comparator.

By default its a Max Heap, but we can add greater<T> and also define lambda for custom object


```c++

#include <queue>

std::priority_queue<int> pq;                    // Max heap
std::priority_queue<int, std::vector<int>, 
                   std::greater<int>> min_pq;   // Min heap

pq.emplace(10) // O(log n)
int top = pq.top() // O(1) -> Max or Min based on type
//pq.top() always return a copy, not a reference
//If you update the top value, you need to pop and push again
//Restoring the heap property
pq.pop() // O(log n)

pq.empty() //-> check if its empty, return bool
pq.size()  //-> current no of elements

```

Now lets say we have a class,  where we have people's
property. 


```c++
#include <queue>
#include <vector>
#include <iostream>
#include <string>
using namespace std;

class Person {
public:
    string name;
    int age;
    double salary;

    // Constructor
    Person(string name, int age, double salary)
        : name(move(name)), age(age), salary(salary) {}

        
    void print() const {
        cout << left << setw(10) << "Name:" << setw(20) << name
             << setw(6) << "Age:" << setw(5) << age
             << setw(10) << "Salary: $" << salary << endl;
    }
};

int main() {
    // Create sample people
    vector<Person> people = {
        Person("Alice Johnson", 28, 75000),
        Person("Bob Smith", 35, 95000),
        Person("Charlie Brown", 28, 82000),
        Person("Diana Prince", 42, 120000),
        Person("Eve Davis", 31, 78000),
        Person("Frank Miller", 35, 88000)
    };

    // 1. Priority Queue by Age (youngest first - min heap)
    auto ageCompYoungest = [](const Person& a, const Person& b) {
        return a.age > b.age;  // > for min heap behavior
        return a.age < b.age;  // < for max heap behavior
    };
    priority_queue<Person, vector<Person>, decltype(ageCompYoungest)>
        pqByAgeYoungest(ageCompYoungest);

    for (const auto& ppl : people) {
        pqByAgeYoungest.emplace(ppl);
    }

    while (!pqByAgeYoungest.empty()) {

        pqByAgeYoungest.top().print();
        pqByAgeYoungest.pop();

    }

    return 0;
}
```

The output is like such

```bash
/Users/arafat/CLionProjects/untitled1/cmake-build-debug/untitled1
Name:     Alice Johnson       Age:  28   Salary: $ 75000
Name:     Charlie Brown       Age:  28   Salary: $ 82000
Name:     Eve Davis           Age:  31   Salary: $ 78000
Name:     Bob Smith           Age:  35   Salary: $ 95000
Name:     Frank Miller        Age:  35   Salary: $ 88000
Name:     Diana Prince        Age:  42   Salary: $ 120000

Process finished with exit code 0


```