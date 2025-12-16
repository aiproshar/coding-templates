
# String As Streams

This allows us to read from string, like we are reading from memory, instead of disk. Tou can treat this as Reading from memory lived files! YaY

Very Powerful and useful for data validation


### Example of input string stream
```c++
string info{"Arafat 1507017 CSE 2.77"};

    //stringstream is the generic stream class
    //istringstream ->input string

    string info{"Arafat 1507017 CSE 2.1"};

    istringstream input_string(info);
    //now input_string is a stream, we can read if drome file, 
    //use getline, read word by word anything


    string name;
    unsigned int roll;
    string dept;
    string CGPA;
    input_string >> name >> roll >> dept >> CGPA;

    cout << name  << ":"  << " roll: " << roll << " GPA: dw" << CGPA << endl;
```