Given a binary tree, return the inorder traversal of its nodes' values.

For example:  
Given binary tree [1,null,2,3],  

```
   1
    \
     2
    /
   3
```

return [1,3,2].

Note: Recursive solution is trivial, could you do it iteratively?  

# Solution

### Solution 1, Morris in-order traversal using threading

__Time complexity O(number of nodes), space complexity O(1).__

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> reval;
        
        while (root) {
            if (root->left) {
                // Predecessor must come from left tree.
                TreeNode* predecessor = root->left;
                
                while (predecessor->right && predecessor->right != root) {
                    predecessor = predecessor->right;
                }
                
                if (predecessor->right != root) { // Going down
                    predecessor->right = root;
                    root = root->left;
                } else { // Going up
                    predecessor->right = nullptr; // Revert it back
                    reval.emplace_back(root->val);
                    root = root->right;
                }
            } else{
                reval.emplace_back(root->val);
                root = root->right;
            }
        }
        
        return reval;
    }
};
```

### Solution 2, iterative solution with a stack

Whenever we first encounter a node, we push it to stack, and keep going to its left and adding nodes to stack.

Whenever current node is nullptr, we begin to pop node out of stack. For every node popped from stack, its left branch has been visited. We process this node, and set current node pointer to its right child, which is not pushed to stack yet.

__Time complexity O(number of nodes), space complexity O(depth of tree).__

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> res;
        if(!root) return res;
        stack<TreeNode*> my_stack;
        while(root || !my_stack.empty()){
            if(root){//root to leaf,going down
                my_stack.push(root);
                root = root->left;
            } else {// leaf to root, going up
                root = my_stack.top();
                my_stack.pop();
                res.push_back(root->val);
                root = root->right; // Note it is not res.push_back(root->right)!
            }
        }
        return res;
    }
};
```

### Solution 3, recursive solution

__Time complexity O(number of nodes), space complexity O(depth of recursion).__

```cpp
void inorder(TreeNode* root, vector<int>& nodes) {
    if (!root) return;
    inorder(root -> left, nodes);
    nodes.push_back(root -> val);
    inorder(root -> right, nodes);
}
vector<int> inorderTraversal(TreeNode* root) {
    vector<int> nodes;
    inorder(root, nodes);
    return nodes;
} 
```

# Knowledge

### Morris in-order traversal using threading

![alt](https://2.bp.blogspot.com/-Oi7ZBzR9Wzs/V5YWWkB1FII/AAAAAAAAY8k/hVhzEWlwigM5HpHHN1VIZITzw9By4zAOACLcB/s1600/threadedBT.png)

A binary tree is threaded by making every left child pointer (that would otherwise be null) point to the in-order predecessor of the node (if it exists) and every right child pointer (that would otherwise be null) point to the in-order successor of the node (if it exists).

__Advantages:__

1. Avoids recursion, which uses a call stack and consumes memory and time.  
2. The node keeps a record of its parent.  

__Disadvantages:__

1. The tree is more complex.  
2. We can make only one traversal at a time.  
3. It is more prone to errors when both the children are not present and both values of nodes point to their ancestors.  

Morris traversal is an implementation of in-order traversal that uses threading:

1. Create links to the in-order successor.  
2. Print the data using these links.  
3. Revert the changes to restore original tree.    

### Threaded binary tree

In computing, a threaded binary tree is a binary tree variant that allows fast traversal: given a pointer to a node in a threaded tree, it is possible to cheaply find its in-order successor (and/or predecessor).

![alt](https://upload.wikimedia.org/wikipedia/commons/7/7a/Threaded_tree.svg)

A threaded binary tree defined as follows: "A binary tree is threaded by making all right child pointers that would normally be null point to the inorder successor of the node (if it exists), and all left child pointers that would normally be null point to the inorder predecessor of the node."
