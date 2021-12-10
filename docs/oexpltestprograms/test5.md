---
title: 'Test Program 5'
hide:
    - toc
---

This program tests the correct set up of the virtual function table by the compiler.

## Input : If n < 0
## Output
```
In A F0
```

## Input : If n == 0
## Output
```
In B F0
```

## Input : If n > 0
## Output
```
In C F0
```

This program uses the virtual function table, inheritance and subtype polymorphism.

```
class
A
{
decl
int f0();
int f1();
enddecl
int f0() {
begin
write("In class A f0");
return 1;
end
}
int f1() {
begin
write("In class A f1");
return 1;
end
}
}
B extends A
{
decl
int f0();
int f2();
enddecl
int f0() {
begin
write("In class B f0");
return 1;
end
}
int f2() {
begin
write("In class B f2");
return 1;
end
}
}
C extends B
{
decl
int f0();
int f2();
int f4();
enddecl
int f0() {
begin
write("In class C f0");
return 1;
end
}
int f2() {
begin
write("In class C f2");
return 1;
end
}
int f4() {
begin
write("In class C f4");
return 1;
end
}
}
endclass
decl
A obj ;
A test_obj;
enddecl
int main() {
decl
int temp,n;
enddecl
begin
temp= initialize();
read(n);
if(n < 0)then
obj = new(A);
else
if(n == 0)then
obj = new(B);
else
if(n > 0)then
obj = new(C);
endif;
endif;
endif;
test_obj = obj;
write(test_obj.f0());
return 1;
end
}
```
