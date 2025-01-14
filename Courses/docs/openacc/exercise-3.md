####<u>Collapse</u>
The collapse clause can be used for the nested loop; an entire part of the iteration will be divided by an available number of threads. If the outer loop is equal to the available threads, then the outer loop will be divided number of threads. The figure below shows an example of not using the `collapse` clause. Therefore, only the outer loop is parallelised; each outer loop index will have N number of inner loop iterations. 

<figure markdown>
![](../figures/parallel-loop-acc.png){align=center width=500}
<figcaption></figcaption>
</figure>

This is not what we want. Instead, with the available threads, we would like to parallelise the loops as efficiently as we could.
Moreover, most of the time, we would have more threads in a given GPU, in our case, we will test Nvidia A100 GPU.
Therefore, when adding the `collapse` clause, we notice that the available threads execute every single iteration, as seen in the figure below.

<figure markdown>
![](../figures/parallel-acc-collapse.png){align=center width=500}
<figcaption></figcaption>
</figure>



We will now look into basic matrix multiplication.
In this example, we will perform the matrix multiplication. Matrix multiplication involves a nested loop.
Again, most of the time, we might end up doing computation with a nested loop.
Therefore, studying this example would be good practice for solving the nested loop in the future. 

<figure markdown>
![](../figures/mat.png){align=center width=500}
</figure>

 - Allocating the CPU memory for A, B, and C matrices.
   Here we notice that the matrix is stored in a
   1D array because we want to consider the same function concept for CPU and GPU.
```c
// Initialize the memory on the host
float *restrict a, *restrict b, *restrict c;

// Allocate host memory
a  = (float*)malloc(sizeof(float) * (N*N));
b  = (float*)malloc(sizeof(float) * (N*N));
c  = (float*)malloc(sizeof(float) * (N*N));
```

 - Now we need to fill the values for the matrix A and B.
```c
// Initialize host matrix
for(int i = 0; i < (N*N); i++)
   {
    a[i] = 2.0f;
    b[i] = 2.0f;
   }
```

 - Calling function
```c
// Function call
Matrix_Multiplication(d_a, d_b, d_c, N);
```

    ??? "matrix multiplication function call"
        
        === "Serial"
            ```c
            void Matrix_Multiplication(float *a, float *b, float *c, int width)
            {
              for(int row = 0; row < width ; ++row)
                {
                  for(int col = 0; col < width ; ++col)
                    {
                      float temp = 0;
                      for(int i = 0; i < width ; ++i)
                        {
                          temp += a[row*width+i] * b[i*width+col];
                        }
                      c[row*width+col] = float;
                    } 
                }   
            }
            ```


        === "OpenACC"
            ```c
            void Matrix_Multiplication(float *restrict a, float *restrict b, float *restrict c, int width)
            {
              int length = width*width;
              float sum = 0;
            #pragma acc parallel copyin(a[0:(length)], b[0:(length)]) copyout(c[0:(length)])
            #pragma acc loop collapse(2) reduction (+:sum)
             for(int row = 0; row < width ; ++row)
                {
                  for(int col = 0; col < width ; ++col)
                    {
                      for(int i = 0; i < width ; ++i)
                        {
                          sum += a[row*width+i] * b[i*width+col];
                        }
                      c[row*width+col] = sum;
                      sum=0;
                    }
                }
            }	    
            ```

 - Deallocate the host memory
```c
// Deallocate host memory
free(a); 
free(b); 
free(c);
```

### <u>Questions and Solutions</u>


