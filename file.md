

## Opening A File

Every I/O in C++ is a stream, which is a interface to read/write from external systems such as files, network, console.

To read from a file, the very first step is defining our file stream, an interface to read/write from file


We create fstrea. aka filestream, but its better to create read stream and write stream separately.

So to read, we create ifstresm (input file stream)
To write, we create ofstream (output file stream)


```c++

#include<fstream>
#include<iostream>
using namespace std;

int main(){
    ifstream in_file;
    in_file.open(fname);
    //Example fname: text.txt, logs.txt. dino.csv
    if (!in_file.is_open()) {
        cout << "File Opening Failed";
        return -1;
    }


    //Always remember to closwe files
    in_file.close()
    return 0;
}
```

## Reading A File

So after opening a file, what to we want to do ? Read from the stream. Theres maily 2 ways to read a file, 
- Word by Word (terminates at space/EOF/newline)
- Read line by line (terminates at newline/EOF)

### Example

```c++
string word;
//Reading word by word until space
while (in_file >> word) {
    cout << word << endl;
}

//Reading one line at a time
while (getline(in_file, word)) {
    cout << word << endl;
}
```