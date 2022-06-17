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
> Underpopulation count = 9

#### Overpopulation
Overpopulation happens when a living cell has more than 3 neighbors or `{4,5,6,7,8}`.
Four neighbors combination count = `8!/(8-4)! = 1680`
Five neighbors combination count = `8!/(8-5)! = 6720`
Six neighbors combination count = ` 8!/(8-6)! = 20160`
Seven neighbors combination count =  `8!/(8-7)! = 40320`
Eight neighbors combination count = ` 8!/(8-8)! = 40320`
> Overpopulation count = 109200

Of course we need to take into account that the chance of the cell actually being alive is `50%` thus
halving the amount of state changes
#
> Total state changes by death = `109209 /2 = 54604`
## Being Born
A new cell is born if the cell is dead and has 3 neighbors (of course with the chance of the cell being dead `50%`)
> Total state changes by being born = `8!/(8-3)! / 2 = 336`

> Total state changes count = `54604 + 336 = 54940`

The chance of a change is the total amount of change states divided by the total grid size.

Change chance = `54940/8! = 0.7930024104732899 = 79.3%`

By randomizing an entire grid of `n*n` the average data that needs to be stored by storing only changed positions equals `8 * n * n * 0.793.. = n * n * 6.344019283786319 = ~ 6.3 n^2`


The point of peak is when `6.3 * n^2 = n ... TODO: FIX THIS DOESN'T TAKE EDGES & CORNERS INTO ACCOUNT BAD CALCULATIONS

## Result

On an average case the amount of data required to store changes on 
