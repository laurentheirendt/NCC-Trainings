Vector addition is one of the basic linear algebra routines.
It involves adding two vectors into one where each index of the corresponding vector should be added.
This vector addition example covers two of the most important OpenACC constructs and clauses: compute constructs and data clauses. They are:

 - Since the computation involves a loop, we could use, **`#pragma acc parallel loop`** or **`#pragma acc kernels loop`**
 - And we need to transfer the data to the GPU. For this purpose, OpenACC provides a rich set of data mapping clauses in OpenACC. 

Data clauses in OpenACC provide a convenient way of handling the data between CPU and GPU.
The following list explains usage and description.

 - **`copy`**: create a space for a variable in the device, copy the data to the device before the region and copy the data back to the host after the region. And releases the memory of the variable in the device. 
 - **`copyin`**:   create a space for a variable in the device, copy the data to the device before the region and do not copy the data back to the host after the region. And releases the memory of the variable in the device. 
 - **`copyout`**: create a space for a variable in the device; do not copy the data to the device before the region and copy the data back to the host after the region. And releases the memory of the variable in the device. 
 - **`create`**:creates a memory of the device; do not copy from host to device or device to host. 
 - **`present`**: The listed variables are already present on the device, so no further action needs to be taken. 
 - **`deviceptr`**: this is quite useful when data has to be managed outside of the OpenACC.

<figure markdown>
![](../figures/OpenACC-data.png){align=middle, width=750}
<figcaption></figcaption>
</figure>



    
!!! Info "Data Constructs"

    === "C/C++"
        ```
        #pragma acc data [clause-list] new-line
           structured block
        ```
	
    === "FORTRAN"
        ```
        !$acc data [clause-list]
           structured block
        !$acc end data
        ```
        
??? "Available caluses for data"

    === "C/C++ and FORTRAN"
        ```c
        if( condition )
        async [( int-expr )]
        wait [( wait-argument )]
        device_type( device-type-list )
        copy( var-list )
        copyin( [readonly:]var-list )
        copyout( [zero:]var-list )
        create( [zero:]var-list )
        no_create( var-list )
        present(a var-list )
        deviceptr( var-list )
        attach( var-list )
        default( none | present )
        ```



We would need just two data clauses from OpenACC in the vector addition example.
The two initialized vectors should be copied to the device from the host; for this purpose, we can use `copyin`.
At the same time, product vectors do not need to be copied from host to device.
However, it should be copied from device to host; for this, we could just use `copyout.` 

The following are the steps for learning vector addition example:

<figure markdown>
![](../figures/vector_add-external.png){align=middle}
<figcaption></figcaption>
</figure>

 - Allocating the CPU memory for a, b, and out vector
```c
// Initialize the memory on the host
float *restrict a, *restrict b, *restrict c;

// Allocate host memory
a = (float*)malloc(sizeof(float) * N);
b = (float*)malloc(sizeof(float) * N);
c = (float*)malloc(sizeof(float) * N);
```

 - Now we need to fill the values for the
    arrays a and b. 
```c
// Initialize host arrays
for(int i = 0; i < N; i++)
  {
    a[i] = 1.0f;
    b[i] = 2.0f;
  }
```

 - Vector addition kernel function call definition

    ??? "vector addition function call"

        === "Serial-version"
            ```c
            // CPU function that adds two vector 
            void Vector_Addition(float *a, float *b, float *c, int n) 
            {
              for(int i = 0; i < n; i ++)
                {
                  c[i] = a[i] + b[i];
                }
            }
            ```
        
        === "OpenACC-version"
            ```c
            // function that adds two vector 
            void Vector_Addition(float *restrict a, float *restrict b, float *restrict c, int n) 
            {
            #pragma acc kernels loop copyin(a[0:n], b[0:n]) copyout:n])
            for(int i = 0; i < n; i ++)
              {
                c[i] = a[i] + b[i];
              }
            }	    
            ```




<figure markdown>
![](../figures/vector_add-external-modified.svg) 
<figcaption></figcaption>
</figure>


 - Deallocate the host memory
```c
// Deallocate host memory
free(a); 
free(b); 
free(c);
```

### <u>Questions and Solutions</u>

