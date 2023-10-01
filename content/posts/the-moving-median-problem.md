---
title: "The streaming median problem"
description: Three solutions to the moving median problem in C# 
htmlMetadata: Benchmarking different data structures as string autocompleters.
image: 'stream-resized.png'
tags: data-structures; 
postNumber: 6
date: 'October 1st, 2023'
---

The other day, I stumbled upon a thought-provoking technical interview question: '*How would you calculate the median number of a stream of numbers?*' As a data-structures aficionado, this piqued my interest. Since I've been less active recently, I thought, why not dedicate a blog post to this topic? Let's dig in!

# The basic approach 

First and foremost, let's start with a quick refresher of high-school level mathematics. According to Wikipedia, "*In statistics and probability theory, the median is the value that separates the higher half from the lower half of a data sample, a population, or a probability distribution. For a data set, it may be thought of as "the middle" value.*" In simpler terms, it's the element that lies positionally in the middle when all the elements are arranged in order. Needless to say, to calculate the median, the elements must be sortable and comparable. 

Now, when it comes to programming, calculating the median might seem like a straightforward task. All you need to do is order the elements and access the middle element. If you have an even number of elements, you can average the two middle elements. In the world of .NET, this could be one of the simplest implementations:

```csharp
public class ListBasedMedian : IMedian
    {
        private List<int> _list;
        private double _median = double.NaN;

        public ListBasedMedian()
        {
            _list = new();
        }

        public void Add(int value)
        {
            _list.Add(value);
        }
      
        public double? Value
        {
            get
            {
                UpdateMedian();
                return _median;
            }
        }
        
        private void UpdateMedian()
        {
            if (_list == null || _list.Count == 0)
            {
                _median = double.NaN;
            }

            _list.Sort();

            int middleIndex = _list.Count / 2;

            if (_list.Count % 2 == 0)
            {
                int middleValue1 = _list[middleIndex - 1];
                int middleValue2 = _list[middleIndex];
                _median = (double)(middleValue1 + middleValue2) / 2;
            }
            else
            {
                _median = _list[middleIndex];
            }
        }
    }
```
For the sake of simplicity, above and all the following code examples will use integers.

This implementation runs on O(n log n) time:

