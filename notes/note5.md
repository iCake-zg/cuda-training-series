
## 原子操作，归约运算与线程束洗牌


## MOTIVATING EXAMPLE

Sum - reduction


## REDUCTION: NAIVE THREAD STRATEGY

![[Pasted image 20250805110720.png]]

## ATOMICS TO THE RESCUE

![[Pasted image 20250805110910.png]]


## OTHER ATOMICS

![[Pasted image 20250805142247.png]]

## ATOMIC TIPS AND TRICKS

	Could be used to determine next work item,queue slot, etc

	int my_position = atomicAdd(order,1)

	Most atomics return a value that is the "old" value that was in the location receiving the atomic update.


## CLASSICAL PARALLEL REDUCTION

![[Pasted image 20250805143418.png]]

## PROBLEM: GLOBAL SYNCHRONIZATION

	One possible solution:decompose into multiple kernels:

	- Kernel launch serves as a global synchronization point
	
	- Kernel launch has low SW overhead(but no zero)

	Other possible solutions:

	- Use atomics at the end of threadblock-level reduction

	- Use a threadblock-draining approch

	- Use cooperative groups - cooperative kernel launch


![[Pasted image 20250805145304.png]]



## GRID-STRIDE LOOPS （网络步长循环）

当数据量不一定是 *block*数量*block* 大小的时候，如何写一个通用，可拓展，不浪费线程的核？

![[Pasted image 20250805151810.png]]

## PUTTING IT  ALL TOGETHER

![[Pasted image 20250805173614.png]]


![[Pasted image 20250805174003.png]]


## WARP SHUFFLE

 ![[Pasted image 20250805174358.png]]

## INTRODUCING WARP SHUFFLE

![[Pasted image 20250805174511.png]]


## WARP SHUFFLE REDUCTION

![[Pasted image 20250806102237.png]]

## WARP SHUFFLE BENEFITS

	Reduce or elminate shared memory usage

	Single instruction vs 2 or more instructions

	Reduce level of explicit synchronization


## WARP SHUFFLE TIPS AND TRICKS

	Broadcast a value to all threads in the warp in a single instruction

	Perform a warp-level prefix sum

	Atomic aggregation

