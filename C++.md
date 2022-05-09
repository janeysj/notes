## vector
### 一维vector
```
    // init and set value
    vector<int> vecTest;
    vecTest.clear();
    vecTest.push_back(10);

    // traversal
    for(auto  &changeValue:vecTest)
    {
        changeValue = changeValue * 2;
    }

```    

### 二维vector
```
    // init and set value
    vector<vector<int>> ret(numRows);
    for (int i = 0; i < numRows; ++i) {
        ret[i].resize(i+1);
        ret[i][0] = ret[i][i] = 1;
        for (int j = 1; j < i; ++j) {
            ret[i][j] = ret[i-1][j] + ret[i-1][j-1];
        }
    }

    // traversal
    for (int i=0; i!=ret.size(); i++)
    {
        for (int j = 0; j != ret[i].size(); j++) {
            cout << ret[i][j] << "";
        }
        cout << endl;
    }


```
## cout
cout 只能一个值一个值的int,char这样的输出，不能直接输出一个vector或者字符串