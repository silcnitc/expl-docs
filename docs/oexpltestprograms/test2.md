---
title: 'Test Program 2 : Linked list'
hide:
    - toc
---

#Test Program 2 : Linked list
This test program reads elements into a linked list and prints them.

## Input
Value of length of list and list of values from standard input.

## Output
The list of values stored in linked list.

This program test the working of dynamic memory allocation functions like Initialize(), Alloc() and Free().

## Code
```
type
    list{
        int data;
        list next;
    }
endtype
class
linkedlist{
decl
list head;
list tail;
int length;
int getlength();
int init();
list insert(int data);
int printlinkedlist();
enddecl
int getlength(){
begin
return self.length;
end
}
int init(){
begin
self.head=null;
self.tail=null;
self.length=0;
return 1;
end
}
list insert(int data){
decl
list temp;
enddecl
begin
temp=alloc();
temp.data=data;
temp.next=null;
if(self.head== null)then
self.head=temp;
self.tail=temp;
else
self.tail.next=temp;
self.tail=temp;
endif;
self.length=self.length+1;
return temp;
end
}
int printlinkedlist(){
decl
list temp;
enddecl
begin
temp=self.head;
while(temp!= null)do
write(temp.data);
temp=temp.next;
endwhile;
return 1;
end
}
}
endclass
decl
linkedlist obj;
enddecl
int main(){
decl
int x,y,z;
list a;
enddecl
begin
x=initialize();
obj=new(linkedlist);
x=obj.init();
read(x);
while(x!=0)do
read(y);
a=obj.insert(y);
x=x-1;
endwhile;
write(obj.getlength());
x=obj.printlinkedlist();
return 1;
end
}
```
