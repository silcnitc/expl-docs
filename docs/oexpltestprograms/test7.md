---
title: 'Test program 7'
hide:
    - toc
---

#Test program 7

This program tests the implementation of inheritance and subtype polymorphism.

## Input : > 0

## Output
```
Rogers
37
```

## Input : <= 0

## Output :
```
Mathew
35
999
CS
```
This program tests the concepts of inheritance, subtype polymorphism and virtual function table.

## code

```
class
Person{
decl
str name;
int age;
int printDetails();
str findName();
int createPerson(str name, int age);
enddecl
int printDetails(){
decl
enddecl
begin
write(self.name);
write(self.age);
return 1;
end
}
str findName(){
decl
enddecl
begin
return self.name;
end
}
int createPerson(str name, int age){
decl
enddecl
begin
self.name=name;
self.age=age;
return 1;
end
}
}
Student extends Person{
decl
int rollnumber;
str dept;
int printDetails();
int createStudent(str name, int age,int rollNo, str dept);
enddecl
int createStudent(str name, int age,int rollNo, str dept){
decl
enddecl
begin
self.name =name;
self.age = age;
self.rollnumber = rollNo;
self.dept = dept;
return 1;
end
}
int printDetails(){
decl
enddecl
begin
write(self.name);
write(self.age);
write(self.rollnumber);
write(self.dept);
return 1;
end
}
}
endclass
decl
int n,temp;
str name;
Person first;
Student second;
Person arbitrary;
enddecl
int main(){
decl
enddecl
begin
n = initialize();
first=new(Person);
temp = first.createPerson("Rogers", 37);
second=new(Student);
temp = second.createStudent("Mathew", 35, 999, "CS");
read(n);
if (n>0) then
arbitrary = first;
else
arbitrary = second;
endif;
n = arbitrary.printDetails();
return 0;
end
}
```
