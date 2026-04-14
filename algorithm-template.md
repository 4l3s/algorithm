##并查集  
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
                parent[n] = FindParent(parent[n]);
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
