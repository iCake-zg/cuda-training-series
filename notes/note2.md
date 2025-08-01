
## SHARING MEMORY BETWEEN THREADS


	1. Within a block, threads share data via shared memory

	2. Declare using __shared__,allocated per block

	3. Data is not visible to threads in other blocks


  **SHARED MEMORT**  :  on GPU chip  (higher Bandwidth lower latency)
  *DIFFERENT* with      
  **GLOBAL MEMORY**   :  on memory with GPU


## IMPLEMENTING WITH SHARED MEMORY

Cache data in shared memory

	read(blockDim.x + 2 * radius) input elements from global memory to shared memory.

	Compute blockDim.x output elements.

	Write blockDim.x output elements to global memory


Each block needs a halo of **radius** elements at each boundary

## STENCIL KERNEL

![[Pasted image 20250731084942.png]]
```cpp


__global__ void stencil_1d(int *in, int *out){

	__shared__ int temp[BLOCK_SIZE + 2 * RADIUS]

	int gindex = threadIdx.x + blockIdx.x * blockDim.x;
	int lindex = threadIdx.x + RADIUS;

	// read input elements into shared memory

	temp[lindex] = in[gindex];

	if(threadIdx.x < RADIUS){
		temp[lindex - RADIUS] = in[gindex - RADIUS];
		temp[lindex + BLOCK_SIZE] = in[gindex + BLOCK_SIZE];
	}

	// Synchronize(ensure all the data is available)

	__syncthreads();

	int result = 0;
	for(int offset = -RADIUS; offset <= RADIUS; offset++)
		result += temp[lindex+offset];
	out[gindex] = result
}

```



## SYNCTHREADS()

```cpp
void __syncthreads();
```

Synchronizes all threads within a block

	Used to prevent RAW/WAR/WAW hazards

All threads must reach the barrier

	In conditional code, the condition must be uniform across the block

## REVIEW


```cpp
Use __shared__ tp declare a variable/array in shared memory
```

	Data is shared between threads in a block

	Not visible to threads in other blocks

```cpp
Use __sychthreads() as a barrier
```

	Use to prevent data hazards


    