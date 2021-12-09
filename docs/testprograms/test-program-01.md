---
title: 'Test Program 1: Bubblesort(iterative)'
hide:
    - toc
---

# Test Program 1: Bubblesort(iterative)

This test program reads elements into an array and sorts them using the classic bubblesort algorithm. (iterative version)

#### Input
1. Number of elements to be sorted from standard input.
2. Elements to be sorted

#### Output
A sorted array of elements.

This program test the iteration, conditional and arrays.

```

decl
   int n,arr[50],i,j,dup;
enddecl

int main()
{
  begin
  read(n);

  i=0;
  while(i<n) do
    read(arr[i]);
    i = i+1;
  endwhile;

  i=0;
  while(i<n) do
    j=i;
    while(j<n) do
      if(arr[i]>arr[j]) then
        dup = arr[i];
        arr[i] = arr[j];
        arr[j] = dup;
      endif;
      j = j + 1;
    endwhile;
    i = i+1;
  endwhile;

  i=0;
  while(i<n) do
    write(arr[i]);
    i = i+1;
  endwhile;

  return 0;
  end
}

```
