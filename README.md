# From-Majority-to-Frequent-Elements-Boyer-Moore-and-Misra-Gries
## Boyer-Moore Majority Vote Algorithm

The Boyer-Moore majority vote algorithm[^wiki-article] is an efficient algorithm used for finding the majority element, if any, in a sequence of elements using linear time complexity *$O(n)$* and constant space complexity *$O(1)$*.  

A majority element is an element that appears more than $\lfloor n / 2 \rfloor$[^floor] times in a sequence, where n is the size of the sequence.  
Based on this definition, there can be only one majority element (or none) in a given sequence.


## Brute-Force Approach
One way to solve this problem is by simply iterating through the array, supposing that the current element is indeed the majority element, counting its number of occurrences, and if it is repeated more than $\lfloor n / 2 \rfloor$ times, then it must be the majority element (since there can be at most one majority element); otherwise we repeat this same process for the remaining array elements.  
This approach uses nested loops, so its time complexity is *$O(n^2)$*, and it keeps track of only one candidate at a time, so its space complexity is *$O(1)$*.

```python
def majority_element(array):
    if len(array) == 0:
        raise ValueError("No majority element: Empty sequence")

    threshold = len(array) // 2
    for candidate in array:
        count = 0
        for element in array:
            if candidate == element:
                count += 1
        if count > threshold:
            return candidate
    
    raise ValueError("Majority element does not exist.")
```

## Dictionary (Mapping) Approach
Another, more efficient, way of solving this problem is by using a dictionary or a hashmap to store the counts of unique elements in the input array and then returning the majority element if it exists.  
This approach has a time complexity of *$O(n)$* since we need to pass through the array once in order to count the occurrences of each element and construct a counts dictionary.  
Although to be more accurate, the time complexity is an ***amortized** $O(n)$* because the dictionary creation can go up to *$O(n^2)$* in case there were too many hash collisions[^hash-collision].  
As for the space complexity, it would be an *$O(n)$* since we need a dictionary to store the counts of each element in the input array.

```python
def majority_element(array):
    if len(array) == 0:
        raise ValueError("No majority element: Empty sequence")

    threshold = len(array) // 2
    counts = {}
    for candidate in array:
        if candidate not in counts:
            counts[candidate] = 0
        counts[candidate] += 1
        if counts[candidate] > threshold:
            return candidate
    
    raise ValueError("Majority element does not exist.")
```

Or we can use `collections.Counter` which is more efficient for such a task

```python
from collections import Counter

def majority_element(array):
    if len(array) == 0:
        raise ValueError("No majority element: Empty sequence")

    counts = Counter(array)
    threshold = len(array) // 2
    [(candidate, count)] = counts.most_common(1)
    if count > threshold:
        return candidate            
    raise ValueError("Majority element does not exist.")
```


## Boyer-Moore Algorithm[^original-paper]
Suppose that an election has taken place and that there was a clear winner with a number of votes greater than at least half the number of total votes (or voters).  
The winner represents the majority element in our case, and we are interested in fiding it of course.  
Well, let's suppose that humans weren't very civilized at this period of history, and that the way to settle an election was a bit (ok, maybe too much)
bloody.  
The voters had to knock each other out until one clear winner survived.  
The rules are that a voter for a certain candidate can only be knocked out by a voter for another, different, candidate.  
And he must be knocked out in the next round by another voter, for a different candidate of course, if such a voter exists. In other words, no voter can win more than one round, he has to be eliminated after he wins.   
If a winner exists, that is a majority element, then he would have more than half of the voters.
If he wins by a margin, that is by a single vote, then he would have `m + 1` voters on his side, and the other candidates would have all the remaining voters, that is `m` voters, where `m = (total number of voters) / 2`.

This means that if the winner's voters were against their *enemies* in pairs of two, they would cancel each other out, one by one, until one person was left.  
Let us try to visualize this with an example: say we have 3 candidates: `A`, `B` and `C`.  
Candidate `A` has 4 voters: `A1`, `A2`, `A3` and `A4`.  
Candidate `B` has 2 voters: `B1` and  `B2`.  
Candidate `C` has 1 voter &nbsp;: `C1`.  

* In the first round `A1` goes against `B1`, and `B1` wins (it doesn't really matter who wins first; but we're just trying to simulate a worst case and this aligns perfectly with it I guess).  
* In the second round we have `B1`, the winner of the previous round, against `A2` for example. According to the rules, `B1` can't win a second time, so he gets knocked out by `A2` who proceeds to the next round.  
* In the 3rd round `B2` knocks out `A2`.
* In the 4th round `A3` knocks out `B2`.
* In the 5th round `C1` knocks out `A3`.
* In the 6th and final round `A4` knocks out `C1`.
* No more voters are remaining and the only one standing is `A4`, so `A` is the winner.  

And this process would work if the (winner) majority element had way more votes (occurrences) than the remaining candidates since in the worst case we had at least one voter who's still standing.  