??? example "Examples: Matrix Multiplication" 


    === "Serial-version"
        ```c
        #include<stdio.h>
        #include<stdlib.h>
        
        void Matrix_Multiplication(float *h_a, float *h_b, float *h_c, int width)   
        {                                                                 
          for(int row = 0; row < width ; ++row)                           
            {                                                             
              for(int col = 0; col < width ; ++col)                       
                {                                                         
                  float temp = 0;                                       
                  for(int i = 0; i < width ; ++i)                         
                    {                                                     
                      temp += h_a[row*width+i] * h_b[i*width+col];      
                    }                                                     
                  h_c[row*width+col] = temp;                            
                }                                                         
            }   
        }

        int main()
        {
          
          printf("Programme assumes that matrix size is N*N \n");
          printf("Please enter the N size number \n");
          int N =0;
          scanf("%d", &N);

          // Initialize the memory on the host
          float *a, *b, *c;       
    
          // Allocate host memory
          a = (float*)malloc(sizeof(float) * (N*N));
          b = (float*)malloc(sizeof(float) * (N*N));
          c = (float*)malloc(sizeof(float) * (N*N));
  
          // Initialize host matrix
          for(int i = 0; i < (N*N); i++)
            {
              a[i] = 1.0f;
              b[i] = 2.0f;
            }
   
          // Device function call 
          Matrix_Multiplication(a, b, c, N);

          // Verification
          for(int i = 0; i < N; i++)
            {
              for(int j = 0; j < N; j++)
                 {
                  printf("%f ", c[j]);
                 }
              printf("%f ", c[j]);
           }
    
          // Deallocate host memory
         free(a); 
         free(b); 
         free(c);

         return 0;
        }
        ```

    === "OpenACC-template"

        ```c        
        #include<stdio.h>
        #include<stdlib.h>
        #include<openacc.h>
        #include<stdbool.h>
               
        void Matrix_Multiplication(float *restrict a, float *restrict b, float *restrict c, int width)
        {
          int length = width*width;
          float sum = 0;
        //#pragma acc ....
        //#pragma acc ....
         for(int row = 0; row < width ; ++row)
            {
              for(int col = 0; col < width ; ++col)
                {
                  for(int i = 0; i < width ; ++i)
                    {
                      sum += a[row*width+i] * b[i*width+col];
                    }
                  c[row*width+col] = sum;
                  sum=0;
                }
            }
        }    


        // Host call (matrix multiplication)
        void CPU_Matrix_Multiplication(float *h_a, float *h_b, float *h_c, int width)   
        {                                                                 
          for(int row = 0; row < width ; ++row)                           
            {                                                             
              for(int col = 0; col < width ; ++col)                       
                {                                                         
                  float single_entry = 0;                                       
                  for(int i = 0; i < width ; ++i)                         
                    {                                                     
                      single_entry += h_a[row*width+i] * h_b[i*width+col];      
                    }                                                     
                  h_c[row*width+col] = single_entry;                            
                }                                                         
            }   
        }

        int main()
        {
        
          printf("Programme assumes that matrix size is N*N \n");
          printf("Please enter the N size number \n");
          int N =0;
          scanf("%d", &N);
          
          // Initialize the memory on the host
          float *a, *b, *c, *host_check;
                    
          // Initialize host matrix
          for(int i = 0; i < (N*N); i++)
            {
              a[i] = 2.0f;
              b[i] = 2.0f;
            }
            
          // Device function call 
          Matrix_Multiplication(a, b, c, N);

	
          // CPU computation for verification 
          Matrix_Multiplication(a, b, host_check, N);
          
          // Verification
          bool flag=1;
          for(int i = 0; i < N; i++)
            {
             for(int j = 0; j < N; j++)
               {
                 if(c[j*N+i]!= host_check[j*N+i])
                   {
                     flag=0;
                     break;
                   }
               }
            }
          if (flag==0)  
              printf("Two matrices are not equal\n");
          else
              printf("Two matrices are equal\n");
            
          // Deallocate host memory
          free...

          return 0;
        }
        ```
        
    === "OpenACC-version"

        ```c
        #include<stdio.h>
        #include<stdlib.h>
        #include<openacc.h>
        #include<stdbool.h>
        
        void Matrix_Multiplication(float *restrict a, float *restrict b, float *restrict c, int width)
        {
          int length = width*width;
          float sum = 0;
        #pragma acc parallel copyin(a[0:(length)], b[0:(length)]) copyout(c[0:(length)])
        #pragma acc loop collapse(2) reduction (+:sum)
         for(int row = 0; row < width ; ++row)
            {
              for(int col = 0; col < width ; ++col)
                {
                  for(int i = 0; i < width ; ++i)
                    {
                      sum += a[row*width+i] * b[i*width+col];
                    }
                  c[row*width+col] = sum;
                  sum=0;
                }
            }
        }	    


        // Host call (matrix multiplication)
        void CPU_Matrix_Multiplication(float *h_a, float *h_b, float *h_c, int width)   
        {                                                                 
          for(int row = 0; row < width ; ++row)                           
            {                                                             
              for(int col = 0; col < width ; ++col)                       
                {                                                         
                  float single_entry = 0;                                       
                  for(int i = 0; i < width ; ++i)                         
                    {                                                     
                      single_entry += h_a[row*width+i] * h_b[i*width+col];      
                    }                                                     
                  h_c[row*width+col] = single_entry;                            
                }                                                         
            }   
        }


        int main()
        {
        
          cout << "Programme assumes that matrix (square matrix) size is N*N "<<endl;
          cout << "Please enter the N size number "<< endl;
          int N = 0;
          cin >> N;

          // Initialize the memory on the host
          float *a, *b, *c, *host_check;
          
          // Initialize the memory on the device
          float *d_a, *d_b, *d_c;
          
          // Allocate host memory
          a   = (float*)malloc(sizeof(float) * (N*N));
          b   = (float*)malloc(sizeof(float) * (N*N));
          c   = (float*)malloc(sizeof(float) * (N*N));
          host_check = (float*)malloc(sizeof(float) * (N*N));
          
          // Initialize host matrix
          for(int i = 0; i < (N*N); i++)
            {
              a[i] = 2.0f;
              b[i] = 2.0f;
            }
            
          // Device function call 
          Matrix_Multiplication(d_a, d_b, d_c, N);
          
          // cpu computation for verification 
          CPU_Matrix_Multiplication(a,b,host_check,N);
          
          // Verification
          bool flag=1;
          for(int i = 0; i < N; i++)
            {
             for(int j = 0; j < N; j++)
               {
                 if(c[j*N+i]!= host_check[j*N+i])
                   {
                     flag=0;
                     break;
                   }
               }
            }
          if (flag==0)
            {
              cout <<"Two matrices are not equal" << endl;
            }
          else
            cout << "Two matrices are equal" << endl;
            
          // Deallocate host memory
          free(a); 
          free(b); 
          free(c);
          free(host_check);
          
          return 0;
        }
        ```

