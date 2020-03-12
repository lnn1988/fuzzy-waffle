# 树


```py
# 定义树结构体
class Tree:
    def __init__(self, val):
        self.val = val
        self.left = None
        self.right = None


class tree_func:
    def __init__(self):
        self.re = []

    # 深度优先（递归）
    def dfs_search(self, node):
        if node == None:
            return
        self.re.append(node.val)
        if node.left:
            self.dfs_search(node.left)
        if node.right:
            self.dfs_search(node.right)
        return

    # 广度优先（递归）
    def bfs_search(self, nodes):
        if nodes == []:
            return
        child_nodes = []
        for node in nodes:
            if node:
                self.re.append(node.val)
                if node.left:
                    child_nodes.append(node.left)
                if node.right:
                    child_nodes.append(node.right)
        self.bfs_search(child_nodes)
        return

    # 翻转二叉树
    def reverse(self, node):
        if node == None:
            return None
        node.left, node.right = self.reverse(node.right), self.reverse(node.left)
        return node

    # 深度优先（循环方法）
    def dfs_search_cycle(self, node):
        if node == None:
            return
        nodes = [node]
        c = 0
        while nodes:
            c += 1
            cur_node = nodes[0]
            nodes = nodes[1:]
            self.re.append(cur_node.val)
            if cur_node.right:
                nodes = [cur_node.right] + nodes

            if cur_node.left:
                nodes = [cur_node.left] + nodes

    # 广度优先（循环方法）
    def bfs_search_cycle(self, node):
        if node == None:
            return
        nodes = [node]
        while nodes:
            for node in nodes:
                if node:
                    self.re.append(node.val)
                    nodes = nodes[1:]
                if node.left:
                    nodes.append(node.left)
                if node.right:
                    nodes.append(node.right)



t1 = Tree(1)
t2 = Tree(2)
t3 = Tree(3)
t4 = Tree(4)
t5 = Tree(5)
t6 = Tree(6)
t7 = Tree(7)
t1.left = t2
t1.right = t3
t2.left = t4
t2.right = t5
t3.left = t6
t5.left = t7

# f = tree_func()
# f.dfs_search(t1)
# print(f.re)
# f.re = []
# f.bfs_search([t1])
# print(f.re)

# t = f.reverse(t1)
# f.dfs_search(t)
# print(f.re)

# f = tree_func()
# f.bfs_search_cycle(t1)
# print(f.re)
```