

Sometimes we need a vector of custom data types, like a pair<int,int> and maybe a struct. What if we need to sort them

Eg, we have a list of cars position and another list of cars speed

We want to sort based on who will go there first,but also want to keep the position


```c++
//vector<pair<int, int>> vl

//pair <start_position, finish_time>;

//We want to sort based on finish time, v[0] -> first finisher, we also want to store where she/he finished from


//So, we need to override the vector comparator, so we sort based on pair.first value



int main() {
    // pair<start_position, finish_time>
    vector<pair<int, int>> races = {{3, 45}, {1, 30}, {5, 25}, {2, 35}};
    
    // Sort by finish_time (second element)
    sort(races.begin(), races.end(), [](const pair<int, int>& a, const pair<int, int>& b) {
        return a.second < b.second;  // Ascending, Non Decreasing

        return a.second > b.second; //Descending,  Non increasing
    });
    
    cout << "Sorted by finish time:\n";
    for (const auto& p : races) {
        cout << "Position: " << p.first << ", Time: " << p.second << "\n";
    }
    
    return 0;
}
```