PG9版本的
- class matNM类
```
template <typename T, const int w, const int h>//注意这个w表示有几个vectorN，就是列数。h表示每个vectorN的维度，就是行数
class matNM
{
public:
    typedef class matNM<T,w,h> my_type;//注意这个w表示有几个vectorN，就是列数。h表示每个vectorN的维度，就是行数
    typedef class vecN<T,h> vector_type;//矩阵的一个列向量
    vecN<T,h> data[w];//数学上正常写矩阵，一个vecN表示一列
}
m[][]表示先列后行
```

- vmath的othor函数的矩阵第[2,2]个元素应该是$2/(n-f)$，第[3,2]个元素应该是$(n+f)/(n-f)$