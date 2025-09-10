

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

### Reading CSV format, comma separated line

We can override the default getline delimeter. The default delim is "\n" AKA newline if we dont define our delim in getline


Lets assume we have a text file/CSV like below
```text
NAME,LEG_LENGTH,DIET
Hadrosaurus,1.4,herbivore
Struthiomimus,0.72,omnivore
Velociraptor,1.8,carnivore
Triceratops,0.47,herbivore
Euoplocephalus,2.6,herbivore
Stegosaurus,1.50,herbivore
Tyrannosaurus Rex,6.5,carnivore
Deinonychus,1.1,carnivore
Allosaurus,5.2,carnivore
Compsognathus,0.25,carnivore
Parasaurolophus,1.8,herbivore
Ornithomimus,0.95,omnivore
Carnotaurus,4.1,carnivore
Gallimimus,1.3,omnivore
```

We need to parse each line, and convert them to native types of c++
Means string, double, boolean...
```c++
std::ifstream in_file;

//Lets define the input stream
in_file.open("d2.csv");

//Check if we were able to open the file;
//If the file doesn't exis or we dont have permission or wrong path
//The opening will fail and 
if (in_file.is_open() == false) {
    cout << "File opening error";
    return -1;
}

getline(in_file, word); //Skipping the first line;

while (in_file.eof() == false) {
    getline(in_file, name, ',');
    getline(in_file, leg_len, ',');
    getline(in_file, diet);

    //Do things with the data
    //Maybe store them in another file
    //Maybe create a class and store them.
    //WHatever your heart desire




}
in_file.close();
```


## Writing on a file

Suppose we want to write a file, and  store some array or array of arrays.
How we do it ?

```c++
int main() {


    //create the stream
    ofstream out_file;
    //Open the file using the stream
    out_file.open("nums.txt");
    if (out_file.is_open() == false) {
        cout << "file opening failed";
        return -1;
    }

    vector<vector<int>>nums{{1,2,3},{4,5,6},
     {7,8,9,10}, {100,200,300}, {6969}, };

    for (int i = 0; i < nums.size(); i++) {
        for (int j = 0; j < nums[i].size()-1; j++) {
            out_file << nums[i][j] << ",";
        }
        //inserting the last element without comma
        out_file << nums[i][nums[i].size()-1] << endl;

    }

    out_file.close();

    return 0;

}

```

Lets look at the output of our file

```txt
1,2,3
4,5,6
7,8,9,10
100,200,300
6969

```