The previous process, the fighting phase, is formerly known as the *pairing phase*, because the votes (voters) are taken down one by one in a pairs.  

Now, what happens if a majority element doesn't exist? There would still be someone who's standing at the end even though its corresponding candidate doesn't represent tha majority element.  
This is why a second phase is necessary, called the *counting phase* where we would verify if the selected candidate is indeed a majority element, by going through all of the votes one more time, counting the number of votes in favor of this candidate only and comparing it with the threshold ($\lfloor n / 2 \rfloor$).  

Now how do we translate this algorithm into actual code?  
It's quite simple actually. We only need 2 variables, one to keep track of the current candidate and a counter to keep track of its current number of supporters or voters.  
**Note**: The elements of our array would represent the voters, and their values would be that of the candidates, so if a voter supports candidate `X` then its value would be `X`.  
For example:  
* Voters(Array): [1, 1, 2, 2, 1, 2]
* Candidates: 1 and 2

We can choose any element from the array to be a candidate (simply choose the first element) and we initialize the counter to 0 (since no supporters have been found before iterating through the array).  
Then, as we go through the array, if we encounter a supporter, that is the same value as the candidate we would increment the counter, otherwise we should decrement it, since that supporter would be knocked out (but the remaining supporters, if any; depending on the value of counter, are still there to knock or be knocked out).  
What if the counter reaches the value 0? That would mean that all supporters of the current candidate have been knocked out, and the one standing right now is the current voter, so we would have to update our candidate variable.  
And this process continues until we exhaust the entire array. This was the pairing phase, and I believe that the counting phase is self-explanatory and quite easy to implement.  

This makes the time complexity of this algorithm *$O(n)$* since we need to loop through the input array twice, once for each phase, and the space complexity is constant *$O(1)$* since we only need 2 variables, one for the candidate and another for the counter.

```python
def majority_element(array):
    if len(array) == 0:
        raise ValueError("No majority element: Empty sequence")
    
    # Set the initial candidate and counter
    candidate = array[0]
    count = 0

    # Loop through the array
    for element in array:
        # If we find the current candidate, increment the counter
        if element == candidate:
            count +=1
        # If the counter reaches zero, switch to a new candidate and
        # reset the counter
        elif count == 0:
            candidate = element
            count = 1
        # Otherwise, decrement the counter for any non-matching element
        else:
            count -= 1

    # Check if the candidate has enough votes to be the majority
    threshold = len(array) // 2
    candidate_count = 0
    for element in array:
        if element == candidate:
            candidate_count += 1
    
    # Alternatively you could use: array.count(candidate) > threshold
    if candidate_count > threshold:
        return candidate

    raise ValueError("Majority element does not exist.")
```

## The Frequent Items Problem and Misra-Gries Summary[^misra-gries]  
A generalization of the previous problem is when we want to find not only the majority element, but all frequent elements which are elements occurring more than a given fraction of the time, that is more than a certain proportion of the input size.
For example, we might want to find all elements that appear more than $\lfloor n / 3 \rfloor$ times. In other words, find the 2 most frequent elements, if any.  
Why the **2** most frequent elements you say? It's because that is the maximum number possible given the constraint.  
And in the general case, there would be a maximum of **$(k - 1)$** most frequent elements for a threshold of $\lfloor n / k \rfloor$.


So how can we generalize the Boyer-Moore Majority Vote Algorithm (which can be thought of as the **1** most frequent element problem) to solve this problem? We simply need to imagine another fight :)  