??? example "Examples: Vector Addition"


    === "Serial-version"
    
        ```c  
        // Vector-addition.c
        
        #include <stdio.h>
        #include <stdlib.h>
        #include <math.h>
        #include <assert.h>
        #include <time.h>
        
        #define N 5120
        #define MAX_ERR 1e-6

        // CPU function that adds two vector 
        float * Vector_Addition(float *a, float *b, float *c, int n) 
        {
          for(int i = 0; i < n; i ++)
            {
              c[i] = a[i] + b[i];
            }
          return c;
        }

        int main()
        {
          // Initialize the memory on the host
          float *a, *b, *c;       
  
          // Allocate host memory
          a = (float*)malloc(sizeof(float) * N);
          b = (float*)malloc(sizeof(float) * N);
          c = (float*)malloc(sizeof(float) * N);
  
          // Initialize host arrays
          for(int i = 0; i < N; i++)
            {
              a[i] = 1.0f;
              b[i] = 2.0f;
            }
    
          // Start measuring time
          clock_t start = clock();

          // Executing CPU function 
          Vector_Addition(a, b, c, N);

          // Stop measuring time and calculate the elapsed time
          clock_t end = clock();
          double elapsed = (double)(end - start)/CLOCKS_PER_SEC;
        
          printf("Time measured: %.3f seconds.\n", elapsed);
  
          // Verification
          for(int i = 0; i < N; i++)
            {
              assert(fabs(c[i] - a[i] - b[i]) < MAX_ERR);
            }
          printf("PASSED\n");
    
          // Deallocate host memory
          free(a); 
          free(b); 
          free(c);
   
          return 0;
        }
        ```

    === "OpenACC-template"
    
        ```c  
        // Vector-addition-template.c
	
        #include <stdio.h>
        #include <stdlib.h>
        #include <math.h>
        #include <assert.h>
        #include <time.h>
        #include <openacc.h>	


        #define N 5120
        #define MAX_ERR 1e-6


        // GPU function that adds two vectors 
        // function that adds two vector 
        void Vector_Addition(float *restrict a, float *restrict b, float *restrict c, int n) 
        {

        // add here either parallel or kernel plus data map clauses
        #pragma acc 
        for(int i = 0; i < n; i ++)
           {
             c[i] = a[i] + b[i];
           }
        }

        int main()
        {
          // Initialize the memory on the host
          float *restrict a, *restrict b, *restrict c;       
  
          // Allocate host memory
          a = (float*)malloc(sizeof(float) * N);
          b = (float*)malloc(sizeof(float) * N);
          c = (float*)malloc(sizeof(float) * N);
  
          // Initialize host arrays
          for(int i = 0; i < N; i++)
            {
              a[i] = 1.0f;
              b[i] = 2.0f;
            }
    
          // Start measuring time
          clock_t start = clock();

          // Executing CPU function 
          Vector_Addition(a, b, c, N);

          // Stop measuring time and calculate the elapsed time
          clock_t end = clock();
          double elapsed = (double)(end - start)/CLOCKS_PER_SEC;
        
          printf("Time measured: %.3f seconds.\n", elapsed);
  
          // Verification
          for(int i = 0; i < N; i++)
            {
              assert(fabs(c[i] - a[i] - b[i]) < MAX_ERR);
            }

          printf("PASSED\n");
    
          // Deallocate host memory
          free(a); 
          free(b); 
          free(c);
   
          return 0;
        }
        ```
	
    === "OpenACC-version"
    
        ```c  
        // Vector-addition-openacc.c
        
        #include <stdio.h>
        #include <stdlib.h>
        #include <math.h>
        #include <assert.h>
        #include <time.h>
        #include <openacc.h>

        #define N 5120
        #define MAX_ERR 1e-6


        // function that adds two vector 
        void Vector_Addition(float *restrict a, float *restrict b, float *restrict c, int n) 
        {
        // or #pragma acc kernels loop copyin(a[0:n], b[0:n]) copyout(c[0:n])
        #pragma acc kernels loop copyin(a[0:n], b[0:n]) copyout(c[0:n])
        for(int i = 0; i < n; i ++)
           {
            c[i] = a[i] + b[i];
           }
        }

        int main()
        {
          // Initialize the memory on the host
          float *restrict a, *restrict b, *restrict c;       
  
          // Allocate host memory
          a = (float*)malloc(sizeof(float) * N);
          b = (float*)malloc(sizeof(float) * N);
          c = (float*)malloc(sizeof(float) * N);
  
          // Initialize host arrays
          for(int i = 0; i < N; i++)
            {
              a[i] = 1.0f;
              b[i] = 2.0f;
            }
    
          // Start measuring time
          clock_t start = clock();

          // Executing CPU function 
          Vector_Addition(a, b, c, N);

          // Stop measuring time and calculate the elapsed time
          clock_t end = clock();
          double elapsed = (double)(end - start)/CLOCKS_PER_SEC;
          
          printf("Time measured: %.3f seconds.\n", elapsed);
  
          // Verification
          for(int i = 0; i < N; i++)
            {
              assert(fabs(c[i] - a[i] - b[i]) < MAX_ERR);
            }

          printf("PASSED\n");
    
          // Deallocate host memory
          free(a); 
          free(b); 
          free(c);
   
          return 0;
        }
        ```


??? "Compilation and Output"

    === "Serial-version"
        ```c
        // compilation
        $ gcc Vector-addition.c -o Vector-Addition-CPU
        
        // execution 
        $ ./Vector-Addition-CPU
        
        // output
        $ ./Vector-addition-CPU 
        PASSED
        ```
        
    === "OpenACC-version"
        ```c
        // compilation
        $ nvc -fast -acc=gpu -gpu=cc80 -Minfo=accel Vector-addition-openacc.c -o Vector-Addition-GPU
        Vector_Addition:
        12, Generating copyin(a[:n]) [if not already present]
            Generating copyout(c[:n]) [if not already present]
            Generating copyin(b[:n]) [if not already present]
        14, Loop is parallelizable
            Generating NVIDIA GPU code
            14, #pragma acc loop gang, vector(128) /* blockIdx.x threadIdx.x */

        // execution
        $ ./Vector-Addition-GPU
        
        // output
        $ ./Vector-addition-GPU
        PASSED
        ```


??? Question "Questions"

    - Do you notice any performance difference using `parallel` and `kernels` compute constructs
    - Alternatively, you could also use  `num_gangs()`, `num_workers()` and `vector_length()` to manually set the number of grids, thread blocks and threads. 
      And check if you could see any performance difference compared to the default thread used by a compiler. 
