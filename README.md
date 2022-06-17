# Conway's Game Of Life - Copy the whole array?
Conway's Game Of Life requires all calculations on each cell to happen simultaneously. In order to achieve such goals data needs to be copied.

 Should the entire grid be copied? Should only the changed positions be copied?
In this markdown I'll cover this question.

# Solution
Firstly, a maximum grid size should be agreed upon. I will use a grid of `uint32` (4 bytes).

The cost of copying an entire grid of size `n*n ` equals `n*n/sizeof(byte) => n*n/8`. This is because each cell contains a boolean which could be optimized into a single bit.

> Full Copy = `n*n/8`

The cost of copying / storing `n` changed cells equals  `n * sizeof(position_info)`.
Position information could be `(x,y)` but that requires two integers costing us 8 bytes.

 We can use one index by using our grid as a one dimensional array but that won't optimize the size used as the last index is `max_size(int) ** 2 = 0xfffffffe00000001`.

> Single Cell Change = `8 * n`

In order to decide which way is more efficient we need to know the chance of a change occurring.

# Change States

The states in which a change happens requires storage.
What causes change? Alive dying of under/over population / Dead Being born with 3 neighbors 
## Death
#### Underpopulation
Underpopulation happens when a living cell has zero / one living neighbors.
Zero neighbors combination count = `8!/(8-0)! = 1`
One neighbor combination count = `8!/(8-1)! = 8`

> Underpopulation center count = 9
> Underpopulation edge count = `5!/(5-0)! + 5!/(5-1)! = 1 + 5 = 6`
> Underpopulation corner count = `3!/(3-0)! + 3!/(3-1)! = 1 + 3 = 4`

#### Overpopulation
Overpopulation happens when a living cell has more than 3 neighbors or `{4,5,6,7,8}`.
> Overpopulation center count = `Σ(i=4, n=8) {n!/(n-i)!} = 109200`
> Overpopulation edge count = `Σ(i=4, n=6) {n!/(n-i)!} = 1800`
> Overpopulation edge count = `0`

Of course we need to take into account that the chance of the cell actually being alive is `50%` thus
halving the amount of state changes
#
> Total state change by death = `(109209 + 1800 + 0) /2 = 55504`
## Being Born
A new cell is born if the cell is dead and has 3 neighbors (of course with the chance of the cell being dead `50%`)
> Born center count = `8!/(8-3)! / 2 = 336`
> Born center count = `6!/(6-3)! / 2 = 60`
> Born corner count = `1`

> Total state changes by being born = `336 + 60 + 1 = 397`

#

> `total_change_count = 55504 + 397 = 55901`

The chance of a change is the total amount of change states divided by the total grid size.

> Total center count = `Σ(i=0, n=8) {n!/(n-i)!} = 109601`
> Total edge count = `Σ(i=0, n=6) {n!/(n-i)!} = 1957`
> Total corner count = `Σ(i=0, n=4) {n!/(n-i)!} = 65`
> `total_count = 109601 + 1957 + 65 = 111623`

`change_chance = total_change_count/total_count = 0.5008018060793923 = ~ 50%`

By randomizing an entire grid of `n*n` the average data that needs to be stored by storing only changed positions equals `position_info * cell_count * change_chance =`
 `= 8 * (n * n) * 0.500.. = n * n * 4.00.. = ~ 4 * n^2`


The point of peak is when `4 * change_count = n * n`
or `change_count = n*n/4`

## Result

If the number of changed cells is more than a quarter of the entire grid just copy the entire grid, otherwise store data on each changed cell
