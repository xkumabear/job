```c++
int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
        //层次遍历图时，最先找到的一定是最短路径
        vector<bool> used( wordList.size() , false );
        queue<string> que;
        int count = 0;
        que.push(beginWord);
        while(!que.empty()){
            count++;
            // 最蠢的事情 对于容器来说当我们进行插入和删除时，其size 以及迭代器是动态的变换的 在for循环中不可将其作为循环条件。
            int size = que.size();
            for( int i = 0 ; i < size ; i++ ){
                string temp = que.front();
                que.pop();
                for( int j = 0 ; j < wordList.size() ; j++ ){
                    if( !used[j] && isvalid( temp , wordList[j])){
                        if( endWord == wordList[j] ) return count + 1;
                        que.push(wordList[j]);
                        used[j] = true;
                    }
                }  
            }
        }
        return 0;
    }
```

