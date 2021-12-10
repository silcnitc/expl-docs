---
title: 'Test Program 1 : Binary Search Tree'
hide:
    - toc
---

# Test Program 1 : Binary Search Tree

This test program inserts elements into a binary search tree and prints the elements in inorder, preorder and postorder traversal.
The program stops taking input elements (that are to be inserted in Binary Search Tree) when the user enters 0.

## Input
The elements that are to be inserted into a Binary Search Tree and a 0 at last (indicating the end of input).

## Output
The inorder, preorder and postorder traversal of the tree.

This program test the iteration, recursion, conditional, arrays and passing of user-defined datatype as return value of function.

## Code
```
type
    bst{
        int a;
        bst left;
        bst right;
    }
endtype
class
    bstclass{
        decl
            bst root;
            int init();
            bst getroot();
            int setroot(bst n1);
            bst getnode(int key);
            bst insert(bst h, int key);
            int inOrder_fun(bst h);
            int preOrder_fun(bst h);
            int postOrder_fun(bst h);
        enddecl
        int init(){
            begin
                self.root=null;
                return 1;
            end
        }
        bst getroot(){
            begin
                return self.root;
            end
        }
        int setroot(bst n1){
            begin
                self.root=n1;
                return 1;
            end
        }
        bst getnode(int key){
            decl
                bst temp;
            enddecl
            begin
                temp=alloc();
                temp.a=key;
                temp.left=null;
                temp.right=null;
                return temp;
            end
        }
        bst insert(bst h, int key){
            begin
                if (h == null) then
                    h = self.getnode(key);
                else
                    if (key < h.a) then
                        h.left = self.insert(h.left, key);
                    else
                        if (key > h.a) then
                            h.right = self.insert(h.right, key);
                        endif;
                    endif;
                endif;
                return h;
            end
        }
        int inOrder_fun(bst h){
            decl
                int in;
            enddecl
            begin
                if(h!= null) then
                    in=self.inOrder_fun(h.left);
                    write(h.a);
                    in=self.inOrder_fun(h.right);
                endif;
                return 1;
            end
        }
        int preOrder_fun(bst h){
            decl
                int in;
            enddecl
            begin
                if(h!= null) then
                    write(h.a);
                    in=self.preOrder_fun(h.left);
                    in=self.preOrder_fun(h.right);
                endif;
                return 1;
            end
        }
        int postOrder_fun(bst h){
            decl
                int in;
            enddecl
            begin
                if(h!= null) then
                    in=self.postOrder_fun(h.left);
                    in=self.postOrder_fun(h.right);
                    write(h.a);
                endif;
                return 1;
            end
        }
    }
endclass
decl
    bstclass obj;
enddecl
int main(){
    decl
        bst Root;
        int x,in,val;
    enddecl
    begin
        x=initialize();
        obj = new(bstclass);
        x=obj.init();
        read(val);
        Root = obj.getroot();
        while(val!=0) do
            Root = obj.insert(Root,val);
            read(val);
        endwhile;
        x = obj.setroot(Root);
        in = obj.inOrder_fun(obj.getroot());
        in = obj.preOrder_fun(obj.getroot());
        in = obj.postOrder_fun(obj.getroot());
        return 0;
    end
}
```
