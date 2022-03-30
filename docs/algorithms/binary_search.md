# Binary search

Binary Search is a searching algorithm for finding an element's position in a sorted array.

**Algorithm**

```cpp
int binarySearch(std::vector<int> array, int x) {
  int low = 0;
  int high = array.size() - 1;

  while (low <= high) {
    int mid = (low + high) / 2;

    if (array[mid] == x) {
      return mid;
    } else if (array[mid] < x) {
      low = mid + 1;
    } else {
      high = mid - 1;
    }
  }

  // Not found
  return -1;
}
```

