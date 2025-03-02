# ACM输入输出模板

输入部分分为C++和JAVA语言

[牛客网在线编程_算法篇_输入输出练习](https://www.nowcoder.com/exam/oj?page=1&tab=算法篇&topicId=372)

**lower_bound** 返回指向第一个值不小于 val 的位置，也就是返回第一个大于等于val值的位置。

前提是有序的情况下，**upper_bound** 返回第一个大于--val值的位置。

## 定义链表

```C++
#include<iostream>
using namespace std;
struct ListNode{
    int val;
    ListNode* nxt;
    ListNode(int _val):val(_val),nxt(nullptr){}
   	ListNode(int _val, ListNode* _nxt):val(_val),nxt(_nxt){}
}
int main() {
    // 创建一个单链表
    ListNode* head = nullptr;
    ListNode* pre = head;
    ListNode* cur = nullptr;
    int num;
    while(cin >> num) {
        if(num == -1) {
            break;
        }
        cur = new ListNode(num);
        if(head == nullprt) {
            head = cur;
            pre = cur;
        } else {
            pre.nxt = cur;
            pre = cur;
        }
    }
    return 0;
}
```

## 定义二叉树

```C++
#include <iostream>
#include <vector>
#include <queue>

using namespace std;

//定义树节点
struct TreeNode
{
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode():val(0),left(nullptr),right(nullptr){}
    TreeNode(int _val):val(_val),left(nullptr),right(nullptr){}
    TreeNode(int _val,TreeNode* _left,TreeNode* _right):val(0),left(_left),right(_right){}
};

//根据数组生成树
TreeNode* buildTree(const vector<int>& v)
{
    vector<TreeNode*> vTree(v.size(),nullptr);
    TreeNode* root = nullptr;
    for(int i = 0; i < v.size(); i++)
    {
        TreeNode* node = nullptr;
        if(v[i] != -1)
        {
            node = new TreeNode(v[i]);
        }
        vTree[i] = node;
    }
    root = vTree[0];
    for(int i = 0; 2 * i + 2 < v.size(); i++)
    {
        if(vTree[i] != nullptr)
        {
            vTree[i]->left = vTree[2 * i + 1];
            vTree[i]->right = vTree[2 * i + 2];
        }
    }
    return root;
}
 
int main()
{
	// 验证
    vector<int> v = {4,1,6,0,2,5,7,-1,-1,-1,3,-1,-1,-1,8};
    TreeNode* root = buildTree(v);
    
    return 0;
}

```

