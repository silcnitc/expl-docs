---
title: 'Test Program 3 : Sum of Factorials'
hide:
    - toc
---

# Test Program 3 : Sum of Factorials
This test program reads an element and finds the sum of factorials of all the numbers till that element.

## Input
Value of n.
## Output
Sum of factorials of all the numbers till n.

This program tests the declaration of a member field which is an object of another class. This program also tests the recursion, parameter passing, calling a function inside the call of another function.

## code

```
class
fact{
decl
int x;
int findfactorial(int n);
enddecl
int findfactorial(int n){
decl
int p;
enddecl
begin
if(n<=1)then
p=1;
else
p=n*self.findfactorial(n-1);
endif;
return p;
end
}
}
testfactsum{
decl
fact o1;
int testfun(int n);
enddecl
int testfun(int n){
decl
int sum;
enddecl
begin
self.o1=new(fact);
sum=0;
while(n!=0)do
sum=sum+self.o1.findfactorial(n);
n=n-1;
endwhile;
return sum;
end
}
}
endclass
decl
testfactsum obj;
enddecl
int main(){
decl
int x,n;
enddecl
begin
x=initialize();
obj=new(testfactsum);
read(n);
write(obj.testfun(n));
return 1;
end
}
```

