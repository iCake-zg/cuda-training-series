

## MEMORY HIERARCHY REVIEW

	local storage
		- each thread has own local storage
		
		- Typically register (managed by the compiler)

	Shared memory / L1 
		- Program configurable: typically up to 48KB,64KB,96KB

		- Shared memory is accessible by shreads in the same threadblock

		- Very low latency

		- Very high throughput: 1TB/s aggregate

	L2:
		- All accesses to global memory go through L2,including copies to/from CPU host.

	Global memory

		- Accessible by all threads as well as host(CPU)

		- High latency(hundreds of cycles)

		- THroughput: up tp 900GB/s

## MEMORY ARCHITECTURE

![[Pasted image 20250804162939.png]]
![[Pasted image 20250804163143.png]]


## GMEM OPEARTIONS


	Loads：

	- Caching

		- Default mode
		- Attempt to hit in L1，then L2，then GMEM
		- Load granularity is 128-byte line

	Stores：
	
	- Invalidate L1，write-back for L2


## LOAD OPERATION

Memory operations are issued per warp(32 threads)

	Just like all other instructions

Operations:

	Threads in a warp provide memory addresses

	Determine which lines/segments are needed

	Request the needed lines/segments

![[Pasted image 20250804165227.png]]

![[Pasted image 20250804170523.png]]

![[Pasted image 20250804174255.png]]

![[Pasted image 20250804174531.png]]

![[Pasted image 20250804174702.png]]


## GMEM OPTIMIZATION GUIDELINES

	Strive for perfect coalescing

		- Align starting address - may require padding
		- A warp should access within a contiguous region
	Havr enought concurrent accesses to saturate the bus

		- Process serval elements per thread
			- Multiple loads get pipelined
			- Indexing calculations can often be reused
		- Launch enough threads to maximize throughtput
			- Latency is hidden by switching threads(warps)

	Use all the caches!!!


## SHARED MEMORY

Uses:

	Inter-thread communication within a block

	Cache data to reduce redundant global memory accesses

	Use it to improve global memory access patterns

Organization:

	32 banks, 4 bytes wide banks

	Success 4-byte words belong to different bank

Performance:

	Typically: 4bytes per bank per 1 or 2 clocks per multiprocessor

	shared accesses are issued per 32 threads(warp)

	serialization: if N threads of 32 access different 4-bytes words in the same bank,, N accesses are executed serially

	multicast: N threads access the same word in one fetch

		Could be different bytes within the same word

## BANK ADDRESSING EXAMPLE

![[Pasted image 20250805085526.png]] 

![[Pasted image 20250805094232.png]]

## SHARED MEMORY：AVOIDING BANK CONFLICTS

- 32 * 32 SMEM 

- Warp accesses a column: 

	- 32- way bank conflicts(threads in a warp access the same bank)

![[Pasted image 20250805095703.png]]

- Warp accesses a column:

	- 32 different banks, no bank conflicts
![[Pasted image 20250805101904.png]]

## SUMMARY

	Kernal Launch Configuration:

		Launch enough threads per SM to hide latency

		Launch tnough threadblocks to load the GPU
		
	Global memory:

		Maximize throughtput(GPU has lots of bandwidth,use it effectively)

	Use shared memory when applicable(over 1TB/s bandwidth)

	Use analysis(profiling when optimizing)

		"Analysis-driven Optimization"(future session)
	