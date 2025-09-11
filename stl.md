

# Arrays and Linked List


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


# Sets and Lookup Tables

### Set / Unordered Set 
Store unique elements, find them, add them

Constant time lookUp/Add for unordered set, lg lookup for ordered

```c++
#include<set>
#include<unordered_set>

set<type>st; //RBT

unordered_set<type>st;

st.reserve() //Only possible for unordered, cause its a hashtable

st.insert(element); //O(1) for unordered, lgn for unordered
st.contains(element) -> bool //return true if found or false
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