Since there can be at most $(k - 1)$ most frequent elements, we can keep $(k - 1)$ candidate variables and counters respectively.  
If we encounter one of our candidates, we simply increment its count as usual.  
If we encounter a completely new candidate, then we should decide whether to eliminate one of our previous candidates and add this one or not.  
This is where the tricky part comes into play.  
The first thing to note is that there's no need to have a fight between our $(k - 1)$ candidates, since they should all win (if they are the true most frequent elements). If a fight were to happen between them, then remember, at the end there would be only one person standing, and that person would represent the majority element, but that's not what we want right? We want to have all the most frequent elements.  
So then, when should a fight happen? It should happen when we already have a list of $(k - 1)$ candidates and a new one appears. We should determine whether to eliminate one of the previous candidates and add him to the list or not. This time though, a candidate can knock out all of his enemies, not just one. A bit confusing, right? However, if you think about it, you'll find that this is done because multiple candidates can be eliminated at once (there's no reason to choose one over the other), not just one, and all candidates should lose one vote because they suffered a loss.  
Alternatively, you can choose to eliminate only one candidate randomly (for example, the first one with an insufficient number of votes), thus always maintaining $(k - 1)$ candidates (if you've reached that number).  
You might have doubts at this point regarding the correctness of this algorithm, but I can assure you that it works fine in both cases, since if a most frequent element exists, then he should appear again in the list of candidates in an upcoming round.  
And if we delete only one candidate from the list, then we should be able to verify its eligibility when we perform an extra check at the end, just like in the original algorithm.  


This makes the time complexity of the algorithm linear *$O(n * k)$* since we would have to loop through the input sequence and potentially go through all $(k - 1)$ candidates when a fight should happen, and the space complexity is constant *$O(k)$* since we would need at most $(k - 1)$ variables to keep track of the candidates. 

Let's make this into Python code:  
```python
def most_frequent(array, k):
    if len(array) == 0:
        raise ValueError("No majority element: Empty sequence")
    if k <= 0:
        raise ValueError("Invalid k value: must be positive integer")

    # No need to do all the extra work if it isn't necessary
    if k >= len(array):
        return array.copy()

    # Use a dictionary to store element counts.
    counts = {}

    # First loop: count occurrences of each element.
    for element in array:
        if element in counts:
            counts[element] += 1
        else:
            counts[element] = 1
            # If exceeding maximum candidates, keep only elements with
            # positive count.
            if len(counts) > k:
                # Dictionaries can't be mutated as you loop through them
                # that's why we're creating a new dictionary to select
                # the corresponding candidates
                # Alternatively, we could've saved the current candidates
                # in a list, looped through the dictionary and deleted 
                # the corresponding candidates without any problems
                # but this approach is simpler
                new_counts = {}
                for candidate, count in counts.items():
                    # because count - 1 == 0 and that candidate
                    # should be eliminated
                    if count > 1:
                        new_counts[candidate] = count - 1
                counts = new_counts
    
    # Create a separate dictionary to count again for final verification.
    counts_check = dict.fromkeys(counts, 0)

    # Second loop: verify counts against threshold.
    for element in array:
        if element in counts_check:
            counts_check[element] += 1

    # Calculate the minimum required count for an element to be considered
    # frequent. Do you see why it's (k + 1)?
    threshold = len(array) // (k + 1)

    # Return the list of elements exceeding the threshold
    return [
        candidate
        for candidate, count in counts_check.items()
        if count > threshold
    ]
```


## Taking advantage of the structure of the problem
It is important to treat each problem separately and take advantage of its constraints whenever possible.  
If the input array was sorted, the majority element problem would be greatly simplified (it is not worth it to sort the array yourself, since the time complexity needed would be *$O(nlog(n))$*).  
We would simply loop through the array one item at a time and increment a counter variable as we're still consuming the same element (since the array is sorted, equal elements would be held next to each other in contiguous slots).  

If this element has been repeated as many times as needed, then we've found our majority element; otherwise, if we counter a different element, we would have to repeat the same process again.

```python
def majority_element(array):
    if len(array) == 0:
        raise ValueError("No majority element: Empty sequence")
    
    array = sorted(array)
    count = 0
    candidate = array[0]
    threshold = len(array) // 2
    for element in array:
        if element == candidate:
            count += 1
        else:
            candidate = element
            count = 1
        if count > threshold:
            return candidate

    raise ValueError("Majority element does not exist.")
```  
One minor remark is that there's no need to check if the candidate's count is greater than the threshold until you've looped through at least half of the array. I leave it as an exercise to the reader to try and adapt this remark to the code if he wishes.  

One other way to take advantage of a sorted array is the fact that if a majority element exists, then it must be the median of this array[^median].  
Since the majority element would have at least `n / 2 + 1` occurrences, where n is the size of the array, we would get the following:
* If the first occurrence of this element is at the first position of the array, then it would at least reach the `n / 2`th position which represents the median of the array.  
* If the last occurrence of this element is at the last position of the array, then it would at least start from the `n / 2`th position, which represents the median of the array again. (you can imagine the array in reverse order if you're having a hard time believing that the median would indeed be included).  
* Any other possibility would be bound by the two previous cases, making it inevitable to have the median as the majority element.  

Of course, the majority check is not necessary for this algorithm if it is guaranteed to have a majority element in the input. (Neither is it necessary for the Boyer-Moore algorithm; that is the counting phase can be eliminated).

```python
def majority_element(array):
    if len(array) == 0:
        raise ValueError("No majority element: Empty sequence")
    
    array = sorted(array)
    median = len(array) // 2 # the median index is the same as the threshold
    candidate = array[median]
    if array.count(candidate) > median:
        return candidate

    raise ValueError("Majority element does not exist.")
```


## References
[^wiki-article]: [Boyer–Moore majority vote algorithm. wikipedia](https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_majority_vote_algorithm)
[^floor]: [Floor and ceiling functions. wikipedia](https://en.wikipedia.org/wiki/Floor_and_ceiling_functions)
[^hash-collision]: [Hash collision. wikipedia](https://en.wikipedia.org/wiki/Hash_collision)
[^original-paper]: [A Fast Majority Vote Algorithm. J Strother Moore](https://www.cs.utexas.edu/users/boyer/ftp/ics-reports/cmp32.pdf) 
[^median]: [Median. wikipedia](https://en.wikipedia.org/wiki/Median)  
[^misra-gries]: [Misra-Gries Summaries. Graham Cormode](https://people.csail.mit.edu/rrw/6.045-2019/encalgs-mg.pdf)
