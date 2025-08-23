

// Below is the general template for two pointers


```c++

for (int right = 0, left = 0; right < n; right++) {
    // Add element at right to window, increase the window
    //Change necessary state (ie. sum of sliding window update or frequency counter update)
    //We do the operation and then check if its invalid because of growing

    while (window_invalid) {
        // Our previous operation ofgrowing made us invalid window
        //Time to shrink until valid again
        //do the opposite of state change we did when adding an element
        //For example, decrease frequency, reduce rolling sum
        //Example sum -= nums[left]
        left++; //shrink the window
    }

    // Update result
    //Example Maxumim of Sliding window sum
    //Maximum Unique elems on the valid window
}


```

problems: https://leetcode.com/problems/longest-repeating-character-replacement/ 

problems: https://leetcode.com/problems/longest-substring-without-repeating-characters/

