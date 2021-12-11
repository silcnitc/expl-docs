---
title: 'Constant Function'
---

```
decl
 int fun(int x),a;
enddecl

int fun(int x)
{
 begin
  if(x >= 91) then
   a = 91;
  else
   a = fun(fun(x+11));
  endif;

  return a;
 end
}

int main()
{
 decl
  int b,c;
 enddecl

 begin
  read(b);
  c = fun(b);
  write(c);

  return 0;
 end
}
```