- Adding a number to a ```List``` is constant time O(1);
- [Array.Sort()](https://learn.microsoft.com/en-us/dotnet/api/system.collections.arraylist.sort?view=net-7.0) method runs in O(n log n) time as the docs describe.
- Updating the median is also constant time O(1).

**So the weight of the median operation lays in sorting the array anytime the median is requested**. In terms of space, the implementation is **quite optimized**: Internally List<int> uses an array of fixed size. So we have basically the list of integers and the median value itself. It's worth noting that the array will be copied internally to a new instance to increase its size once it's full. This will be happening any time the number of elements duplicates starting from 4: eg 4, 8, 16 , 32, etc. In any case, the space complexity of this solution will be linear with respect of the size of the input - eg O(n). 

What is the problem of this? On demand median extraction is quite fast, but what if **we're processing a stream of numbers in realtime getting the median average after every element?**

# Looking for the fastest real-time median calculation possible: priority queues

Optimizing for constant time access to the latest median value, denoted as O(1), requires the use of more advanced data structures. In this scenario, we can harness the power of [priority queues](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.priorityqueue-2?view=net-6.0), which were introduced in .NET version 6. A priority queue is a specialized data structure designed to efficiently manage elements with associated priorities. The key feature of a priority queue is that it ensures that the element with the highest (or lowest, which is the case in dotnet) priority is always readily accessible.

These priority queues function as binary heaps that automatically maintain their elements in order after insertion. By employing two priority queues, one for elements greater than the median and another for elements smaller than the median, we can achieve insertion times ranging from O(1) to O(log n), while peeking at the top element remains a constant-time operation. **This is crucial because the median computation relies on peeking elements from the tops of these heaps**.

As the following code snippet demonstrates, we must handle the careful insertion of elements into the respective priority queues. However, this is merely an implementation detail:

```csharp
public class PriorityQueueBasedMedian: IMedian
    {
        private int? _median;
        private readonly PriorityQueue<int, int> _prioritySmaller;
        private readonly PriorityQueue<int, int> _priorityBigger;

        public PriorityQueueBasedMedian()
        {
            _priorityBigger = new PriorityQueue<int, int>();
            _prioritySmaller = new PriorityQueue<int, int>();
        }

        public void Add(int value)
        {
            //first element
            if (_median is null && _priorityBigger.Count == 0 && _prioritySmaller.Count == 0)
            {
                _median = value;
                return;
            }

            //second element
            if (_median is not null && _priorityBigger.Count == 0 && _prioritySmaller.Count == 0)
            {
                if (value < _median)
                {
                    _prioritySmaller.Enqueue(value, -value);
                    _priorityBigger.Enqueue(_median.Value, _median.Value);
                    _median = null;

                    return;
                }

                if (value > _median)
                {
                    _priorityBigger.Enqueue(value, value);
                    _prioritySmaller.Enqueue(_median.Value, -_median.Value);
                    _median = null;
                    return;
                }

                if (value == _median)
                {
                    _priorityBigger.Enqueue(value, value);
                    _prioritySmaller.Enqueue(value, -value);
                    _median = null;
                    return;
                }

                _median = value;
                return;
            }

            //succesive elements
            if (_priorityBigger.Count != 0 && _prioritySmaller.Count != 0)
            {
                if (_median is null)
                {
                    if (_priorityBigger.Peek() == value || _prioritySmaller.Peek() == value)
                    {
                        _median = value;
                        return;
                    }

                    if (_priorityBigger.Peek() > value && _prioritySmaller.Peek() < value)
                    {
                        _median = value;
                        return;
                    }

                    if (_prioritySmaller.Peek() > value)
                    {
                        _median = _prioritySmaller.EnqueueDequeue(value, -value);
                        return;
                    }

                    if (_priorityBigger.Peek() < value)
                    {
                        _median = _priorityBigger.EnqueueDequeue(value, value);
                        return;
                    }
                }

                if (_median is not null)
                {
                    if (_median == value)
                    {
                        _priorityBigger.Enqueue(value, value);
                        _prioritySmaller.Enqueue(value, -value);
                        _median = null;
                        return;
                    }

                    if (_prioritySmaller.Peek() > value)
                    {
                        _prioritySmaller.Enqueue(value, -value);
                        _priorityBigger.Enqueue(_median.Value, _median.Value);
                        _median = null;
                        return;
                    }

                    if (_priorityBigger.Peek() < value)
                    {
                        _priorityBigger.Enqueue(value, value);
                        _prioritySmaller.Enqueue(_median.Value, -_median.Value);
                        _median = null;
                        return;
                    }
                }
            }
        }

        public double? Value
        {
            get
            {
               if(_median is not null) { 
                    return _median.Value; 
                }

               if( _median is null && _prioritySmaller.Count != 0) {
                    return ((double)_prioritySmaller.Peek() + (double)_priorityBigger.Peek()) / 2;
                }

                return double.NaN;
            }
        }
    }

```
One caveat of the aforementioned solution lies in its space complexity. For each element injected into a priority queue, we're also inserting its priority, which essentially duplicates the used space. This duplication can be highly inefficient and poses a critical challenge for the nature of the solution.

# A space-optimized, yet time-inefficient alternative

What if we know the potential set of values beforehand? Instead of storing every individual sample, **we can compute a dictionary with the sample set as keys, where each key points to the number of occurrences for that value**.

To calculate the median using this approach, we must access the **keys in order** to reach the middle sample by aggregating their associated counts. This is precisely why we opt for a ```SortedDictionary``` instead of a regular ```Dictionary```. To achieve this, we rely on the total sample count stored as a state variable, and the middle position will be exactly half of the sample count. The process of seeking the key in the middle position is a linear operation with a time complexity of O(n). Additionally, the act of ordering the keys necessitates O(n log n) time complexity. Consequently, this solution may not be highly efficient in terms of time complexity, especially if we need real-time updates of the median: 

```csharp
public class DictionaryMedian : IMedian
    {
        private readonly SortedDictionary<int, int> _numberCounts;
        private uint _numberOfSamples = 0;
        private double _median = double.NaN;

        public DictionaryMedian()
        {
            _numberCounts = new();
        }

        public double? Value
        {
            get
            {
                var keys = _numberCounts.Keys;

                int temporarySum = 0;
                bool unevenSamples = _numberOfSamples % 2 != 0;
                var medianPosition = Convert.ToUInt32(Math.Abs(_numberOfSamples / 2)) + 1;
                int? previousKey = 0;

                foreach (var key in keys)
                {
                    temporarySum += _numberCounts[key];

                    if (temporarySum < medianPosition)
                    {
                        previousKey = key;
                        continue;
                    }
                    else if (temporarySum >= medianPosition)
                    {
                        if (unevenSamples)
                        {
                            _median = key;
                        }
                        else
                        {
                            _median = previousKey == 0 ? key : (double)(((double)key + (double)previousKey) / 2);
                        }

                        break;
                    }
                    else if (temporarySum > medianPosition)
                    {
                        _median = key;
                        break;
                    }
                }

                return _median;
            }
        }

        public void Add(int number)
        {
            _numberOfSamples++;

            if (_numberCounts.ContainsKey(number))
            {
                _numberCounts[number]++;
            }
            else
            {
                _numberCounts[number] = 1;
            }       
        }
    }
```
So calculating the median executes in O(n log(n)) complexity; As per big "O" properties, given a sum of functions, we take the order of that one that grows faster, in this case, O(n log(n)).

Before going into the benchmarks, let's ensure the correctness of the three solutions,by passing them through the following unit test:

```csharp
public class DictionaryMedianTests
{
    [Theory]
    [InlineData(new[] { 5 }, 5)] // single element
    [InlineData(new[] { 2, 4, 6 }, 4)]
    [InlineData(new[] { 10, 20, 30, 40 }, 25)]
    [InlineData(new[] { 7, 3, 1, 5, 9 }, 5)]
    [InlineData(new[] { 7, 3, 1, 5, 9, 9, 9 }, 7)]
    [InlineData(new[] { 7, 7, 7, 7 }, 7)]  // All elements are the same
    [InlineData(new[] { 7, 7, 7, 7, 7 }, 7)]  // All elements are the same, uneven
    [InlineData(new[] { 1, 3, 5, 7, 9 }, 5)]  // Odd number of elements
    [InlineData(new[] { 1, 2, 3, 4, 5, 6 }, 3.5)]  // Even number of elements
    [InlineData(new[] { 1, 2, 3 }, 2)]  // Sorted in ascending order
    [InlineData(new[] { 3, 2, 1 }, 2)]  // Sorted in descending order
    [InlineData(new[] { 2, 1, 3 }, 2)]  // Unsorted with median in the middle
    [InlineData(new[] { 3, 2, 2, 1, 3 }, 2)]// Unsorted with multiple medians
    public void AddedNumbers_ReturnsCorrectValue(int[] numbers, double expectedMedian)
    {
        IMedian median = new DictionaryMedian(); // We can copy the test for any of the implementations.

        foreach (var number in numbers)
        {
            median.Add(number);
        }

        Assert.Equal(expectedMedian, median.Value);
    }
}
```

# Benchmarking

So let's compare the performance of these Median implementation in two different scenarios: after each element median calculation and only after adding a number of elements. The number of potential distinct elements in the sample is bounded and relatively small (100). 

- Add 100K elements, only then calculate median: 

```
|              Method | DataSize |      Mean |     Error |    StdDev |     Gen0 |     Gen1 |     Gen2 |  Allocated |
|-------------------- |--------- |----------:|----------:|----------:|---------:|---------:|---------:|-----------:|
|          ListMedian |   100000 |  2.798 ms | 0.0398 ms | 0.0372 ms | 285.1563 | 285.1563 | 285.1563 | 1024.55 KB |
| PriorityQueueMedian |   100000 |  4.660 ms | 0.0164 ms | 0.0153 ms | 531.2500 | 515.6250 | 500.0000 | 2048.96 KB |
|    DictionaryMedian |   100000 | 14.161 ms | 0.0318 ms | 0.0249 ms |        - |        - |        - |    5.05 KB |
```

- Add 1000 elements, median is updated after each *Add* operation. 

```
|              Method | DataSize |        Mean |     Error |    StdDev |    Gen0 | Allocated |
|-------------------- |--------- |------------:|----------:|----------:|--------:|----------:|
|          ListMedian |     1000 | 3,255.41 us | 38.082 us | 39.108 us |       - |    8.3 KB |
| PriorityQueueMedian |     1000 |    77.59 us |  1.356 us |  1.900 us |  1.9531 |  16.48 KB |
|    DictionaryMedian |     1000 | 2,607.54 us |  8.231 us |  7.699 us | 15.6250 | 150.93 KB |
```

It becomes evident that ```DictionaryMedian``` performs optimally when the component is not under heavy stress, resulting in minimal memory allocation. However, during each median calculation, a significant amount of memory is allocated, possibly due to a specific line within the BCL ```SortedDictionary``` type: ```public KeyCollection Keys => _keys ??= new KeyCollection(this);```. This allocation is necessary to access the ordered key collection.

On the contrary, **in scenarios involving pure streaming medians, it becomes imperative to opt for a priority queue-based implementation**, even though it incurs double the memory consumption by using the BCL types. It's noteworthy that potential optimization lies in creating a custom ```PriorityQueue``` that does not require a pair value-priority relationship but treats them as identical elements. This approach would allow us to achieve the same memory usage as ```ListMedian``` while significantly improving performance by two orders of magnitude.

And that's it :) . The full code can be downloaded here: https://github.com/jorgeolive/onlinemedian



*Disclaimer: Text enhanced with the help of ChatGPT*