??? "Compilation and Output"

    === "Serial-version"
        ```c
        // compilation
        $ gcc Matrix-multiplication.c -o Matrix-Multiplication-CPU
        
        // execution 
        $ ./Matrix-Multiplication-CPU
        
        // output
        $ g++ Matrix-multiplication.cc -o Matrix-multiplication
        $ ./Matrix-multiplication
        Programme assumes that matrix (square matrix) size is N*N 
        Please enter the N size number 
        4
        16 16 16 16 
        16 16 16 16  
        16 16 16 16  
        16 16 16 16 
        ```
        
    === "OpenACC-version"
        ```c
        // compilation
        $ nvcc -arch=compute_70 Matrix-multiplication.cu -o Matrix-Multiplication-GPU
        Matrix_Multiplication:
              9, Generating copyin(a[:length]) [if not already present]
                 Generating copyout(c[:length]) [if not already present]
                 Generating copyin(b[:length]) [if not already present]
                 Generating NVIDIA GPU code
                 12, #pragma acc loop gang collapse(2) /* blockIdx.x */
                     Generating reduction(+:sum)
                 14,   /* blockIdx.x collapsed */
                 16, #pragma acc loop vector(128) /* threadIdx.x */
                    Generating implicit reduction(+:sum)
             16, Loop is parallelizable
	        
        // execution
        $ ./Matrix-Multiplication-GPU
        Programme assumes that matrix (square matrix) size is N*N 
        Please enter the N size number
        $ 256
        
        // output
        $ Two matrices are equal
        ```

??? Question "Questions"
    ```
    - Try to compute different matrix sizes instead of square matrices.
    ```

####<u>Three levels of parallelism</u>

By default, the compiler chooses the best combination of the thread blocks needed for the computation. However, sometimes, if needed as a programmer, you could also control the threads block in the program. OpenACC provides straightforward clauses that can control the threads and thread blocks in the application. 


<figure markdown>
![](../figures/OpenACC-Gang-Workers-Vector.png){align=center width=500}
</figure>


|__OpenACC__|__CUDA__|__Parallelism__|
|----------------------|-------------------|----|
|num_gangs|Grid Block|coarse|
|numn_workers|Warps|fine |
|vector_length|Threads|SIMD or vector|


??? Question "Questions"

    - Change the values in `num_gangs()`, `num_workers()` and `vector_length()` and
    check if you would see any performance difference compared to the default thread used by a compiler.
