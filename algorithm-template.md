## 并查集  
并查集模板：
```java
    class UnionFind{
        private int[] parent;
        private int[] rank;

        public UnionFind(int n){
            parent =  new int[n];
            rank = new int[n];
            for(int i= 0; i < n; i++){
                parent[i] = i;
                rank[i] = 0;
            }
        }
        public int FindParent(int n){
            if(parent[n] == n){
                return n;
            }else {
                parent[n] = FindParent(parent[n]);//路径压缩
                return  parent[n];
            }
        }

        public void Union(int x,int y){
            int rootx = FindParent(x);
            int rooty = FindParent(y);
            if(rootx!=rooty){
                if(rank[rootx] > rank[rooty]){
                    parent[rooty] = rootx;
                }else if(rank[rootx]<rank[rooty]){
                    parent[rootx] = rooty;
                } else {
                    parent[rooty] = rootx;
                    rank[rootx]++;
                }
            }

        }
    }
```
q1：  
rank; 是不是真有必要  
不是非要不可，主要是为了性能优化。两个数合并，rank小的合并到大的，查找快  
q2：  
并查集 中，每次find之后，路径都会被压缩，rank其实是变化的，意义在哪里  
在路径压缩后，rank 确实不再代表树的“真实高度”，它变成了一个“过期的”数值。但它的意义在于：它仍然是一个有效的“高度上界”。  
只要它能保证“这棵树不会无限长高”，它就完成了使命。  
为了让你彻底理解，我们可以从以下三个维度来拆解：  
1. rank 的真实身份：是“上界”而非“实测值”
2. rank 是为了防“下限”，不是为了求“最优
   
